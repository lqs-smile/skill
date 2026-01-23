


## 数组去重


基本数据类型去重

```javascript
const arr = [1, 2, 2, 3, 3, 4];
const uniqueArr = [...new Set(arr)];
// 或
const uniqueArr = Array.from(new Set(arr));
```

通过Set自带的去重能力
适合：原始值或相同引用
不适合：对象数组



对象类型

reduce方式：空数组或者空对象 + 数组循环，就是针对普通循环依赖外部容器变量的改进版
基于传统方式升级

```javascript

const arr = [1, 2, 2, 3, 3, 4];
const uniqueArr = arr.reduce((acc, cur) => {
	// acc: 上一次返回的数据
	// cur： 当前数据项
    if (!acc.includes(cur)) {
        acc.push(cur);
    }
    return acc; // 当前数据与外部数据结合后返回给下一次循环使用
}, []);

```


传统方式：

```javascript
function unique(arr) {
    const result = [];
    for (let i = 0; i < arr.length; i++) {
        if (result.indexOf(arr[i]) === -1) {
            result.push(arr[i]);
        }
    }
    return result;
}
```


Map处理

性能最好推荐使用

Map:
1. 自带去重能力，是根据key来做去重的，所以key不能设置一样的
2. 
```javascript
const orders = [
    {userId: 1, productId: 101, date: '2024-01-01'},
    {userId: 2, productId: 102, date: '2024-01-01'},
    {userId: 1, productId: 101, date: '2024-01-01'}, // 重复
    {userId: 1, productId: 101, date: '2024-01-02'}, // 不同日期
];

// 根据userId一个属性去重
const uniqueUsers = Array.from(
    new Map(orders.map(item => [userId.id, item])).values()
);

// 根据userId + productId + date 去重
const uniqueOrders = Array.from(
    new Map(orders.map(order => [
        `${order.userId}-${order.productId}-${order.date}`,
        order
    ])).values()
);
```


封装成工具函数

```javascript
/**
 * 对象数组去重工具函数
 * @param {Array} array - 要去重的数组
 * @param {String|Function} keyGetter - 获取key的方式
 * @returns {Array} 去重后的数组
 */
function uniqueBy(array, keyGetter) {
    const map = new Map();
    
    for (const item of array) {
        const key = typeof keyGetter === 'function' 
            ? keyGetter(item)  // 主动调用你传进来的函数，并且接收返回值作为key
            : item[keyGetter]; // 获取key
        
        map.set(key, item);
    }
    
    return Array.from(map.values());
}

// 使用示例
const users = [
    {id: 1, name: 'Alice'},
    {id: 2, name: 'Bob'},
    {id: 1, name: 'Alice'},
];

// 按属性名
const unique1 = uniqueBy(users, 'id');

// 按自定义逻辑
const unique2 = uniqueBy(users, item => `${item.id}-${item.name[0]}`);

// 按多个属性
const unique3 = uniqueBy(users, item => `${item.id}-${item.name}`);
```



## 排序

Array.prototype.sorft

规则：a - b，从小到大，b - a 从大到小

数组原生排序方法

默认按字符串Unicode编码排序
```javascript

// 默认按字符串 Unicode 编码排序
const fruits = ['banana', 'apple', 'cherry'];
fruits.sort(); // ['apple', 'banana', 'cherry']

```

数字
```javascript
const numbers = [40, 100, 1, 5, 25];
numbers.sort((a, b) => a - b); // 升序: [1, 5, 25, 40, 100]
numbers.sort((a, b) => b - a); // 降序: [100, 40, 25, 5, 1]
```


对象
```javascript
// 对象数组排序
const users = [
  { name: 'John', age: 30 },
  { name: 'Jane', age: 25 },
  { name: 'Bob', age: 35 }
];
users.sort((a, b) => a.age - b.age); // 按年龄升序
```



## 多条件排序

```javascript
const data = [
  { name: 'Alice', age: 25, score: 85 },
  { name: 'Bob', age: 25, score: 90 },
  { name: 'Charlie', age: 30, score: 80 }
];

// 先按年龄升序，再按分数降序
data.sort((a, b) => {
  if (a.age !== b.age) {
    return a.age - b.age;
  }
  return b.score - a.score;
});
```


## 自定义排序

```javascript
// 按字符串长度排序
const strings = ['apple', 'kiwi', 'banana', 'pear'];
strings.sort((a, b) => a.length - b.length);

// 按日期排序
const dates = ['2024-01-15', '2023-12-01', '2024-01-01'];
dates.sort((a, b) => new Date(a) - new Date(b));
```