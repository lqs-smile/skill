# 高质量代码编写范式

  

## 1. 卫语句 (Guard Clause) — 早返回，减少嵌套

  

```ts

// ❌

function process(user) {

  if (user) {

    if (user.isActive) {

      // 20 行逻辑...

    }

  }

}

  

// ✅

function process(user) {

  if (!user) return;

  if (!user.isActive) return;

  // 20 行逻辑...

}

```

  

## 2. 提取解释性变量 (Explanatory Variable) — 用命名代替复杂表达式

  

```ts

// ❌

if (user.age >= 18 && user.status === 'active' && !user.isBanned && user.balance > 0) {}

  

// ✅

const isEligible = user.age >= 18 && user.status === 'active' && !user.isBanned;

const hasBalance = user.balance > 0;

if (isEligible && hasBalance) {}

```

  

## 3. 查表法 / 映射驱动 (Table-Driven / Map-Driven) — 消灭 if-else 链

  

```ts

// ❌

function getLabel(status) {

  if (status === 1) return '待审核';

  if (status === 2) return '已通过';

  if (status === 3) return '已拒绝';

  return '未知';

}

  

// ✅

const STATUS_MAP = { 1: '待审核', 2: '已通过', 3: '已拒绝' };

function getLabel(status) {

  return STATUS_MAP[status] ?? '未知';

}

```

  

## 4. 空值合并与可选链 (Nullish Coalescing & Optional Chaining) — 安全取值

  

```ts

// ❌

const name = user && user.profile && user.profile.name ? user.profile.name : '匿名';

  

// ✅

const name = user?.profile?.name ?? '匿名';

```

  

## 5. 策略模式 (Strategy Pattern) — 按类型分派行为

  

```ts

// ❌

function handle(type, data) {

  if (type === 'export') { /* 50行 */ }

  else if (type === 'import') { /* 50行 */ }

  else if (type === 'delete') { /* 50行 */ }

}

  

// ✅

const strategies = {

  export: (data) => { /* ... */ },

  import: (data) => { /* ... */ },

  delete: (data) => { /* ... */ },

};

function handle(type, data) {

  return strategies[type]?.(data);

}

```

  

## 6. 管道 / 链式调用 (Pipeline) — 数据流清晰可读

  

```ts

// ❌

const result = filter(sort(map(list, transform), compare), predicate);

  

// ✅

const result = list

  .map(transform)

  .sort(compare)

  .filter(predicate);

```

  

## 7. 单一出口变量 (Result Variable) — 复杂分支统一出口

  

```ts

// ✅

function calcPrice(order) {

  let price = order.basePrice;

  if (order.isVip) price *= 0.9;

  if (order.coupon) price -= order.coupon.value;

  return Math.max(price, 0);

}

```

  

## 8. 参数对象化 (Parameter Object) — 超过 3 个参数时用对象

  

```ts

// ❌

function createUser(name, age, email, role, department) {}

  

// ✅

function createUser({ name, age, email, role, department }) {}

```

  

## 9. 默认值兜底 (Default Values) — 减少防御性判断

  

```ts

// ❌

function fetch(options) {

  const page = options.page !== undefined ? options.page : 1;

  const size = options.size !== undefined ? options.size : 10;

}

  

// ✅

function fetch({ page = 1, size = 10 } = {}) {}

```

  

## 10. 断言式编程 (Assertion) — 让不可能的状态尽早暴露

  

```ts

function divide(a, b) {

  console.assert(b !== 0, '除数不能为 0');

  return a / b;

}

```

  

## 11. 提前计算 / 派生状态 (Derived State) — 避免冗余状态

  

```tsx

// ❌ 同时存 list 和 filteredList，容易不同步

const [list, setList] = useState([]);

const [filteredList, setFilteredList] = useState([]);

  

// ✅ filteredList 由 list 派生

const [list, setList] = useState([]);

const [keyword, setKeyword] = useState('');

const filteredList = useMemo(

  () => list.filter(item => item.name.includes(keyword)),

  [list, keyword],

);

```

  

## 12. 不可变更新 (Immutable Update) — 避免隐式副作用

  

```ts

// ❌

user.address.city = 'Shanghai';

  

// ✅

const updated = { ...user, address: { ...user.address, city: 'Shanghai' } };

```

  

---

  

## 速查总结

  

| 范式 | 解决的问题 |

|------|-----------|

| 卫语句 | 深层嵌套 |

| 解释性变量 | 复杂条件不可读 |

| 查表法 / 策略模式 | if-else / switch 爆炸 |

| 空值合并 + 可选链 | 冗长的空判断 |

| 管道链式 | 嵌套函数调用 |

| 参数对象化 + 默认值 | 参数过多、防御代码多 |

| 派生状态 | 冗余状态不同步 |

| 不可变更新 | 隐式副作用 |

  

> 核心原则：**减少嵌套、减少分支、减少重复、让意图自解释**。
