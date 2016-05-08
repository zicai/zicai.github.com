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

当你使用`<input type="files">`控件时，你就是在进行授权。

HTML5 给所有的 `<input type="file">` 控件定义了 `files` 属性。这个集合是一个 `FileList`，它是一个类数组结构，包含着 `File` 对象。你可以使用类似下面的代码来访问用户选择的文件。

```
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

监听 file 控件的 change 事件。记住，总是可以通过 JavaScript 访问 files 属性，不一定要等到 change 。
### 拖拽文件

HTML5 Drag and Drop 提供了另外一种授权网站访问文件的方式：从计算机中拖动文件到浏览器中。你要做的就是监听两个事件。

为了获取投放到页面某区域的文件，你必须监听 dargover 和 drop 事件，取消它们的默认行为。告诉浏览器由你来直接处理该动作，例如：打开图片文件。

```
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

一旦你获得了文件引用，就可以利用它来做一些很酷的事儿了：通过 Ajax 上传。FormData 对象让着一切变为可能。FormData 定义在 [XMLHttpRequest Level 2]。它代表一个 HTML 表单，允许你通过 append() 方法添加要提交到服务器的键值对。

```
var form = new FormData();
form.append("name","Nicholas");
```

FormData 对象厉害之处在于，你可以直接添加一个文件进去，从而模拟文件上传表单。你只需将文件引用和一个名字添加进去，剩下的就交给浏览器。如下：

```
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

只要 FormData 传递到 send()，就会自动设置合适的 HTTP headers，使用文件时，你不用担心设置正确的 form encoding。服务端就像接收到常规表单提交一样处理即可。

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

上面这些方法初始化一个文件读取，类似于 XHR() 对象的 send() 方法初始化 HTTP 请求。同样的，你必须在开始读取之前监听 load 事件。读取的结果总是由 event.target.result 表示。如下：

```
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

```
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

```
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

可以将 ArrayBuffer 直接传递给 XHR 对象的 send() 方法，来发送 raw data 到服务器（服务器从请求中读取这些数据并重建文件）。

接下来，要讲 FileReader 事件和错误处理。

## 第三部分：Progress 事件和错误

### Progress 事件

progress 事件越来越常见，所以变成了一个单独的规范。这些事件大体上设计为表示数据转移的进度。例如：从服务端读取数据，从硬盘读取数据，也就是 FileReader 所做的。

有六个 progress 事件：

- loadstart：指示加载数据的流程开始，总是第一个被触发
- progress：在数据加载过程中多次被触发，可以访问中间数据
- error：当加载失败时触发
- abort：通过调用 abort()（XMLHttpRequest 和 FileReader 都支持） 来取消数据加载时触发
- load：当所有数据都成功读取时触发
- loadend：当对象结束数据转移时触发。在 error,abort,load 之后都会触发

前面已经提到了 error 和 load 事件，其它的事件用来细粒度的控制数据转移。

#### 跟进 progress

当你想要跟进 file reader 的进度时，使用 progress 事件。该事件对象包含三个属性用于监控数据转移情况：

- lengthComputable：布尔值，指示浏览器是否可以确定数据的完整大小。
- loaded：已经读取的字节总数
- total：需要读取的字节总数

可以使用 HTML5 的 <progress> 标签来展示这些数据

```
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

```
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

## 第四部分：Object URL

## 第五部分：Blobs

原文地址：

- [https://www.nczonline.net/blog/tag/file-api/](https://www.nczonline.net/blog/tag/file-api/)

[File API]:()
[XMLHttpRequest Level 2]:(http://www.w3.org/TR/XMLHttpRequest/)