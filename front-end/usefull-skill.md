
如果对Blob不了解，可以先看看我好些年之前写的“理解DOMString、Document、FormData、Blob、File、ArrayBuffer数据类型”一文。

原理其实很简单，我们可以将文本或者JS字符串信息借助Blob转换成二进制，然后，作为<a>元素的href属性，配合download属性，实现下载。

代码也比较简单，如下示意（兼容Chrome和Firefox）：
```js
var funDownload = function (content, filename) {
    // 创建隐藏的可下载链接
    var eleLink = document.createElement('a');
    eleLink.download = filename;
    eleLink.style.display = 'none';
    // 字符内容转变成blob地址
    var blob = new Blob([content]);
    eleLink.href = URL.createObjectURL(blob);
    // 触发点击
    document.body.appendChild(eleLink);
    eleLink.click();
    // 然后移除
    document.body.removeChild(eleLink);
};
```
