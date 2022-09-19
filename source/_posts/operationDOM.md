---
title: 超详细的DOM操作
excerpt: DOM的增删改查
categories:
- 技术文章
tags:
- js
---

## 节点创建型API（增）
### createElement
createElement通过传入指定的一个标签名来创建一个元素，如果传入的标签名是一个未知的，则会创建一个自定义的标签，注意：IE8以下浏览器不支持自定义标签
```javascript
var div = document.createElement("div");
```
使用createElement要注意：通过createElement创建的元素并不属于html文档，它只是创建出来，并未添加到html文档中，要调用appendChild或insertBefore等方法将其添加到HTML文档树中;

### createTextNode
createTextNode用来创建一个文本节点，用法如下
```javascript
var textNode = document.createTextNode("一个TextNode");
```
createTextNode接收一个参数，这个参数就是文本节点中的文本，和createElement一样，创建后的文本节点也只是独立的一个节点，同样需要append Child将其添加到HTML文档树中

### cloneNode
cloneNode是用来返回调用方法的节点的一个副本，它接收一个bool参数，用来表示是否复制子元素，使用如下：
```javascript
var parent = document.getElementById("parentElement"); 
var parent2 = parent.cloneNode(true);// 传入true
parent2.id = "parent2";
```
这段代码通过cloneNode复制了一份parent元素，其中cloneNode的参数为true，表示parent的子节点也被复制，如果传入false，则表示只复制了parent节点
```html
<body>
    <div id="parent">
        我是父元素的文本
        <br/>
        <span>
        我是子元素
        </span>
    </div>
    <button id="btnCopy">复制</button>
    <script>
        var parent = document.getElementById("parent");
        //点击id＝"btnCopy"点击执行执行函数（函数内是一个元素的拷贝）  
        document.getElementById("btnCopy").onclick = function(){
            var parent2 = parent.cloneNode(true);
            parent2.id = "parent2";
            document.body.appendChild(parent2);
        }
    </script>
</body>
```
这段代码很简单，主要是绑定button事件，事件内容是复制了一个parent，修改其id，然后添加到文档中
这里有几点要注意：
- 和createElement一样，cloneNode创建的节点只是游离有html文档外的节点，要调用appendChild方法才能添加到文档树中
- 如果复制的元素有id，则其副本同样会包含该id，由于id具有唯一性，所以在复制节点后必须要修改其id
- 调用接收的bool参数最好传入，如果不传入该参数，不同浏览器对其默认值的处理可能不同

除此之外，我们还有一个需要注意的点：如果被复制的节点绑定了事件，则副本也会跟着绑定该事件吗？这里要分情况讨论：
- 如果是通过addEventListener或者比如onclick进行绑定事件，则副本节点不会绑定该事件
- 如果是内联方式绑定比如`<div onclick="showParent()"></div>`这样的话，副本节点同样会触发事件

### createDocumentFragment
createDocumentFragment方法用来创建一个DocumentFragment。在前面我们说到DocumentFragment表示一种轻量级的文档，它的作用主要是存储临时的节点用来准备添加到文档中
createDocumentFragment方法主要是用于添加大量节点到文档中时会使用到。假设要循环一组数据，然后创建多个节点添加到文档中
```html
<ul id="list"></ul>
<input type="button" value="添加多项" id="btnAdd" />

document.getElementById("btnAdd").onclick = function(){
    var list = document.getElementById("list");
    for(var i = 0;i < 100; i++){
        var li = document.createElement("li");
        li.textContent = i;
        list.appendChild(li);
    }
}
```
这段代码将按钮绑定了一个事件，这个事件创建了100个li节点，然后依次将其添加HTML文档中。这样做有一个缺点：每次一创建一个新的元素，然后添加到文档树中，这个过程会造成浏览器的回流。所谓回流简单说就是指元素大小和位置会被重新计算，如果添加的元素太多，会造成性能问题。

这个时候，就是使用createDocumentFragment了DocumentFragment不是文档树的一部分，它是保存在内存中的，所以不会造成回流问题。我们修改上面的代码如下
```javascript
document.getElementById("btnAdd").onclick = function(){
    var list = document.getElementById("list");    
    var fragment = document.createDocumentFragment();

    for(var i = 0;i < 100; i++){
        var li = document.createElement("li");
        li.textContent = i;
        fragment.appendChild(li);
    }
    list.appendChild(fragment);
}
```
优化后的代码主要是创建了一个fragment，每次生成的li节点先添加到fragment，最后一次性添加到list

### 创建型API总结
创建型api主要包括以下四个方法：
- createElement
- createTextNode
- cloneNode
- createDocumentFragment

需要注意下面几点：
- 它们创建的节点只是一个孤立的节点，
- 要通过appendChild添加到文档中
- cloneNode要注意如果被复制的节点是否包含子节点以及事件绑定等问题
- 使用createDocumentFragment来解决添加大量节点时的性能问题

## 页面修改形API（包括删除和添加）（删）（改）
前面我们提到创建型api，它们只是创建节点，并没有真正修改到页面内容，而是要调用appendChild来将其添加到文档树中。

### appendChild
appendChild我们在前面已经用到多次，就是将指定的节点添加到调用该方法的节点的子元素的末尾。调用方法如下：
```javascript
parent.appendChild(child);
```
child节点将会作为parent节点的最后一个子节点

appendChild这个方法很简单，但是还有有一点需要注意：如果被添加的节点是一个页面中存在的节点，则执行后这个节点将会添加到指定位置，其原本所在的位置将移除该节点，也就是说不会同时存在两个该节点在页面上，相当于把这个节点移动到另一个地方
```html
<div id="child">
    要被添加的节点
</div>
<br/>
<br/>
<br/>
<div id="parent">
    要移动的位置
</div>        
<input id="btnMove" type="button" value="移动节点" />
<script>
    document.getElementById("btnMove").onclick = function(){
        var child = document.getElementById("child");
        document.getElementById("parent").appendChild(child);
    }
</script>
```
这段代码主要是获取页面上的child节点，然后添加到指定位置，可以看到原本的child节点被移动到parent中了。
这里还有一个要注意的点：如果child绑定了事件，被移动时，它依然绑定着该事件

### insertBefore
insertBefore用来添加一个节点到一个参照节点之前，用法如下
```javascript
parentNode.insertBefore(newNode,refNode);
```
其中：
- parentNode表示新节点被添加后的父节点
- newNode表示要添加的节点
- refNode表示参照节点，新节点会添加到这个节点之前
```html
<div id="parent">
    父节点
    <div id="child">子元素</div>
</div>
<input type="button" id="insertNode" value="插入节点" />
<script>
    var parent = document.getElementById("parent");
    var child = document.getElementById("child");
    document.getElementById("insertNode").onclick = function(){
        var newNode = document.createElement("div");
        newNode.textContent = "新节点"
        parent.insertBefore(newNode,child);
    }
</script>
```
这段代码创建了一个新节点，然后添加到child节点之前
和appendChild一样，如果插入的节点是页面上的节点，则会移动该节点到指定位置，并且保留其绑定的事件。
关于第二个参数参照节点还有几个注意的地方：
- refNode是必传的，如果不传该参数会报错
- 如果refNode是undefined或null，则insertBefore会将节点添加到子元素的末尾

### removeChild
```javascript
let oldChild = node.removeChild(child);
//OR
element.removeChild(child);
```
其中：
- child 是要移除的那个子节点.
- node 是child的父节点.
- oldChild保存对删除的子节点的引用. oldChild === child.

被移除的这个子节点仍然存在于内存中,只是没有添加到当前文档的DOM树中,因此,你还可以把这个节点重新添加回文档中,当然,实现要用另外一个变量比如上例中的oldChild来保存这个节点的引用. 如果使用上述语法中的第二种方法, 即没有使用 oldChild 来保存对这个节点的引用, 则认为被移除的节点已经是无用的,在短时间内将会被内存管理回收。

如果上例中的child节点不是node节点的子节点,则该方法会抛出异常。
```javascript
// 先定位父节点,然后删除其子节点
var d = document.getElementById("top");
var d_nested = document.getElementById("nested");
var throwawayNode = d.removeChild(d_nested);

// 无须定位父节点,通过parentNode属性直接删除自身
var node = document.getElementById("nested");
if (node.parentNode) {
    node.parentNode.removeChild(node);
}

// 移除一个元素节点的所有子节点
var element = document.getElementById("top");
while (element.firstChild) {
    element.removeChild(element.firstChild);
}
```

### replaceChild
replaceChild用于使用一个节点替换另一个节点，用法如下
```javascript
parent.replaceChild(newChild,oldChild);
```
其中：
- newChild是替换的节点，可以是新的节点，也可以是页面上的节点，如果是页面上的节点，则其将被转移到新的位置
- oldChild是被替换的节点



### 修改页面内容的api总结
修改页面内容的api主要包括：
- appendChild(追加为子元素)
- insertBefore(插入前面)
- removeChild(删除子元素)
- replaceChild(替换子元素)

页面修改型api主要是这四个接口，要注意几个特点：
- 不管是新增还是替换节点，如果新增或替换的节点是原本存在页面上的，则其原来位置的节点将被移除，也就是说同一个节点不能存在于页面的多个位置
- 节点本身绑定的事件会不会消失，会一直保留着

## 节点查询型API(查)
### document.getElementById
这个接口很简单，根据元素id返回元素，返回值是Element类型，如果不存在该元素，则返回null

使用这个接口有几点要注意：
- 元素的Id是大小写敏感的，一定要写对元素的id
- HTML文档中可能存在多个id相同的元素，则返回第一个元素
- 只从文档中进行搜索元素，如果创建了一个元素并指定id，但并没有添加到文档中，则这个元素是不会被查找到的

### document.getElementsByTagName
这个接口根据元素标签名获取元素，返回一个即时的HTMLCollection类型，什么是即时的HTMLCollection类型呢？
```html
<div>div1</div>
<div>div2</div>

<input type="button" value="显示数量" id="btnShowCount"/>
<input type="button" value="新增div" id="btnAddDiv"/>    
<script>   
    var divList = document.getElementsByTagName("div");
    document.getElementById("btnAddDiv").onclick = function(){
        var div = document.createElement("div");
        div.textContent ="div" + (divList.length+1);
        document.body.appendChild(div);
    }

    document.getElementById("btnShowCount").onclick = function(){
        alert(divList.length);
    }
</script>
```
这段代码中有两个按钮，一个按钮是显示HTMLCollection元素的个数，另一个按钮可以新增一个div标签到文档中。前面提到HTMLCollcetion元素是即时的表示该集合是随时变化的，
可以看出其实document.getElementsByTagName返回的是一个数组对象我们可以用数组的方式对其进行操作。

使用document.getElementsByTagName这个方法有几点要注意：
- 如果要对HTMLCollection集合进行循环操作，最好将其长度缓存起来，因为每次循环都会去计算长度，暂时缓存起来可以提高效率
- 如果没有存在指定的标签，该接口返回的不是null，而是一个空的HTMLCollection
- 参数“*”表示所有标签

### document.getElementsByName
getElementsByName主要是通过指定的name属性来获取元素，它返回一个即时的NodeList（节点列表）对象。一般用于获取表单元素的·name·属性

使用这个接口主要要注意几点：
- 返回对象是一个即时的NodeList，它是随时变化的
- 在HTML元素中，并不是所有元素都有name属性，比如div是没有name属性的，但是如果强制设置div的name`属性，它也是可以被查找到的
- 在IE中，如果id设置成某个值，然后传入getElementsByName的参数值和id值一样，则这个元素是会被找到的，所以最好不好设置同样的值给id和name

### document.getElementsByClassName
这个API是根据元素的class返回一个即时的HTMLCollection，用法如下
```javascript
var elements = document.getElementsByClassName(names);
```
这个接口有下面几点要注意：
- 返回结果是一个即时的HTMLCollection，会随时根据文档结构变化
- IE9以下浏览器不支持
- 如果要获取2个以上classname，可传入多个classname，每个用空格相隔，例如var elements = document.getElementsByClassName("test1 test2");

### document.querySelector和document.querySelectorAll
这两个api很相似，通过css选择器来查找元素，注意选择器要符合CSS选择器的规则

**document.querySelector**
querySelector方法返回匹配指定的CSS选择器的元素节点。如果有多个节点满足匹配条件，则返回第一个匹配的节点。如果没有发现匹配的节点，则返回null。
```javascript
var el1 = document.querySelector(".myclass");
var el2 = document.querySelector('#myParent > [ng-click]');
```
querySelector方法无法选中CSS伪元素。

注意，由于返回的是第一个匹配的元素，这个api使用的深度优先搜索来获取元素
```html
<div>
    <div>
        <span class="test">第三级的span</span>    
    </div>
</div>
<div class="test">            
    同级的第二个div
</div>
<input type="button" id="btnGet" value="获取test元素" />
<script>
    document.getElementById("btnGet").addEventListener("click",function(){
        var element = document.querySelector(".test");
        alert(element.textContent);
    })
</script>
```
这个例子很简单，就是两个class都包含“test”的元素，一个在文档树的前面，但是它在第三级，另一个在文档树的后面，但它在第一级，通过querySelector获取元素时，它通过深度优先搜索，拿到文档树前面的第三级的元素(文档树前面优先深度优先)

**document.querySelectorAll**
document.querySelectorAll的不同之处在于它返回的是所有匹配的元素，querySelectorAll方法的参数，可以是逗号分隔的多个CSS选择器，返回所有匹配其中一个选择器的元素。
```javascript
var matches = document.querySelectorAll("div.note, div.alert");
```
eg：
```html
<div class="test">
    class为test
</div>
<div id="test">
    id为test
</div>
<input id="btnShow" type="button" value="显示内容" />

<script>
    document.getElementById("btnShow").addEventListener("click",function(){
        var elements = document.querySelectorAll("#test,.test");    
        for(var i = 0,length = elements.length;i<length;i++){
            alert(elements[i].textContent);
        }    
    })
</script>
```
querySelectorAll也是通过深度优先搜索，搜索的元素顺序和选择器的顺序无关
返回的是一个非即时的NodeList，也就是说结果不会随着文档树的变化而变化

**注：querySelector和querySelectorAll在ie8以下的浏览器不支持**

### elementFromPoint()
elementFromPoint方法返回位于页面指定位置的元素。
```javascript
var element = document.elementFromPoint(x, y);
```
上面代码中，elementFromPoint方法的参数x和y，分别是相对于当前窗口左上角的横坐标和纵坐标，单位是CSS像素。
elementFromPoint方法返回位于这个位置的DOM元素，如果该元素不可返回（比如文本框的滚动条），则返回它的父元素（比如文本框）。如果坐标值无意义（比如负值），则返回null。

### 小结
document.getElementById返回一个对象
document.getElementsByName和document.getElementsByClasName返回一个对象数组

## 元素属性型操作（属性节点的操作）
### getAttribute()(获取属性)
getAttribute()用于获取元素的attribute值
```javascript
node.getAttribute('id');
//表示获取node元素的id属性的  ‘值’
```

### createAttribute()(创建属性)
createAttribute()方法生成一个新的属性对象节点，并返回它。
```javascript
attribute = document.createAttribute(name);
//createAttribute方法的参数name，是属性的名称。
```

### setAttribute()(设置属性)
setAttribute()方法用于设置元素属性
```javascript
var node = document.getElementById("div1");
node.setAttribute(name, value);
//name为属性名称 ；value为属性值

//例如

var node = document.getElementById("div1");
node.setAttribute("id", "ct");

//等同于

var node = document.getElementById("div1");
var a = document.createAttribute("id");
a.value = "ct";
node.setAttributeNode(a);
```

### romoveAttribute()(删除属性)
removeAttribute()用于删除元素属性
```javascript
node.removeAttribute('id');
```

### element.attributes（将属性生成数组对象）
当然上面的方法做的事情也可以通过类似操作数组属性element.attributes来实现

语法
```javascript
var attr = element.attributes;
```
示例
```javascript
// 获取文档的第一个 <p> 元素
var para = document.getElementsByTagName("p")[0];
//获取该元素属性（多个属性会形成一个数组对象）
var atts = para.attributes;
```

**遍历元素的属性**
索引有利于遍历一个元素的所有属性。

在以下例子中会遍历文档中 id 为 "paragraph" 元素的属性节点，并打印出来。
```html
<p id="paragraph" style="color: green;">Sample Paragraph</p>
<form action="">
    <p>
        <input type="button" value="显示第一个属性及其值" onclick="showFirstAttr();">
        <input id="result" type="text" value="">
    </p>
</form>
<script type="text/javascript">
    function showFirstAttr() {
        var paragraph= document.getElementById("paragraph");
        var result= document.getElementById("result");

        // 首先，让我们确认这个段落有一些属性。
        //hasAttributes属性返回一个布尔值true或false,来表明当前元素节点是否有至少一个的属性(attribute).
        if (paragraph.hasAttributes()) {
            var attrs = paragraph.attributes;
            var output= ""; 
            for(var i=attrs.length-1; i>=0; i--) {
                output+= attrs[i].name + "->" + attrs[i].value;
            }
            result.value = output;  //写入 value=""使文本框出现内容
        } else {
            result.value = "没有属性显示了" //写入 value="没有属性显示了"使文本框出现内容
        }
    }
</script>
```

## innerText和innerHTML（outerHTML）有什么区别？
### innerText
innerText是一个可写属性，返回元素内包含的文本内容，在多层次的时候会按照元素由浅到深的顺序拼接其内容
```html
<div id="test">
    <!--注释-->
    <p>我的名字叫<span>哈哈哈</span></p>
</div>
<script>
    var test = document.getElementById('test')
    console.log(test.innerText);
    //返回的结果是：我的名字叫哈哈哈
</script>
```
只包含文本节点

### innerHTML、outerHTML
innerHTML属性作用和innerText类似，但是不是返回元素的文本内容，而是返回元素的HTML结构，在写入的时候也会自动构建DOM
```html
<div id="test">
    <!--注释-->
    <p>我的名字叫<span>哈哈哈</span></p>
</div>
<script>
    var test = document.getElementById('test')
    console.log(test.innerHTML);
    //返回的结果是：
    //<!--注释-->
    //<p>我的名字叫<span>哈哈哈</span></p>
</script>
```
包含整个html结构；（元素节点、注释节点、文本节点）

再如`outerHTML`：
```html
<div id="test">
    <!--注释-->
    <p>我的名字叫<span>哈哈哈</span></p>
</div>
<script>
    var test = document.getElementById('test')
    console.log(test.innerHTML);
    //返回的结果是：
    // <div id="test">
    //     <!--注释-->
    //     <p>我的名字叫<span>哈哈哈</span></p>
    // </div>
</script>
```
包括自身和包含整个html结构；（元素节点、注释节点、文本节点）

区别：
- innerText返回的是元素内包含的文本内容（只返回文本节点类型）；
- innerHTML返会元素内HTML结构，包括元素节点、注释节点、文本节点；
- outerHTML返回包括元素节点自身和里面的所有元素节点、注释节点、文本节点；

## 参考链接
[【DOM】DOM的操作（增删改查）](https://www.jianshu.com/p/b0aa846f4dcc)