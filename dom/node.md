---
title: DOM 模型概述
layout: page
category: dom
date: 2013-10-07
modifiedOn: 2014-05-18
---

## 基本概念

### DOM

DOM是JavaScript操作网页的接口，全称为“文档对象模型”（Document Object Model）。它的作用是将网页转为一个JavaScript对象，从而可以用脚本进行各种操作（比如增删内容）。

浏览器会根据DOM模型，将结构化文档（比如HTML和XML）解析成一系列的节点，再由这些节点组成一个树状结构（DOM Tree）。所有的节点和最终的树状结构，都有规范的对外接口。所以，DOM可以理解成网页的编程接口。DOM有自己的国际标准，目前的通用版本是[DOM 3](http://www.w3.org/TR/2004/REC-DOM-Level-3-Core-20040407/core.html)，下一代版本[DOM 4](http://www.w3.org/TR/dom/)正在拟定中。

严格地说，DOM不属于JavaScript，但是操作DOM是JavaScript最常见的任务，而JavaScript也是最常用于DOM操作的语言。本章介绍的就是JavaScript对DOM标准的实现和用法。

### 节点

DOM的最小组成单位叫做节点（node）。文档的树形结构（DOM树），就是由各种不同类型的节点组成。每个节点可以看作是文档树的一片叶子。

节点的类型有七种。

- `Document`：整个文档树的顶层节点
- `DocumentType`：`doctype`标签（比如`<!DOCTYPE html>`）
- `Element`：网页的各种HTML标签（比如`<body>`、`<a>`等）
- `Attribute`：网页元素的属性（比如`class="right"`）
- `Text`：标签之间或标签包含的文本
- `Comment`：注释
- `DocumentFragment`：文档的片段

这七种节点都属于浏览器原生提供的节点对象的派生对象，具有一些共同的属性和方法。

### 节点树

一个文档的所有节点，按照所在的层级，可以抽象成一种树状结构。这种树状结构就是DOM。

最顶层的节点就是`document`节点，它代表了整个文档。文档里面最高一层的HTML标签，一般是`<html>`，它构成树结构的根节点（root node），其他HTML标签节点都是它的下级。

除了根节点以外，其他节点对于周围的节点都存在三种关系。

- 父节点关系（parentNode）：直接的那个上级节点
- 子节点关系（childNodes）：直接的下级节点
- 同级节点关系（sibling）：拥有同一个父节点的节点

DOM提供操作接口，用来获取三种关系的节点。其中，子节点接口包括`firstChild`（第一个子节点）和`lastChild`（最后一个子节点）等属性，同级节点接口包括`nextSibling`（紧邻在后的那个同级节点）和`previousSibling`（紧邻在前的那个同级节点）属性。

## 特征相关的属性

所有节点对象都是浏览器内置的`Node`对象的实例，继承了`Node`属性和方法。这是所有节点的共同特征。

以下属性与节点对象本身的特征相关。

### Node.nodeName，Node.nodeType

`nodeName`属性返回节点的名称，`nodeType`属性返回节点类型的常数值。具体的返回值，可查阅下方的表格。

类型 | nodeName | nodeType
-----|----------|---------
ELEMENT_NODE | 大写的HTML元素名 | 1
ATTRIBUTE_NODE | 等同于Attr.name | 2
TEXT_NODE | #text | 3
COMMENT_NODE | #comment | 8
DOCUMENT_NODE | #document | 9
DOCUMENT_FRAGMENT_NODE | #document-fragment | 11
DOCUMENT_TYPE_NODE | 等同于DocumentType.name |10

以`document`节点为例，它的`nodeName`属性等于`#document`，`nodeType`属性等于9。

```javascript
document.nodeName // "#document"
document.nodeType // 9
```

如果是一个`<p>`节点，它的`nodeName`是`P`，`nodeType`是1。文本节点的`nodeName`是`#text`，`nodeType`是3。

通常来说，使用`nodeType`属性确定一个节点的类型，比较方便。

```javascript
document.querySelector('a').nodeType === 1
// true

document.querySelector('a').nodeType === Node.ELEMENT_NODE
// true
```

上面两种写法是等价的。

### Node.nodeValue

`Node.nodeValue`属性返回一个字符串，表示当前节点本身的文本值，该属性可读写。

由于只有Text节点、Comment节点、XML文档的CDATA节点有问文本值，因此只有这三类节点的`nodeValue`可以返回结果，其他类型的节点一律返回`null`。同样的，也只有这三类节点可以设置`nodeValue`属性的值。对于那些返回`null`的节点，设置`nodeValue`属性是无效的。

### Node.textContent

`Node.textContent`属性返回当前节点和它的所有后代节点的文本内容。

```javascript
// HTML代码为
// <div id="divA">This is <span>some</span> text</div>

document.getElementById('divA').textContent
// This is some text
```

上面代码的`textContent`属性，自动忽略当前节点内部的HTML标签，返回所有文本内容。

该属性是可读写的，设置该属性的值，会用一个新的文本节点，替换所有原来的子节点。它还有一个好处，就是自动对HTML标签转义。这很适合用于用户提供的内容。

```javascript
document.getElementById('foo').textContent = '<p>GoodBye!</p>';
```

上面代码在插入文本时，会将p标签解释为文本，即&amp;lt;p&amp;gt;，而不会当作标签处理。

对于Text节点和Comment节点，该属性的值与`nodeValue`性相同。对于其他类型的节点，该属性会将每个子节点的内容连接在一起返回，但是不包括Comment节点。如果一个节点没有子节点，则返回空字符串。

`document`节点和`doctype`节点的`textContent`属性为`null`。如果要读取整个文档的内容，可以使用`document.documentElement.textContent`。

### Node.baseURI

`Node.baseURI`属性返回一个字符串，表示当前网页的绝对路径。如果无法取到这个值，则返回`null`。浏览器根据这个属性，计算网页上的相对路径的URL。该属性为只读。

```javascript
// 当前网页的网址为
// http://www.example.com/index.html
document.baseURI
// "http://www.example.com/index.html"
```

不同节点都可以调用这个属性（比如`document.baseURI`和`element.baseURI`），通常它们的值是相同的。

该属性的值一般由当前网址的URL（即`window.location`属性）决定，但是可以使用HTML的`<base>`标签，改变该属性的值。

```html
<base href="http://www.example.com/page.html">
<base target="_blank" href="http://www.example.com/page.html">
```

设置了以后，`baseURI`属性就返回`<base>`标签设置的值。

## 相关节点的属性

以下属性返回当前节点的相关节点。

### Node.ownerDocument

`Node.ownerDocument`属性返回当前节点所在的顶层文档对象，即`document`对象。

```javascript
var d = p.ownerDocument;
d === document // true
```

`document`对象本身的`ownerDocument`属性，返回`null`。

### Node.nextSibling

`Node.nextSibling`属性返回紧跟在当前节点后面的第一个同级节点。如果当前节点后面没有同级节点，则返回null。注意，该属性还包括文本节点和评论节点。因此如果当前节点后面有空格，该属性会返回一个文本节点，内容为空格。

```javascript
var el = document.getElementById('div-01').firstChild;
var i = 1;

while (el) {
  console.log(i + '. ' + el.nodeName);
  el = el.nextSibling;
  i++;
}
```

上面代码遍历`div-01`节点的所有子节点。

下面两个表达式指向同一个节点。

```javascript
document.childNodes[0].childNodes[1]
document.firstChild.firstChild.nextSibling
```

### Node.previousSibling

previousSibling属性返回当前节点前面的、距离最近的一个同级节点。如果当前节点前面没有同级节点，则返回null。

```javascript
// html代码如下
// <a><b1 id="b1"/><b2 id="b2"/></a>

document.getElementById("b1").previousSibling // null
document.getElementById("b2").previousSibling.id // "b1"
```

对于当前节点前面有空格，则`previousSibling`属性会返回一个内容为空格的文本节点。

### Node.parentNode

`parentNode`属性返回当前节点的父节点。对于一个节点来说，它的父节点只可能是三种类型：`element`节点、`document`节点和`documentfragment`节点。

下面代码是如何从父节点移除指定节点。

```javascript
if (node.parentNode) {
  node.parentNode.removeChild(node);
}
```

对于document节点和documentfragment节点，它们的父节点都是null。另外，对于那些生成后还没插入DOM树的节点，父节点也是null。

### Node.parentElement

parentElement属性返回当前节点的父Element节点。如果当前节点没有父节点，或者父节点类型不是Element节点，则返回null。

```javascript
if (node.parentElement) {
  node.parentElement.style.color = "red";
}
```

上面代码设置指定节点的父Element节点的CSS属性。

在IE浏览器中，只有Element节点才有该属性，其他浏览器则是所有类型的节点都有该属性。

### Node.childNodes

childNodes属性返回一个NodeList集合，成员包括当前节点的所有子节点。注意，除了HTML元素节点，该属性返回的还包括Text节点和Comment节点。如果当前节点不包括任何子节点，则返回一个空的NodeList集合。由于NodeList对象是一个动态集合，一旦子节点发生变化，立刻会反映在返回结果之中。

```javascript
var ulElementChildNodes = document.querySelector('ul').childNodes;
```

### Node.firstChild，Node.lastChild

`firstChild`属性返回当前节点的第一个子节点，如果当前节点没有子节点，则返回`null`（注意，不是`undefined`）。

```html
<p id="para-01"><span>First span</span></p>

<script type="text/javascript">
  console.log(
    document.getElementById('para-01').firstChild.nodeName
  ) // "span"
</script>
```

上面代码中，`p`元素的第一个子节点是`span`元素。

注意，`firstChild`返回的除了HTML元素子节点，还可能是文本节点或评论节点。

```html
<p id="para-01">
  <span>First span</span>
</p>

<script type="text/javascript">
  console.log(
    document.getElementById('para-01').firstChild.nodeName
  ) // "#text"
</script>
```

上面代码中，`p`元素与`span`元素之间有空白字符，这导致`firstChild`返回的是文本节点。

`Node.lastChild`属性返回当前节点的最后一个子节点，如果当前节点没有子节点，则返回null。

## 节点对象的方法

### appendChild()，hasChildNodes()

以下方法与子节点相关。

**（1）appendChild()**

`appendChild`方法接受一个节点对象作为参数，将其作为最后一个子节点，插入当前节点。

```javascript
var p = document.createElement('p');
document.body.appendChild(p);
```

如果参数节点是文档中现有的其他节点，appendChild方法会将其从原来的位置，移动到新位置。

**（2）hasChildNodes()**

`hasChildNodes`方法返回一个布尔值，表示当前节点是否有子节点。

```javascript
var foo = document.getElementById("foo");

if ( foo.hasChildNodes() ) {
  foo.removeChild( foo.childNodes[0] );
}
```

上面代码表示，如果foo节点有子节点，就移除第一个子节点。

`hasChildNodes`方法结合`firstChild`属性和`nextSibling`属性，可以遍历当前节点的所有后代节点。

```javascript
function DOMComb(parent, callback) {
  if (parent.hasChildNodes()) {
    for (var node = parent.firstChild; node; node = node.nextSibling) {
      DOMComb(node, callback);
    }
  }
  callback.call(parent);
}
```

上面代码的`DOMComb`函数的第一个参数是某个指定的节点，第二个参数是回调函数。这个回调函数会依次作用于指定节点，以及指定节点的所有后代节点。

```javascript
function printContent() {
  if (this.nodeValue) {
    console.log(this.nodeValue);
  }
}

DOMComb(document.body, printContent);
```

### cloneNode()，insertBefore()，removeChild()，replaceChild()

下面方法与节点操作有关。

**（1）cloneNode()**

cloneNode方法用于克隆一个节点。它接受一个布尔值作为参数，表示是否同时克隆子节点，默认是false，即不克隆子节点。

{% highlight javascript %}

var cloneUL = document.querySelector('ul').cloneNode(true);

{% endhighlight %}

需要注意的是，克隆一个节点，会拷贝该节点的所有属性，但是会丧失addEventListener方法和on-属性（即`node.onclick = fn`），添加在这个节点上的事件回调函数。

克隆一个节点之后，DOM树有可能出现两个有相同ID属性（即`id="xxx"`）的HTML元素，这时应该修改其中一个HTML元素的ID属性。

**（2）insertBefore()**

insertBefore方法用于将某个节点插入当前节点的指定位置。它接受两个参数，第一个参数是所要插入的节点，第二个参数是当前节点的一个子节点，新的节点将插在这个节点的前面。该方法返回被插入的新节点。

{% highlight javascript %}

var text1 = document.createTextNode('1');
var li = document.createElement('li');
li.appendChild(text1);

var ul = document.querySelector('ul');
ul.insertBefore(li,ul.firstChild);

{% endhighlight %}

上面代码在ul节点的最前面，插入一个新建的li节点。

如果insertBefore方法的第二个参数为null，则新节点将插在当前节点的最后位置，即变成最后一个子节点。

将新节点插在当前节点的最前面（即变成第一个子节点），可以使用当前节点的firstChild属性。

```javascript
parentElement.insertBefore(newElement, parentElement.firstChild);
```

上面代码中，如果当前节点没有任何子节点，`parentElement.firstChild`会返回null，则新节点会插在当前节点的最后，等于是第一个子节点。

由于不存在insertAfter方法，如果要插在当前节点的某个子节点后面，可以用insertBefore方法结合nextSibling属性模拟。

```javascript
parentDiv.insertBefore(s1, s2.nextSibling);
```

上面代码可以将s1节点，插在s2节点的后面。如果s2是当前节点的最后一个子节点，则`s2.nextSibling`返回null，这时s1节点会插在当前节点的最后，变成当前节点的最后一个子节点，等于紧跟在s2的后面。

**（3）removeChild()**

removeChild方法接受一个子节点作为参数，用于从当前节点移除该节点。它返回被移除的节点。

{% highlight javascript %}

var divA = document.getElementById('A');
divA.parentNode.removeChild(divA);

{% endhighlight %}

上面代码是如何移除一个指定节点。

下面是如何移除当前节点的所有子节点。

```javascript
var element = document.getElementById("top");
while (element.firstChild) {
  element.removeChild(element.firstChild);
}
```

被移除的节点依然存在于内存之中，但是不再是DOM的一部分。所以，一个节点移除以后，依然可以使用它，比如插入到另一个节点。

**（4）replaceChild()**

replaceChild方法用于将一个新的节点，替换当前节点的某一个子节点。它接受两个参数，第一个参数是用来替换的新节点，第二个参数将要被替换走的子节点。它返回被替换走的那个节点。

```javascript
replacedNode = parentNode.replaceChild(newChild, oldChild);
```

下面是一个例子。

{% highlight javascript %}

var divA = document.getElementById('A');
var newSpan = document.createElement('span');
newSpan.textContent = 'Hello World!';
divA.parentNode.replaceChild(newSpan,divA);

{% endhighlight %}

上面代码是如何替换指定节点。

### contains()，compareDocumentPosition()，isEqualNode()

下面方法用于节点的互相比较。

**（1）contains()**

contains方法接受一个节点作为参数，返回一个布尔值，表示参数节点是否为当前节点的后代节点。

{% highlight javascript %}

document.body.contains(node)

{% endhighlight %}

上面代码检查某个节点，是否包含在当前文档之中。

注意，如果将当前节点传入contains方法，会返回true。虽然从意义上说，一个节点不应该包含自身。

```javascript
nodeA.contains(nodeA) // true
```

**（2）compareDocumentPosition()**

compareDocumentPosition方法的用法，与contains方法完全一致，返回一个7个比特位的二进制值，表示参数节点与当前节点的关系。

二进制值 | 数值 | 含义
---------|------|-----
000000 | 0 | 两个节点相同
000001 | 1 | 两个节点不在同一个文档（即有一个节点不在当前文档）
000010 | 2 | 参数节点在当前节点的前面
000100 | 4 | 参数节点在当前节点的后面
001000 | 8 | 参数节点包含当前节点
010000 | 16 | 当前节点包含参数节点
100000 | 32 | 浏览器的私有用途

```javascript
// HTML代码为
// <div id="writeroot">
//   <form>
//     <input id="test" />
//   </form>
// </div>

var x = document.getElementById('writeroot');
var y = document.getElementById('test');

x.compareDocumentPosition(y) // 20
y.compareDocumentPosition(x) // 10
```

上面代码中，节点x包含节点y，而且节点y在节点x的后面，所以第一个compareDocumentPosition方法返回20（010100），第二个compareDocumentPosition方法返回10（0010010）。

由于compareDocumentPosition返回值的含义，定义在每一个比特位上，所以如果要检查某一种特定的含义，就需要使用比特位运算符。

```javascript
var head = document.head;
var body = document.body;
if (head.compareDocumentPosition(body) & 4) {
  console.log("文档结构正确");
} else {
  console.log("<head> 不能在 <body> 前面");
}
```

上面代码中，compareDocumentPosition的返回值与4（又称掩码）进行与运算（&），得到一个布尔值，表示head是否在body前面。

在这个方法的基础上，可以部署一些特定的函数，检查节点的位置。

```javascript
Node.prototype.before = function (arg) {
  return !!(this.compareDocumentPosition(arg) & 2)
}

nodeA.before(nodeB)
```

上面代码在Node对象上部署了一个before方法，返回一个布尔值，表示参数节点是否在当前节点的前面。

**（3）isEqualNode()**

isEqualNode方法返回一个布尔值，用于检查两个节点是否相等。所谓相等的节点，指的是两个节点的类型相同、属性相同、子节点相同。

```javascript
var targetEl = document.getElementById("targetEl");
var firstDiv = document.getElementsByTagName("div")[0];

targetEl.isEqualNode(firstDiv)
```

### normalize()

normailize方法用于清理当前节点内部的所有Text节点。它会去除空的文本节点，并且将毗邻的文本节点合并成一个。

```javascript
var wrapper = document.createElement("div");

wrapper.appendChild(document.createTextNode("Part 1 "));
wrapper.appendChild(document.createTextNode("Part 2 "));

wrapper.childNodes.length // 2

wrapper.normalize();

wrapper.childNodes.length // 1
```

上面代码使用normalize方法之前，wrapper节点有两个Text子节点。使用normalize方法之后，两个Text子节点被合并成一个。

该方法是`Text.splitText`的逆方法，可以查看《Text节点》章节，了解更多内容。

## NodeList对象，HTMLCollection对象

节点对象都是单个节点，但是有时会需要一种数据结构，能够容纳多个节点。DOM提供两种集合对象，用于实现这种节点的集合：`NodeList`和`HTMLCollection`。

这两个对象都是构造函数。

```javascript
typeof NodeList // "function"
typeof HTMLCollection // "function"
```

但是，一般不把它们当作函数使用，甚至都没有直接使用它们的场合。主要是许多DOM属性和方法，返回的结果是`NodeList`实例或`HTMLCollection`实例，所以一般只使用它们的实例。

### NodeList对象

`NodeList`实例对象是一个类似数组的对象，它的成员是节点对象。`Node.childNodes`、`document.querySelectorAll()`返回的都是`NodeList`实例对象。

```javascript
document.childNodes instanceof NodeList // true
```

`NodeList`实例对象可能是动态集合，也可能是静态集合。所谓动态集合就是一个活的集合，DOM树删除或新增一个相关节点，都会立刻反映在NodeList接口之中。`Node.childNodes`返回的，就是一个动态集合。

```javascript
var parent = document.getElementById('parent');
parent.childNodes.length // 2
parent.appendChild(document.createElement('div'));
parent.childNodes.length // 3
```

上面代码中，`parent.childNodes`返回的是一个`NodeList`实例对象。当`parent`节点新增一个子节点以后，该对象的成员个数就增加了1。

`document.querySelectorAll`方法返回的是一个静态集合。DOM内部的变化，并不会实时反映在该方法的返回结果之中。

`NodeList`接口实例对象提供`length`属性和数字索引，因此可以像数组那样，使用数字索引取出每个节点，但是它本身并不是数组，不能使用`pop`或`push`之类数组特有的方法。

```javascript
// 数组的继承链
myArray --> Array.prototype --> Object.prototype --> null

// NodeList的继承链
myNodeList --> NodeList.prototype --> Object.prototype --> null
```

从上面的继承链可以看到，`NodeList`实例对象并不继承`Array.prototype`，因此不具有数组的方法。如果要在`NodeList`实例对象使用数组方法，可以将`NodeList`实例转为真正的数组。

```javascript
var div_list = document.querySelectorAll('div');
var div_array = Array.prototype.slice.call(div_list);
```

注意，采用上面的方法将`NodeList`实例转为真正的数组以后，`div_array`就是一个静态集合了，不再能动态反映DOM的变化。

另一种方法是通过`call`方法，间接在`NodeList`实例上使用数组方法。

```javascript
var forEach = Array.prototype.forEach;

forEach.call(element.childNodes, function(child){
  child.parentNode.style.color = '#0F0';
});
```

上面代码让数组的`forEach`方法在`NodeList`实例对象上调用。

遍历`NodeList`实例对象的首选方法，是使用`for`循环。

```javascript
for (var i = 0; i < myNodeList.length; ++i) {
  var item = myNodeList[i];
}
```

不要使用`for...in`循环去遍历`NodeList`实例对象，因为`for...in`循环会将非数字索引的`length`属性和下面要讲到的`item`方法，也遍历进去，而且不保证各个成员遍历的顺序。

ES6新增的`for...of`循环，也可以正确遍历`NodeList`实例对象。

```javascript
var list = document.querySelectorAll('input[type=checkbox]');
for (var item of list) {
  item.checked = true;
}
```

`NodeList`实例对象的`item`方法，接受一个数字索引作为参数，返回该索引对应的成员。如果取不到成员，或者索引不合法，则返回`null`。

```javascript
nodeItem = nodeList.item(index)

// 实例
var divs = document.getElementsByTagName("div");
var secondDiv = divs.item(1);
```

上面代码中，由于数字索引从零开始计数，所以取出第二个成员，要使用数字索引`1`。

所有类似数组的对象，都可以使用方括号运算符取出成员，所以一般情况下，都是使用下面的写法，而不使用`item`方法。

```javascript
nodeItem = nodeList[index]
```

### HTMLCollection对象

`HTMLCollection`实例对象与`NodeList`实例对象类似，也是节点的集合，返回一个类似数组的对象。`document.links`、`docuement.forms`、`document.images`等属性，返回的都是`HTMLCollection`实例对象。

`HTMLCollection`与`NodeList`的区别有以下几点。

（1）`HTMLCollection`实例对象的成员只能是`Element`节点，`NodeList`实例对象的成员可以包含其他节点。

（2）`HTMLCollection`实例对象都是动态集合，节点的变化会实时反映在集合中。`NodeList`实例对象可以是静态集合。

（3）`HTMLCollection`实例对象可以用`id`属性或`name`属性引用节点元素，`NodeList`只能使用数字索引引用。

`HTMLCollection`实例的`item`方法，可以根据成员的位置参数（从`0`开始），返回该成员。如果取不到成员或数字索引不合法，则返回`null`。

```javascript
var c = document.images;
var img1 = c.item(1);

// 等价于下面的写法
var img1 = c[1];
```

`HTMLCollection`实例的`namedItem`方法根据成员的`ID`属性或`name`属性，返回该成员。如果没有对应的成员，则返回`null`。这个方法是`NodeList`实例不具有的。

```javascript
// HTML代码为
// <form id="myForm"></form>
var elem = document.forms.namedItem('myForm');
// 等价于下面的写法
var elem = document.forms['myForm'];
```

由于`item`方法和`namedItem`方法，都可以用方括号运算符代替，所以建议一律使用方括号运算符。

## ParentNode接口，ChildNode接口

不同的节点除了继承Node接口以外，还会继承其他接口。ParentNode接口用于获取当前节点的Element子节点，ChildNode接口用于处理当前节点的子节点（包含但不限于Element子节点）。

### ParentNode接口

ParentNode接口用于获取Element子节点。Element节点、Document节点和DocumentFragment节点，部署了ParentNode接口。凡是这三类节点，都具有以下四个属性，用于获取Element子节点。

**（1）children**

children属性返回一个动态的HTMLCollection集合，由当前节点的所有Element子节点组成。

下面代码遍历指定节点的所有Element子节点。

```javascript
if (el.children.length) {
  for (var i = 0; i < el.children.length; i++) {
    // ...
  }
}
```

**（2）firstElementChild**

firstElementChild属性返回当前节点的第一个Element子节点，如果不存在任何Element子节点，则返回null。

```javascript
document.firstElementChild.nodeName
// "HTML"
```

上面代码中，document节点的第一个Element子节点是&lt;HTML&gt;。

**（3）lastElementChild**

lastElementChild属性返回当前节点的最后一个Element子节点，如果不存在任何Element子节点，则返回null。

```javascript
document.lastElementChild.nodeName
// "HTML"
```

上面代码中，document节点的最后一个Element子节点是&lt;HTML&gt;。

**（4）childElementCount**

childElementCount属性返回当前节点的所有Element子节点的数目。

### ChildNode接口

ChildNode接口用于处理子节点（包含但不限于Element子节点）。Element节点、DocumentType节点和CharacterData接口，部署了ChildNode接口。凡是这三类节点（接口），都可以使用下面四个方法。但是现实的情况是，除了第一个remove方法，目前没有浏览器支持后面三个方法。

**（1）remove()**

remove方法用于移除当前节点。

```javascript
el.remove()
```

上面方法在DOM中移除了el节点。注意，调用这个方法的节点，是被移除的节点本身，而不是它的父节点。

**（2）before()**

before方法用于在当前节点的前面，插入一个同级节点。如果参数是节点对象，插入DOM的就是该节点对象；如果参数是文本，插入DOM的就是参数对应的文本节点。

**（3）after()**

after方法用于在当前节点的后面，插入一个同级节点。如果参数是节点对象，插入DOM的就是该节点对象；如果参数是文本，插入DOM的就是参数对应的文本节点。

**（4）replaceWith()**

replaceWith方法使用参数指定的节点，替换当前节点。如果参数是节点对象，替换当前节点的就是该节点对象；如果参数是文本，替换当前节点的就是参数对应的文本节点。

## html元素

`html`元素是网页的根元素，`document.documentElement`就指向这个元素。

**（1）clientWidth属性，clientHeight属性**

这两个属性返回视口（viewport）的大小，单位为像素。所谓“视口”，是指用户当前能够看见的那部分网页的大小

`document.documentElement.clientWidth`和`document.documentElement.clientHeight`，基本上与`window.innerWidth`和`window.innerHeight`同义。只有一个区别，前者不将滚动条计算在内（很显然，滚动条和工具栏会减小视口大小），而后者包括了滚动条的高度和宽度。

**（2）offsetWidth属性，offsetHeight属性**

这两个属性返回html元素的宽度和高度，即网页的总宽度和总高度。

### tabindex属性

`tabindex`属性用来指定，当前HTML元素节点是否被tab键遍历，以及遍历的优先级。

```javascript
var b1 = document.getElementById("button1");

b1.tabIndex = 1;
```

如果 tabindex = -1 ，tab键跳过当前元素。

如果 tabindex = 0 ，表示tab键将遍历当前元素。如果一个元素没有设置tabindex，默认值就是0。

如果 tabindex 大于0，表示tab键优先遍历。值越大，就表示优先级越大。

### 页面位置相关属性

**（1）offsetParent属性、offsetTop属性和offsetLeft属性**

这三个属性提供Element对象在页面上的位置。

- offsetParent：当前HTML元素的最靠近的、并且CSS的position属性不等于static的父元素。
- offsetTop：当前HTML元素左上角相对于offsetParent的垂直位移。
- offsetLeft：当前HTML元素左上角相对于offsetParent的水平位移。

如果Element对象的父对象都没有将position属性设置为非static的值（比如absolute或relative），则offsetParent属性指向body元素。另外，计算offsetTop和offsetLeft的时候，是从边框的左上角开始计算，即Element对象的border宽度不计入offsetTop和offsetLeft。

### style属性

style属性用来读写页面元素的行内CSS属性，详见本章《CSS操作》一节。

### Element对象的方法

**（1）选择子元素的方法**

Element对象也部署了document对象的4个选择子元素的方法，而且用法完全一样。

- querySelector方法
- querySelectorAll方法
- getElementsByTagName方法
- getElementsByClassName方法

上面四个方法只用于选择Element对象的子节点。因此，可以采用链式写法来选择子节点。

{% highlight javascript %}

document.getElementById('header').getElementsByClassName('a')

{% endhighlight %}

各大浏览器对这四个方法都支持良好，IE的情况如下：IE 6开始支持getElementsByTagName，IE 8开始支持querySelector和querySelectorAll，IE 9开始支持getElementsByClassName。

**（2）elementFromPoint方法**

该方法用于选择在指定坐标的最上层的Element对象。

{% highlight javascript %}

document.elementFromPoint(50,50)

{% endhighlight %}

上面代码了选中在(50,50)这个坐标的最上层的那个HTML元素。

**（3）HTML元素的属性相关方法**

- hasAttribute()：返回一个布尔值，表示Element对象是否有该属性。
- getAttribute()
- setAttribute()
- removeAttribute()

**（4）matchesSelector方法**

该方法返回一个布尔值，表示Element对象是否符合某个CSS选择器。

{% highlight javascript %}

document.querySelector('li').matchesSelector('li:first-child')

{% endhighlight %}

这个方法需要加上浏览器前缀，需要写成mozMatchesSelector()、webkitMatchesSelector()、oMatchesSelector()、msMatchesSelector()。

**（5）focus方法**

focus方法用于将当前页面的焦点，转移到指定元素上。

```javascript
document.getElementById('my-span').focus();
```

### table元素

表格有一些特殊的DOM操作方法。

- **insertRow()**：在指定位置插入一个新行（tr）。
- **deleteRow()**：在指定位置删除一行（tr）。
- **insertCell()**：在指定位置插入一个单元格（td）。
- **deleteCell()**：在指定位置删除一个单元格（td）。
- **createCaption()**：插入标题。
- **deleteCaption()**：删除标题。
- **createTHead()**：插入表头。
- **deleteTHead()**：删除表头。

下面是使用JavaScript生成表格的一个例子。

```javascript
var table = document.createElement('table');
var tbody = document.createElement('tbody');
table.appendChild(tbody);

for (var i = 0; i <= 9; i++) {
  var rowcount = i + 1;
  tbody.insertRow(i);
  tbody.rows[i].insertCell(0);
  tbody.rows[i].insertCell(1);
  tbody.rows[i].insertCell(2);
  tbody.rows[i].cells[0].appendChild(document.createTextNode('Row ' + rowcount + ', Cell 1'));
  tbody.rows[i].cells[1].appendChild(document.createTextNode('Row ' + rowcount + ', Cell 2'));
  tbody.rows[i].cells[2].appendChild(document.createTextNode('Row ' + rowcount + ', Cell 3'));
}

table.createCaption();
table.caption.appendChild(document.createTextNode('A DOM-Generated Table'));

document.body.appendChild(table);
```

这些代码相当易读，其中需要注意的就是`insertRow`和`insertCell`方法，接受一个表示位置的参数（从0开始的整数）。

`table`元素有以下属性：

- **caption**：标题。
- **tHead**：表头。
- **tFoot**：表尾。
- **rows**：行元素对象，该属性只读。
- **rows.cells**：每一行的单元格对象，该属性只读。
- **tBodies**：表体，该属性只读。

## 参考链接

- Louis Lazaris, [Thinking Inside The Box With Vanilla JavaScript](http://coding.smashingmagazine.com/2013/10/06/inside-the-box-with-vanilla-javascript/)
- David Walsh, [HTML5 classList API](http://davidwalsh.name/classlist)
- Derek Johnson, [The classList API](http://html5doctor.com/the-classlist-api/)
- Mozilla Developer Network, [element.dataset API](http://davidwalsh.name/element-dataset)
- David Walsh, [The element.dataset API](http://davidwalsh.name/element-dataset) 
