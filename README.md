# GOGOCODE

全网最简单易上手，可读性最强的 AST 处理工具！
官网：https://gogocode.io

# Install

```
    npm install gogocode
```

## 快速开始

对于下面的代码

```javascript
const code = `
  const moment = require('moment');
  var a = 1;
  const b = 2;
  function log (x, y = 'World') {
    console.log('a')
    console.log(a, x, y);
  }
`;
```

创建一个 AST 实例

```javascript
const $ = require('gogocode');
const AST = $(code);
```


小明想将 所有的 `a` 变量名替换为 `c`，只需要

```javascript
$(code).replace('a', 'c')
```


小明改主意了，只想把 `var a = 1` 里的变量名改为  `c`，需要两步：

1.取变量 a 的定义赋值，

```javascript
$(code).find('var a = 1');
```

2.将 a 变量名替换为 c，并输出整体代码

```javascript
$(code)
  .find('var a = 1')
  .attr('declarations.0.id.name', 'c')
  .root()
  .generate();
```

小明又改主意了，想把所有定义语句的 `a` 都改成  `c`，只需要在第一步取定义语句时写成：

```javascript
$(code).find('var a = $_$');
```

> 看到这里，你应该已经理解 `find` 的第一参相当于‘jquery 选择器’，而这里的选择器是你需要查找的代码片段，无论想要匹配多么复杂的代码都可以匹配到，其中 `$_$` 通配符可以匹配任意代码，代码选择器及通配符详细介绍 <a href="/zh/docs/specification/selector">看这里</a>

小明想试试将代码里的 var 改为 let，require 改为 import，他发现用 GoGoCode 居然可以像字符串的 replace 一样简单！

```javascript
$(code)
.replace('var $_$ = $_$', 'let $_$ = $_$');
.replace('const $_$ = require($_$)', 'import $_$ from $_$')
```

# API

## 熟悉的$ 实例化

调用 `$()` 即可将一段代码或一个ast节点构造为GoGoCode的核心AST实例

### $(code, options)
| 入参 |  | 说明 | 类型 | 举例 |
| --- | --- | --- | --- | --- |
| `code` |  | 需要被实例化的代码或AST节点 | string  NodePath  Node | `'var a = 1'` |
|  |  | 支持在代码中插入ast节点，再进行实例化，与`astFragment`入参搭配使用，ast节点占位形式为`$_$astNodeName$_$` | string | `Alert.show({ content: $_$content$_$ })` |
| `options` | `parseOptions` | 基本与babel/parse的options一致<br>唯一区别是解析html时需要定义为`{html: true}` |  | `{ plugins: ['jsx'] }` |
|  | `astFragment` | 需要插入到代码中的ast节点的映射 |  | `{ content: contentNode }` |

```typescript
$('var a = 1')

$(astNode)

$('<div></div>', { 
  parseOption: { plugins: ['jsx'] }
})

$(`Alert.show({content: $content$ , type: $type$ })`, { 
  astFragment: {
	  content: contentNode ,
    type: typeNode
  }
})


```


### $.loadFile(path, options)

与构造函数作用相同，第一个入参可以是文件路径，直接将文件内容实例化


## 节点获取


所有的节点获取操作都会返回一个新的AST实例，实例中可能包含多个AST节点路径，如`find()`、`siblings()`等，某些api返回的实例只会存在一个AST节点路径，如`next()`
### AST.find(selector, options)

| 入参 |  | 说明 | 类型 | 默认值 |
| --- | --- | --- | --- | --- |
| `selector` |  | 代码选择器，可以是代码也可以将代码中的部分内容挖空替换为通配符 | string | 无 |
| `options` | `ignoreSequence` | 匹配时是否忽略顺序<br>忽略顺序的情况：`{a:$_$}`匹配`{b:1, a:2}`<br>需要严格按照顺序匹配的情况：`function($_$, b){}` 匹配`function(a, b){}` | boolean | false |
|  | `parseOptions` |  同构造函数的`parseOptions` |  | |

当selector中存在 `$_$` 通配符时，返回的AST实例中存在 `match` 属性，也就是被 `$_$` 匹配到的AST节点
如：`$('var a = 1').find('var $_$ = $_$')`，得到的结构为：

<img style="width:800px; display:block" src="https://alp.alicdn.com/1614534004290-2222-1032.png"/>


其中对应的 `match` 结构为：

```
[{
    structure: { type: 'Identifier', name: 'a' ... } : Node,
    value: 'a'
}, {
    structure: { type: 'NumericLiteral', value: 1 ... } : Node,
    value: '1'
}]
```



### .parent(level)
获取某个父节点

| 入参 | 说明 | 类型 | 默认值 |
| --- | --- | --- | --- |
| `level` | 自内向外第n层父元素 | number | 0 |



### .parents()
获取所有父节

### .root()
获取根节点，对于js来说是`type`为'File'的节点，对于html来说是`nodeType`为'document'的节点
通常对AST进行操作之后需要获取root元素之后再输出


### .siblings()
获取所有兄弟节点

### .prev()
获取前一个节点

### .prevAll()
获取当前节点之前的同级节点

### .next()
获取后一个节点

### .nextAll()
获取当前节点之后的同级节点

### .each(callback)
以每一个匹配的元素作为上下文来执行一个函数。

| 入参 | 说明 | 类型 | 默认值 |
| --- | --- | --- | --- |
| `callback` | 对于每个匹配的元素所要执行的函数<br>执行函数时，会给函数传递当前节点`node`和`index` | function | 无 |

### .eq(index)
获取当前链式操作中第N个AST对象

| 入参 | 说明 | 类型 | 默认值 |
| --- | --- | --- | --- |
| `index` | 需要获取的AST对象的位置 | number | 0 |

### 节点操作

### .attr()

获取或修改AST节点的属性，入参可分三种情况：

1. 返回属性名称对应的节点或属性值

| 入参 | 说明 | 类型 | 默认值 | 举例 |
| --- | --- | --- | --- | --- |
| `attrName` | ast节点的属性名称，支持多层属性，通过.连接 | string | 无 | declarations<br>declarations.0.id.name |



2. 修改属性名称对应的节点或属性值

| 入参 | 说明 | 类型 | 举例 |
| --- | --- | --- | --- | 
| `attrName` | ast节点的属性名称，支持多层属性，通过.连接 | string | declarations <br>declarations.0.id.name |
| `attrValue` | 将第一个入参获取到的节点或属性修改为该入参 <br>注意：字符串不会被解析为ast节点而是直接替换原有属性 | node  string |  |



3. 修改多个属性名称对应的节点或属性值

| 入参 |  | 类型 | 默认值 | 举例 |
| --- | --- | --- | --- | --- |
| `attrMap` | `attrName` | string | 无 | declarations <br>declarations.0.id.name |
|  | `attrValue` | node | string | 无 |  |



```typescript
AST.attr('init', initNode)

AST.attr({ 
  init: initNode,
  'program.body.0.params.0.name': 'a'
})

AST.attr('program.body.0.params.0.name')
```
### .has(selector, options)
判断是否有某个子节点，返回值为boolean类型
入参同`.find()`

### .clone()
返回由当前节点深度复制的新节点


### .replace(selector, replacer)
在当前节点内部用`replacer`替换`selector`匹配到的代码，返回当前节点

| 入参 |  | 解释 | 类型 | 例 |
| --- | --- | --- | --- | --- |
| `selector` |  | 代码选择器，可以是代码也可以将代码中的部分内容挖空替换为通配符 | `string` | `var $_$ = $_$` |
| `replacer` |  | 替换代码，同代码选择器通配符顺序与selector的保持一致<br>也可以是确定的ast节点 | `string` | `node` | `let $_$ = $_$` |
| `options` | `ignoreSequence` | 匹配时是否忽略顺序 | object | 无 |
|  | `parseOptions` | 解析入参 | object | 无 |

### 
```typescript
AST.replace(`Component`, `module.exports = Magix.View.extend`);

AST.replace(
  `export default function calculateData($_$1){$_$2}`, 
  `function calculateData($_$2){$_$1}`
)

AST.replace(
  `navigateToOutside({url: $_$})`, 
  `jm.attachUrlParams($_$)`, 
  options: { ignoreSequence: true }
)
```
### .replaceBy(replacerAST)
用replacerAST替换当前节点，返回新节点

| 入参 | 类型 |
| --- | --- |
| `replacerAST` | AST <br>node <br> string |

### .after(ast)
在当前节点后面插入一个同级别的节点，返回当前节点

| 入参 | 类型 |
| --- | --- |
| `ast` | AST <br> node <br> string |

### .before()
在当前节点前面插入一个同级别的节点，返回当前节点

| 入参 | 类型 |
| --- | --- |
| `ast` | AST <br> node <br> string |

### .append(attr, ast)
在当前节点内部某个数组属性的末尾插入一个子节点，返回当前节点

| 入参 | 类型 |
| --- | --- |
| `attr` | 当前节点的数组属性名称 |
| `ast` | AST <br> node <br> string |

- 为什么需要传入`attr`？

因为某些节点中多个属性都为数组，如函数，存在入参params和函数体body两个数组子节点，必须通过attr来判断插入节点的位置

```typescript
AST
.find('function $_$() {}')
.append('params', 'b')
.prepend('body', 'b = b || 1;')
```
### .prepend()
在当前节点内部某个数组属性的首位插入一个子节点，返回当前节点

### .empty()
清空当前节点所有子节点，返回当前节点

### .remove()
移除当前节点，返回根节点

### .generate()
将AST对象输出为代码