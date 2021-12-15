### `window.open()` 跳转新标签页被浏览器拦截问题

在浏览器的安全机制里，非用户触发的 `window.open` 方法是会被拦截的。

例如，
我们将 `window.open` 放到了 AJAX 的回调函数里面的话，就会被浏览器拦截，因为浏览器认定这不是用户所触发的。
但如果我们在一个 button 的 click 事件上加上 `window.open` 的话，浏览器就会认为是用户点击触发的。

而我这里遇到的问题就是，将 `window.open` 写在了 AJAX 的回调函数里面，所以导致了浏览器拦截。于是我将 `window.open` 的相关代码，都封装到了一个方法里面，然后在 AJAX 的回调函数中调用这个方法。这样就 ok 了，浏览器就不会拦截了。

### 打开一个空白新标签页，新标签页里的所有内容都从接口获取

接口返给我的内容为：

```html
<html>
     <head>
        <meta http-equiv="Content-Type" content="text/html; charset=Windows-31J">
     </head>
     <body>
        <form name="UnionpayStartCall" action="https://pt01.mul-pay.jp/payment/UnionpayStart.idPass" method="POST">
        <noscript>
        <center>
            <h2>ネット銀聯の決済画面へ遷移します。</h2>
            <input type="submit" value="続行">
        </center>
        </noscript>    <input type="hidden" name="AccessID" id="AccessID" value="d152e8e1a8fb8083fee9df833552bf23" />
    <input type="hidden" name="Token" id="Token" value="zbL634SU13iCi5b71U5jFITofi6suAQBcUc6xFa1+cmwBS7yYvxSiC0zeMVH+O4F" />
     </form>
     <script >

     function OnLoadEvent() {
         console.log("submit");
        document.UnionpayStartCall.submit();
     }
        OnLoadEvent();

     </script>
     </body>
</html>
```

注意，里面有 JavaScript 代码。

然后，现在来看一下整体的实现思路：

```javascript
let newWin = window.open('_blank');  // 先打开一个新标签
newWin.document.write(response); // 然后把接口返的东西放到新标签里
```

由于接口返给我们的页面内容里有 JavaScript 代码，如果直接把接口返的页面内容放到新标签里，JavaScript 是不会执行的。

解决方法是，将 response 存到 localStorage 里，再拿出来用。（如果没有 JavaScript 代码也就不需要这一步了）

```javascript
window.localStorage.setItem('responseHTML', response);
let newWin = window.open('_blank');
newWin.document.write(localStorage.getItem('responseHTML'));
newWin.document.close(); // 关闭输出流
```

到此为止，该问题就解决了。

但要注意，如果在 AJAX 的回调函数中直接使用 `window.open` ，现在的浏览器一般会拦截新标签页的打开，详细可参考 [这里](#`window.open()` 跳转新标签页被浏览器拦截问题) 。

### 点击某按钮后，为整个页面添加遮罩

想实现，用户点击付款，然后用户跳转到新页面去付款。于是当前页面跟着盖上遮罩，让用户无法操作该页面，并在这遮罩上添加一个“支付完成”按钮，用户点击该按钮，遮罩和该按钮都会去掉。

```javascript
    createMask() {
      if (document.querySelector('#mask')) {
        return true;
      }
      let mask = document.createElement('div');
      let finish = document.createElement('button');
      mask.id = 'mask';
      mask.className = 'mask';
      mask.style = 'left: 0; top: 0; bottom:0; right:0; position: fixed; z-index: 				200; background: rgba(0,0,0,0.6);';
      finish.id = 'finish';
      finish.className = 'finish';
      finish.style = 'position: absolute; z-index: 300; top: 50%; right: 50%; 						margin-top: -25px; margin-right: -91px; padding: 1em 2em; font-size: 14px; 				 border: none; border-radius: 4px; color: #fff; background-color: #409eff';
      finish.innerHTML = '支払い終わりました';
      document.body.appendChild(mask);
      document.body.appendChild(finish);
      document.documentElement.classList.add('htmlMask');
      finish.addEventListener('click', this.deleteMask);
    },

    deleteMask() {
      let mask;
      let finish;
      if (mask = document.querySelector('#mask')) {
        mask.parentNode.removeChild(mask);
        document.documentElement.classList.remove('htmlMask');
      }
      if (finish = document.querySelector('#finish')) {
        finish.parentNode.removeChild(finish);
      }
    },
```

以上代码是写在 Vue methods 里面的，createMask 函数绑定到增加遮罩的点击事件上。

以上代码有几个知识点需要注意：

1. mask 的 style 写法

2. finish 按钮的 style 写法
   `top: 50%; right: 50%; margin-top: -25px; margin-right: -91px;` 这里使用了负 margin 以让按钮垂直水平居中。

3. 文档对象模型DOM

   + `let mask = document.createElement('tagName')` 创建一个新的元素；[Document.createElement]([Document.createElement() - Web API 接口参考 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/createElement))

   + `document.body.appendChild(mask)` 将一个节点附加到指定父节点的子节点列表的末尾处。
     如果将被插入的节点已经存在于当前文档的文档树中，那么 `appendChild()` 只会将它从原先的位置移动到新的位置（不需要事先移除要移动的节点）；

     [Node.appendChild]([Node.appendChild - Web API 接口参考 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/API/Node/appendChild))

   + 尽量使用 `document.querySelector()`  和 `document.querySeletorAll()` 来获取元素；
     不推荐用 `document.getElementById()` 和 `document.getElementByTagName()` 和 `document.getElementByClassName()` 来获取元素。

   + `mask.parentNode` 返回指定的节点，即 mask 节点在DOM树中的父节点。[Node.parentNode]([Node.parentNode - Web API 接口参考 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/API/Node/parentNode))

   + `mask.parentNode.removeChild(mask)` 删除 mask 节点自己。`Node.removeChild()` 方法从DOM中删除一个子节点。返回删除的节点。

   + `document.documentElement.classList` 返回一个元素的类属性的实时 [`DOMTokenList`](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMTokenList) 集合。我们可以对这个集合做 `.add('className')` 和 `.remove('className')` 等操作。[Element.classList]([Element.classList - Web API 接口参考 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/classList))
   



### 关于 `process.env.NODE_ENV`

在 node 中，有全局变量 process 表示的是当前的 node 进程。

我们可以在项目中把这个 process 打出来看看。

`process.env` 包含着关于系统环境的信息，但是 `process.env` 中并不存在 `NODE_ENV` 这个东西。

`NODE_ENV` 是一个用户自定义的变量，在 webpack 中它的用途是**判断是生产环境还是开发环境**。

```javascript
// 开发环境
process.env.NODE_ENV === "development"

// 生产环境
process.env.NODE_ENV === "production"
```

