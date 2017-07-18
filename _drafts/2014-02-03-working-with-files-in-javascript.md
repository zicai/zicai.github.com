---
layout: post
category : lessons
title: "【译】在 JavaScript 中使用文件"
tagline: "Supporting tagline"
tags : [javascript]
---

## 第一部分：基础知识

### the File type

[File API] 规范定义了 `File` 类型，是一个文件的抽象表示。`File` 类型的每个实例包含以下几个属性：

- `name`：文件名
- `size`：文件大小，单位 byte
- `type`：文件的 MIME 类型

`File` 对象在不直接访问文件内容的前提下，基本上提供了关于文件的必要信息。因为读取文件内容需要 disk access，还要看文件大小（可能会需要很长时间）。一个 `File` 对象仅仅是文件的一个引用，与从文件中读取数据是完全分开的流程。

### 获取文件引用

当然，在网页中访问用户文件是被严格禁止的，因为这是很明显的安全隐患。你不会希望当你浏览网页时，网页可以扫描你的硬盘，查看里面的内容。为了访问计算机上的文件，需要从用户获得授权。然而，没有必要显示麻烦的授权窗口，因为当用户决定上传内容时，他总是给网页授权。

当你使用 `<input type="files">` 控件时，你就是在进行授权。

HTML5 给所有的 `<input type="file">` 控件定义了 `files` 属性。这个集合是一个 `FileList`，它是一个类数组结构，包含着 `File` 对象。你可以使用类似下面的代码来访问用户选择的文件。

```html
<input type="file" id="your-files" multiple>
<script>
var control = document.getElementById("your-files");
control.addEventListener("change", function(event) {

    // When the control has changed, there are new files

    var i = 0,
        files = control.files,
        len = files.length;

    for (; i < len; i++) {
        console.log("Filename: " + files[i].name);
        console.log("Type: " + files[i].type);
        console.log("Size: " + files[i].size + " bytes");
    }

}, false);
</script>

```

监听 file 控件的 change 事件。记住，总是可以通过 JavaScript 访问 files 属性，不一定要等到 change。
### 拖拽文件

HTML5 Drag and Drop 提供了另外一种授权网站访问文件的方式：从计算机中拖动文件到浏览器中。你要做的就是监听两个事件。

为了获取投放到页面某区域的文件，你必须监听 dargover 和 drop 事件，取消它们的默认行为。告诉浏览器由你来直接处理该动作，例如：打开图片文件。

```html
<div id="your-files"></div>
<script>
var target = document.getElementById("your-files");

target.addEventListener("dragover", function(event) {
    event.preventDefault();
}, false);

target.addEventListener("drop", function(event) {

    // cancel default actions
    event.preventDefault();

    var i = 0,
        files = event.dataTransfer.files,
        len = files.length;

    for (; i < len; i++) {
        console.log("Filename: " + files[i].name);
        console.log("Type: " + files[i].type);
        console.log("Size: " + files[i].size + " bytes");
    }

}, false);
</script>
```

`event.dataTransfer.files` 是另外一个 `FileList` 对象，可以用来访问文件信息。

### Ajax 文件上传

一旦你获得了文件引用，就可以利用它来做一些很酷的事儿了：通过 Ajax 上传。FormData 对象让着一切变为可能。FormData 定义在 [XMLHttpRequest Level 2]。它代表一个 HTML 表单，允许你通过 `append()` 方法添加要提交到服务器的键值对。

```javascript
var form = new FormData();
form.append("name","Nicholas");
```

FormData 对象厉害之处在于，你可以直接添加一个文件进去，从而模拟文件上传表单。你只需将文件引用和一个名字添加进去，剩下的就交给浏览器。如下：

```javascript
// create a form with a couple of values
var form = new FormData();
form.append("name", "Nicholas");
form.append("photo", control.files[0]);

// send via XHR - look ma, no headers being set!
var xhr = new XMLHttpRequest();
xhr.onload = function() {
    console.log("Upload complete.");
};
xhr.open("post", "/entrypoint", true);
xhr.send(form);
```

只要 FormData 传递到 `send()`，就会自动设置合适的 HTTP headers，使用文件时，你不用担心设置正确的 form encoding。服务端就像接收到常规表单提交一样处理即可。

上面提到了在浏览器中获取 File metadata 的两种方式：

- 通过表单上传控件
- 通过文件拖拽

以后可能还会出现新的方式。接下来，要讲到的是从文件中读取内容。


## 第二部分：FileReader

### FileReader 类型

`FileReader` 类型只有一个任务：从一个文件中读取数据，保存到一个 JavaScript 变量中。API 有意设计成与 `XMLHttpRequest` 类似，因为他们都是从外部资源（浏览器之外）加载数据。读取过程是异步的，不会阻塞浏览器。

`FileReader` 可以创建多种格式来表示文件数据，and the format must be requested when asking the file to be read.通过调用下面的方法来完成读取：

- `readAsText()` -- 以文本形式返回文件内容
- `readAsBinaryString()` -- 废弃，用 `readAsArrayBuffer()` 代替
- `readAsArrayBuffer()` -- 以 `ArrayBuffer` 格式返回文件内容（适合二进制数据，例如图片）
- `readAsDataURL()` -- 以 data URL 格式返回文件内容

上面这些方法初始化一个文件读取，类似于 XHR() 对象的 `send()` 方法初始化 HTTP 请求。同样的，你必须在开始读取之前监听 load 事件。读取的结果总是由 `event.target.result` 表示。如下：

```javascript
var reader = new FileReader();
reader.onload = function(event) {
    var contents = event.target.result;
    console.log("File contents: " + contents);
};

reader.onerror = function(event) {
    console.error("File could not be read! Code " + event.target.error.code);
};

reader.readAsText(file);
```

当读取成功时调用 onload handler，当读取失败时，调用 onerror handler。在 event handler 中通过 event.target 可以获取 FileReader 实例，建议使用 event.target 而不是直接引用 reader 变量。result 属性在读取成功时包含着文件内容，读取失败时包含着错误信息。

### reading data URIs

Data URIs(有时候叫做 data URLs) 很有用，例如：读取计算机上的图片并显示出来，你可以使用下面的代码：

```javascript
var reader = new FileReader();
reader.onload = function(event) {
    var dataUri = event.target.result,
        img     = document.createElement("img");

    img.src = dataUri;
    document.body.appendChild(img);
};

reader.onerror = function(event) {
    console.error("File could not be read! Code " + event.target.error.code);
};

reader.readAsDataURL(file);
```

由于 data URI 包含着所有的图片数据，可以把它赋值给图片的 src 属性。你还可以通过 `<canvas>` 显示图片：

```javascript
var reader = new FileReader();
reader.onload = function(event) {
    var dataUri = event.target.result,
        context = document.getElementById("mycanvas").getContext("2d"),
        img     = new Image();

    // wait until the image has been fully processed
    img.onload = function() {
        context.drawImage(img, 100, 100);
    };
    img.src = dataUri;
};

reader.onerror = function(event) {
    console.error("File could not be read! Code " + event.target.error.code);
};

reader.readAsDataURL(file);
```

Data URI 通常都是用来显示图片，但是它也可以用于其它类型的文件。读取文件内容到 data URI 最常见的用法就是直接在网页中显示文件内容。

### reading ArrayBuffer

ArrayBuffer 最开始是作为 WebGL 的一部分被引入的。

处理二进制文件时，为了获得对数据细粒度的控制，优先使用 ArrayBuffer。

可以将 ArrayBuffer 直接传递给 XHR 对象的 `send()` 方法，来发送 raw data 到服务器（服务器从请求中读取这些数据并重建文件）。

接下来，要讲 FileReader 事件和错误处理。

## 第三部分：Progress 事件和错误

### Progress 事件

progress 事件越来越常见，所以变成了一个单独的规范。这些事件大体上设计为表示数据转移的进度。例如：从服务端读取数据，从硬盘读取数据，也就是 FileReader 所做的。

有六个 progress 事件：

- loadstart：指示加载数据的流程开始，总是第一个被触发
- progress：在数据加载过程中多次被触发，可以访问中间数据
- error：当加载失败时触发
- abort：通过调用 `abort()`（XMLHttpRequest 和 FileReader 都支持） 来取消数据加载时触发
- load：当所有数据都成功读取时触发
- loadend：当对象结束数据转移时触发。在 error,abort,load 之后都会触发

前面已经提到了 error 和 load 事件，其它的事件用来细粒度的控制数据转移。

#### 跟进 progress

当你想要跟进 file reader 的进度时，使用 progress 事件。该事件对象包含三个属性用于监控数据转移情况：

- lengthComputable：布尔值，指示浏览器是否可以确定数据的完整大小。
- loaded：已经读取的字节总数
- total：需要读取的字节总数

可以使用 HTML5 的 `<progress>` 标签来展示这些数据

```javascript
var reader = new FileReader(),
     progressNode = document.getElementById("my-progress");

reader.onprogress = function(event) {
    if (event.lengthComputable) {
        progressNode.max = event.total;
        progressNode.value = event.loaded;
    }
};

reader.onloadend = function(event) {
    var contents = event.target.result,
        error    = event.target.error;

    if (error != null) {
        console.error("File could not be read! Code " + error.code);
    } else {
        progressNode.max = 1;
        progressNode.value = 1;
        console.log("Contents: " + contents);
    }
};

reader.readAsText(file);
```

### 错误处理

即使读取的是本地文件，也是有可能读取失败的。 File API 规范定义了四种错误：

- NotFoundError
- SecurityError：通常是浏览器加载该文件会有危险或是浏览器当前正在执行多个读取。允许一定误差。
- NotReadableError：通常是由于权限问题
- EncodingError：通常是因为以 data URI 形式读取，结果长度超出了浏览器支持的最大长度

当读取文件时出现错误，FileReader 对象的 error 属性会被赋值为上述某个错误类型的实例，至少规范上是这样写的。事实上，浏览器通过一个 FileError 对象来实现，它有个 code 属性来指示发生的错误类型。每一种错误类型有一个数字常量表示：

- FileError.NOT_FOUND_ERR for file not found errors.
- FileError.SECURITY_ERR for security errors.
- FileError.NOT_READABLE_ERR for not readable errors.
- FileError.ENCODING_ERR for encoding errors.
- FileError.ABORT_ERR when abort() is called while there is no read in progress.

你可以在 error 事件或是 loadend 事件中确定错误类型。

```javascript
var reader = new FileReader();

reader.onloadend = function(event) {
    var contents = event.target.result,
        error    = event.target.error;

    if (error != null) {
        switch (error.code) {
            case error.ENCODING_ERR:
                console.error("Encoding error!");
                break;

            case error.NOT_FOUND_ERR:
                console.error("File not found!");
                break;

            case error.NOT_READABLE_ERR:
                console.error("File could not be read!");
                break;

            case error.SECURITY_ERR:
                console.error("Security issue with file!");
                break;

            default:
                console.error("I have no idea what's wrong!");
        }
    } else {
        progressNode.max = 1;
        progressNode.value = 1;
        console.log("Contents: " + contents);
    }
};

reader.readAsText(file);
```


FileReader 对象是一个功能完备的对象，和 XMLHttpRequest 有很多相似之处。接下来要提到的是另一个强大的新特性。
## 第四部分：Object URL

前面提到的都是传统的处理文件的方式，接下来要讲的是一种全新的方式，可以简化一些常见任务，那就是使用 object URLs。

### Object URL 是什么？

Object URLs 是指向硬盘上文件的URL。假设你想把用户计算机上一张图片显示到网页中，服务器无需知道图片内容，所以没有必要上传。你只是想把图片加载到网页中，你可以用前面提到的方法：用 File 对象获取一个文件引用，读取数据成 data URI，然后将 data URI 赋值给 `<img>` 元素。但是仔细想一想：图片已经存在于硬盘上了，为了使用它为什么要读取成另外一种格式呢？如果你创建一个 object URL，就可以直接将其赋值给 `<img>`，直接访问本地文件。

### Object URL 如何工作？

File API 定义了一个全局变量，称为 URL，它有两个方法。第一个是 `createObjectURL()`，接受一个 File 对象（文件引用），返回一个 object URL，这会指示浏览器创建和管理对应文件的 URL。第二个方法是 `revokeObjectURL()`，指示浏览器销毁传递进来的 URL，有效的释放内存。当然，当页面关闭时，所有 object URL 都会被 revoke，不过还是建议不需要时就释放。

浏览器对 URL 对象的支持，不像对 File API 其它部分那样好。

### 实例

那么如何显示硬盘上的图片到网页中呢？假设你已经获取到了文件引用，保存到变量 file 中。

```javascript
var URL = window.URL || window.webkitURL,
    imageUrl,
    image;

if (URL) {
    imageUrl = URL.createObjectURL(file);
    image = document.createElement("img");

    image.onload = function() {
        URL.revokeObjectURL(imageUrl);
    };

    image.src = imageUrl;
    document.body.appendChild(image);
}
```

为什么要在图片加载完成后就 revoke object URL 呢？因为图片加载完后，URL 就没用了，除非你还想在其它元素上使用。

### 安全和其它方面的考虑

File API 不允许在不同域上使用 object URL。当 object URL 创建时，就被绑定在 JavaScript 执行的当前域中。

Object URL 只存在于创建它的文档中。当文档被卸载时，所有 object URL 都被 revoke。所以将 object URL 保存到本地存储是没有道理的。

You can use object URLs anywhere the browser would make a GET request, which includes images, scripts, web workers, style sheets, audio, and video. You can never use an object URL when the browser would perform a POST, such as within a <form> whose method is set to “post”.


为了在网页中显示，无需再将本地文件读取到 JavaScript，你只需创建一个 URL。这个极大的简化了在网页中使用本地文件的流程。


## 第五部分：Blobs

实际上， File 对象是 Blob 的特定版本，Blob 表示一段二进制数据。size 和 type 属性存在于 Blob 对象，被 File 继承。

在大多数情况，Blob 和 File 可以用在同样的地方。例如，你可以使用 FileReader 从 Blob 中读取数据，还可以使用 `URL.createObjectURL()` 从 Blob 创建 object URL。

### 切片
可以用这种方法，将大文件切片上传。

### 创建 Blob 的旧方法

```javascript
var builder = new BlobBuilder();
builder.append("Hello world!");
var blob = builder.getBlob("text/plain");
```

你可以多次调用 `append()`，来构建 Blob 内容。
### 创建 Blob 的新方法

构造函数接受两个参数，第一个是数组，任意数量的 字符串，blobs 或 ArrayBuffer。第二个参数是对象，包含新创建 Blob 的属性，当前可以定义两个属性：type 和 endings

```javascript
var blob = new Blob(["Hello world!"], { type: "text/plain" });
```



原文地址：

- [https://www.nczonline.net/blog/tag/file-api/](https://www.nczonline.net/blog/tag/file-api/)
- [File API]()
- [XMLHttpRequest Level 2](http://www.w3.org/TR/XMLHttpRequest/)