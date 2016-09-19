---
layout: post
category : lessons
title: "HTML 文档字符集"
tagline: "Supporting tagline"
tags : [html]
---

## The Document Character Set

文档字符集(Document Character Set)规定了文档中可以包含哪些字符

为了提升互操作性，SGML 要求它的每一个应用（包括 HTML）声明文档字符集。文档字符集包含：

- A Repertoire（指令表）：A set of abstract characters
- Code positions: A set of integer references to characters in the repertoire.

每个 SGML 文档（包括 HTML 文档）都是指令表中字符集的序列。计算机系统通过它的 code position 来识别每个字符。例如，在 ASCII 字符集，code position 65,66 和 67 分别引用 A，B 和 C

ASCII 字符集对于 Web 这样一个全球信息系统而言是不够用的，所以 HTML 使用了更完整的字符集，称为 Universal Character Set (UCS)，定义在 [ISO10646]。该标准定义了由全世界使用的成千上万的字符组成的指令表。

[ISO10646] 中定义的字符集与 [UNICODE] 字符集一一对应。如果有新字符加入，两者是同时更新的。

在当前的规范中，[ISO10646] 用来指代文档字符集，而 [UNICODE] 用来指代 Unicode bidirectional text algorithm.

浏览器只凭文档字符集并不足以正确的解析 HTML 文档，因为在网络传输中通常会进行编码。浏览器必须知道用来将文档字符流转换为字节流的特定字符编码（Character encodings）。

## Character encodings
What this specification calls a character encoding is known by different names in other specifications (which may cause some confusion). However, the concept is largely the same across the Internet. Also, protocol headers, attributes, and parameters referring to character encodings share the same name -- "charset" -- and use the same values from the [IANA] registry (see [CHARSETS] for a complete list).

charset 指定了将字节流转换为字符序列的字符编码。

服务器以字节流的形式将 HTML 文档发送到浏览器，浏览器根据 charset 将其解析为字符序列。

字符编码有很多方法。

### 选择一种编码
### 指定字符编码
浏览器和服务器如何确定编码？？？？？？？？？？？

## Character references
一个给定的字符编码可能不足以描述文档字符集中所有字符。对于这些编码，以及硬件或软件配置不允许用户直接输入某些字符时，可以使用 SGML 字符引用。Character references are a character encoding-independent mechanism for entering any character from the document character set.

HTML 中的字符引用有两种形式：

- Numeric character references (either decimal or hexadecimal).
- Character entity references (字符实体引用)

### Numeric character references
Numeric character references 指明了字符在文档字符集中的 code position。Numeric character references 有两种形式：

- `&#D`;   其中 `D` 是十进制数。参见 [ISO10646]
- `&#xH`;  或  `&#XH`;   其中 `H` 是十六进制数，大小写无关。参见 [ISO10646]

### 字符实体引用
为了让作者更轻松地引用文档字符集中的字符，HTML 提供了一个字符实体引用集合。字符实体引用使用象征性的名字，这样作者就不用去记忆 code postion。

HTML 4 并没有给文档字符集中的所有字符都定义实体引用。HTML 4 中实体引用的完整列表 [https://www.w3.org/TR/html4/sgml/entities.html](https://www.w3.org/TR/html4/sgml/entities.html)

字符实体引用是大小写敏感的。

常见的字符实体引用：

- `&lt` 表示  `<`
- `&gt`  表示  `>`
- `&amp`  表示  `&`
- `&quot`  表示  `"`

## 不可见字符


参考资料：

- [https://www.w3.org/TR/html4/charset.html](https://www.w3.org/TR/html4/charset.html)
- [https://mathiasbynens.be/notes/ambiguous-ampersands](https://mathiasbynens.be/notes/ambiguous-ampersands)
- https://developers.whatwg.org/named-character-references.html#named-character-references

[ISO10646]:(https://www.w3.org/TR/html4/references.html#ref-ISO10646)
[UNICODE]:(https://www.w3.org/TR/html4/references.html#ref-UNICODE)












