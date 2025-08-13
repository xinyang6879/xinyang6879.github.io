---
title: 解析 attrs
time: 2022-12-8 20:55
categories: vue
tag: vue
---

# 【AST】-解析 attrs

## 考虑情况

- 普通情况

```js
class="a"
```

- 空格情况

```js
class="a b c"
```

- 无引号情况

```js
data = 1;
```

- 无赋值情况

```js
data - a;
```

## 方案

### 前情

1、attrs 作为一整串字符串传入，因为第二种(空格)情况的存在，不能通过空格进行数组分割

2、存在第二种(无引号)的情况，不能完全通过引号作为标志位进行分割

### 结论

- 以引号和等于同时作为标志位，对字符串进行切分。引号为第一准则，等号为附用

- 在切分 key 与 value 时，以等号作为分割符，但还需要检查 key 中是否存在空格，如果有空格，则表示存在第四种(无赋值情况)情况

## 代码

```js
function getKeyValue(strs) {
  // 存放key-value/attr字符的数组
  let res = [];
  // 是否是有双引号的标志位
  let is_start = false;
  // 是否是有等号的标志位
  let is_equal = false;
  // 存放每个key-value的字符
  let str = "";
  // 遍历整个attrs字符串
  for (let i = 0; i < strs.length; i++) {
    str += strs[i];
    // 如果当前的字符是引号，那么表示一个attr值的开始或者结束，class="(开始)a"(结束)
    if (strs[i] === `"`) {
      // 如果之前已经出现了引号，那么表示这个引号是attr的结束位，class="a"(结束)
      if (is_start) {
        // 那么就需要将当前收集的字符存到attr的数组中
        str = str.replaceAll('"', "");
        res.push(str);
        // 需要重新收集，那么一切回到初始
        is_start = false;
        is_equal = false;
        str = "";
      } else {
        // 如果之前没有引号，那么表示这个引号是attr值的开始位，class="(开始)a"
        is_start = true;
      }
    }
    // 如果有=，但是没有"表示开始，那么就表示是x=1的形式
    if (strs[i] === "=" && !is_start) {
      is_equal = true;
    }
    // 如果当时是空格，并且是出现过等号且没有出现过引号，那么表示是data=1的情况
    if (strs[i] === " " && is_equal && !is_start) {
      // 那么就需要将当前收集的字符存到attr的数组中
      res.push(str);
      // 需要重新收集，那么一切回到初始
      is_start = false;
      is_equal = false;
      str = "";
    }
  }
  // 数据已经遍历完成，但是str还没有结束
  if (str) {
    res.push(str);
  }
  const data = res.reduce((pre, cur) => {
    // 通过等号将attr的str分隔开
    let [key, val = ""] = cur.split("=");
    // 去掉key的左右空格
    key = key.trim();
    // 如果key还存在空格，那么就表示是data-b data=1的情况
    if (key.includes(" ")) {
      // 需要将key以空格分割开
      const keys = key.split(" ");
      // 将在最后一个key之前的key都放入attrs数组里面
      for (let i = 0; i < keys.length - 1; i++) {
        pre.push({ key: keys[i].trim(), val: "" });
      }
      // 最后一个key作为对于的value值
      key = keys[keys.length - 1].trim();
    }
    // 在attrs里面添加attr的key及value
    pre.push({ key: key.trim(), val: val.trim() });
    return pre;
  }, []);
  return data;
}

getKeyValue( ` class="a b  c "   id="d " data-b data-c data=1   data-a="111" data-d`);

// 输出
// res(keyvalue)数组：
 [' class=a b  c ', '   id=d ', ' data-b data-c data=1 ', '  data-a=111', ' data-d']
// data结果：
[
    {key: 'class', val: 'a b  c'},
    {key: 'id', val: 'd'},
    {key: 'data-b', val: ''},
    {key: 'data-c', val: ''},
    {key: 'data', val: '1'},
    {key: 'data-a', val: '111'},
    {key: 'data-d', val: ''},
]
```
