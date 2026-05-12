

  

## 问题现象

  

主应用先访问康佰家云子应用（`/kbj-cloud-mvp/cloud/my`），再点击侧边栏切换到其他子应用时：

  

1. URL 和视图正常切换到目标子应用

2. 页面出现约 5 秒 loading

3. URL 被重定向为 `https://test.kbjcn.comundefined/`，目标子应用被冲掉

  

其他子应用之间互相切换**不会**出现此问题，只有**经过本子应用后**才会触发。

  

---

  

## 根因分析

  

### 核心原因：`BrowserRouter` 监听了浏览器 `popstate` 事件

  

React Router v6 的 `BrowserRouter` 会：

- 监听 `window.popstate` 事件

- 每次导航调用 `history.pushState` / `history.replaceState`

  

而 qiankun 框架对 `history.pushState` 做了补丁（monkey-patch），使其在调用后会派发一个**合成的 `popstate` 事件**，让所有子应用都能感知到 URL 变化。

  

### 触发链条

  

```

主应用点击菜单切换到其他子应用

    ↓

qiankun 调用 history.pushState(新 URL)

    ↓

qiankun 补丁派发合成 popstate 事件

    ↓

本子应用的 BrowserRouter 监听到 popstate

    ↓

react-router 尝试匹配新 URL，发现不在 basename("/kbj-cloud-mvp") 范围内

    ↓

react-router 将路径降级为 "/"

    ↓

命中 <Route path="/" element={<Navigate to="/cloud/my" replace />} />

    ↓

Navigate 组件触发 history.replaceState，将 URL 写回本子应用路径

    ↓

（同时）history.state 被 React Router 的结构 {idx, key, usr} 覆盖

    ↓

qiankun 检测到 URL 又匹配了本子应用的 activeRule → 重新激活/挂载

    ↓

ensureLoggedIn() 执行 → 5秒 loading

    ↓

目标子应用的 vue-router 读取被污染的 history.state → 取到 undefined

    ↓

vue-router 的 changeLocation 把 undefined 拼接到 origin 后面

    ↓

URL 变为 https://test.kbjcn.comundefined/

```

  

### 为什么只有本子应用会触发？

  

| 条件 | 本子应用 | 其他子应用 |

|------|----------|------------|

| 根路径使用 `<Navigate>` 自动重定向 | ✅ | ❌ 一般是真实页面组件 |

| 使用 `BrowserRouter` 监听 popstate | ✅ | 部分用 HashRouter 或 MemoryRouter |

| unmount 前 React 树仍活跃 | ✅ | 大部分框架的卸载时序更快 |

  

三个条件**同时满足**才会触发，其他子应用通常至少缺少其中一条。

  

---

  

## 尝试过的方案及结果

  

### 方案 1：SafeRedirect 组件（渲染时检查 pathname）

  

```tsx

function SafeRedirect({ to, basename }) {

  if (basename && !window.location.pathname.startsWith(basename)) return null;

  return <Navigate to={to} replace />;

}

```

  

**结果：❌ 无效**

  

原因：react-router 内部的 effect 调度发生在渲染之后，检查时机已过。

  

### 方案 2：popstate capture 阶段守卫（提前 unmount React 树）

  

```ts

window.addEventListener('popstate', () => {

  if (!location.pathname.startsWith(basename) && root) {

    root.unmount();

    root = null;

  }

}, true); // capture 阶段

```

  

**结果：❌ 无效**

  

原因：qiankun 切换子应用走的是 `pushState`，不触发原生 `popstate`。qiankun 派发的合成 `popstate` 与 react-router 内部已 schedule 的 state 更新存在时序竞争，capture 拦截后 effect 仍然执行。

  

### 方案 3：DeferredRedirect（setTimeout 延迟重定向）

  

```tsx

function DeferredRedirect({ to, basename }) {

  const navigate = useNavigate();

  useEffect(() => {

    const id = setTimeout(() => {

      if (!location.pathname.startsWith(basename)) return;

      navigate(to, { replace: true });

    }, 0);

    return () => clearTimeout(id);

  }, [to, basename]);

  return null;

}

```

  

**结果：❌ 无效**

  

原因：虽然能延迟本组件的重定向，但 `BrowserRouter` 内部的 state 更新（通过 `useSyncExternalStore` 或类似机制）已经同步发生，后续渲染周期仍然会触发 effect。且 qiankun 的 unmount 调用也是异步的，时序不可控。

  

### 方案 4：unmount 时清空 history.state

  

```ts

export async function unmount() {

  root?.unmount();

  root = null;

  history.replaceState(null, '', location.href);

}

```

  

**结果：❌ 无效**

  

原因：重定向不是由 `history.state` 污染直接触发的，是 `BrowserRouter` 监听到 popstate 后主动发起的导航。清理 state 无法阻止 BrowserRouter 的事件监听器执行。

  

### 方案 5（最终方案）：qiankun 模式下使用 MemoryRouter ✅

  

```tsx

{isQiankun ? (

  <MemoryRouter initialEntries={[initialEntry]}>

    {routeContent}

  </MemoryRouter>

) : (

  <BrowserRouter basename={basename}>

    {routeContent}

  </BrowserRouter>

)}

```

  

**结果：✅ 有效，彻底解决**

  

---

  

## 最终方案详解

  

### 原理

  

`MemoryRouter` 的路由状态完全存储在内存中：

- **不监听** `window.popstate`

- **不调用** `history.pushState` / `replaceState`

- 与浏览器 URL **零交互**

  

因此无论主应用如何切换路由，本子应用的路由系统完全不感知、不响应、不干扰。

  

### 需要额外处理的问题

  

| 问题 | 解决方式 |

|------|----------|

| URL 不同步（地址栏不更新） | `UrlSync` 组件用 `history.replaceState` 手动同步 |

| 初始路径丢失（深链接） | 从 `window.location.pathname` 推导 `initialEntries` |

| 独立运行模式不受影响 | 条件判断 `isQiankun`，独立运行仍用 `BrowserRouter` |

  

### UrlSync 组件

  

```tsx

function UrlSync({ basename }: { basename: string }) {

  const location = useLocation();

  useEffect(() => {

    const target = basename + location.pathname + location.search + location.hash;

    if (window.location.pathname + window.location.search + window.location.hash !== target) {

      window.history.replaceState(window.history.state, '', target);

    }

  }, [basename, location]);

  return null;

}

```

  

---

  

## 经验总结

  

1. **qiankun + BrowserRouter + 根路径 Navigate 是危险组合** — 三者叠加必然导致跨应用路由冲突

2. **qiankun 对 pushState 的补丁会派发合成 popstate** — 所有监听 popstate 的代码都会被影响

3. **React Router 的 effect 调度无法通过外部手段可靠取消** — 只要 BrowserRouter 还活着，它的内部状态就会响应所有 history 事件

4. **MemoryRouter 是 qiankun 子应用的最佳实践** — 路由完全内聚，配合 UrlSync 保持 URL 可用性

5. **调试微前端路由问题时，劫持 `history.pushState` / `replaceState` 打印调用栈**是最高效的定位手段