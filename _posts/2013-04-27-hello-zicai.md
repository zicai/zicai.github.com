---
layout: post
title: "hello zicai"
description: ""
category: 
tags: []
---
{% include JB/setup %}

111基本CSS样式
123
通过添加 .lead 让段落突出显示<p class="lead">...</p>
对于不需要强调的inline或block类型的文本，使用small标签。
<p>
  <small>This line of text is meant to be treated as fine print.</small>
</p>
加粗<strong>rendered as bold text</strong>
斜体<em>rendered as italicized text</em>
文本对齐
<p class="text-left">Left aligned text.</p>
<p class="text-center">Center aligned text.</p>
<p class="text-right">Right aligned text.</p>
文本颜色
<p class="muted">Fusce dapibus, tellus ac cursus commodo, tortor mauris nibh.</p>
<p class="text-warning">Etiam porta sem malesuada magna mollis euismod.</p>
<p class="text-error">Donec ullamcorper nulla non metus auctor fringilla.</p>
<p class="text-info">Aenean eu leo quam. Pellentesque ornare sem lacinia quam venenatis.</p>
<p class="text-success">Duis mollis, est non commodo luctus, nisi erat porttitor ligula.</p>
缩略语，为 <abbr> 标签添加 .initialism 类使其使用更小一些的字号。
<abbr title="attribute" class="initialism">attr</abbr>
地址
<address>
  <strong>Twitter, Inc.</strong><br>
  795 Folsom Ave, Suite 600<br>
  San Francisco, CA 94107<br>
  <abbr title="Phone">P:</abbr> (123) 456-7890
</address>
引用，将任何HTML包裹在<blockquote>之中即可表现为引用。对于直接引用，我们建议用<p>标签。添加<small>标签来注明引用来源。来源名称可以放在<cite> 标签里面。使用.pull-right类，可以让引用展现出向右侧移动、对齐的效果。
<blockquote class="pull-right">
  <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Integer posuere erat a ante.</p>
  <small>Someone famous <cite title="Source Title">Source Title</cite></small>
</blockquote>
描述。.dl-horizontal类使<dl>中的每个条目和其描述一对一水平显示。通过加入text-overflow，水平描述列表将会对过长而无法在左栏中完全显示的列名截断掉一部分。而在较窄的视口（宽度）中，会改变成垂直(形式)表述，来适应当前屏幕。
<dl class="dl-horizontal">
  <dt>...</dt>
  <dd>...</dd>
</dl>
代码
通过<code>标签内嵌一小段代码。
使用<pre>展示多行代码。为了能够正确展示，务必将代码中的任何尖括号做转义。因为在<pre>标签里, tab会被算进去, 所以要保持代码尽可能的靠左侧。你也可以添加.pre-scrollable类，把该区域设置成最大高度为350px并带有一个Y轴滚动条。
表格
为 <table> 标签增加基本样式--很少的内补(padding)并只增加水平分隔线--只要为其增加 .table 类即可。.table-striped在<tbody>内，通过:nth-child CSS选择器 (IE7-8不支持)为表格中的行添加斑马纹样式。.table-bordered为表格增加边框(border)和圆角(rounded corner)。.table-hover为 <tbody> 中的每一行赋予鼠标悬停样式。.table-condensed每个单元格的内补(padding)减半可使表格更紧凑。
选择情景(contextual)类为表格添加颜色。给<tr>添加类.success,.error,.warning,.info

表单
为表单增加.form-search类，并为<input>增加.search-query类，可将输入框变成圆角状。
为表单增加.form-inline类， 结果是一个压缩型排列的表单，其中label左侧对齐、控件为inline-block类型。
右侧对齐并且左侧浮动的label和控件排列在同一行。这需要对默认的表单格式做修改：
为表单添加.form-horizontal类,
将label和控件包裹在.control-group中,
为label添加.control-label类,
将任何相关的控件包裹在.controls中，以确保最佳的对齐

输入框的type可以包括所有HTML5所支持的：text、password、datetime、 datetime-local、date、 month、time、week、number、email、url、search、tel 和 color
<input type="text" placeholder="Text input">
文本域
<textarea rows="3"></textarea>

单选框和复选框。通过添加.inline类，使他们排列在同一行
<label class="checkbox inline">
  <input type="checkbox" value="">
  Option one is this and that—be sure to include why it's great
</label>
 
<label class="radio inline">
  <input type="radio" name="optionsRadios" id="optionsRadios1" value="option1" checked>
  Option one is this and that—be sure to include why it's great
</label>
<label class="radio inline">
  <input type="radio" name="optionsRadios" id="optionsRadios2" value="option2">
  Option two can be something else and selecting it will deselect option one
</label>

下拉框，可使用默认的选项或者指定multiple="multiple"属性一次显示多个选项。
<select multiple="multiple">
  <option>1</option>
  <option>2</option>
  <option>3</option>
  <option>4</option>
  <option>5</option>
</select>

扩展表单控件
前缀、后缀文本或按钮。注意，select控件不支持。.input-prepend,.input-append,.add-on
前后缀文字
<div class="input-prepend input-append">
  <span class="add-on">$</span>
  <input class="span2" id="appendedPrependedInput" type="text">
  <span class="add-on">.00</span>
</div>
前后缀按钮，不需要使用.add-on类
<div class="input-append">
  <input class="span2" id="appendedInputButtons" type="text">
  <button class="btn" type="button">Search</button>
  <button class="btn" type="button">Options</button>
</div>
前后缀下拉菜单
<div class="input-append">
  <input class="span2" id="appendedDropdownButton" type="text">
  <div class="btn-group">
    <button class="btn dropdown-toggle" data-toggle="dropdown">
      Action
      <span class="caret"></span>
    </button>
    <ul class="dropdown-menu">
      ...
    </ul>
  </div>
</div>

控件大小
.input-mini,.input-small,.input-medium,.input-large,.input-xlarge,.input-xxlarge
对输入框使用.span1 到 .span12 可以将输入框的大小对齐到网格大小。
.input-block-level让任何<input>或<textarea>元素表现为一个块级元素。
对于每行有多个输入框的情况,使用 .controls-row 类为输入框增加合适的间距。

给输入框添加disabled属性可阻止用户输入，并且输入框会呈现稍微不同的外观。
<input class="input-xlarge" id="disabledInput" type="text" placeholder="Disabled input here..." disabled>
给<button>添加disabled属性。颜色淡出50%，让按钮看起来无法点击。
<button type="button" class="btn btn-large btn-primary disabled" disabled="disabled">Primary button</button>
<button type="button" class="btn btn-large" disabled>Button</button>
给<a>元素添加.disabled类。颜色淡出50%，让按钮看起来无法点击。
<a href="#" class="btn btn-large disabled">Link</a>
对于在表单中呈现不可编辑的数据，无需使用实际的表单控件。
<span class="input-xlarge uneditable-input">Some value here</span>

将一组行为(按钮)放在表单尾部。当他们放置在.form-actions中时，这些按钮将会自动缩进，和其它表单控件对齐。
<div class="form-actions">
  <button type="submit" class="btn btn-primary">Save changes</button>
  <button type="button" class="btn">Cancel</button>
</div>

表单控件周围可以放置行内或块级元素展示帮助文本。
<input type="text"><span class="help-inline">Inline help text</span>
<input type="text"><span class="help-block">A longer block of help text that breaks onto a new line and may extend beyond one line.</span>

Bootstrap包含了（错误）error、（警告）warning、（通知）info和（成功）success信息的样式。为.control-group添加适当的属性即可使用这些样式。

任何赋予.btn类的页面元素都会显示按钮样式。不过，通常是用于更好的表现<a> 和 <button> 页面元素。
按钮颜色：.btn-primary,.btn-info,.btn-success,btn-warning,btn-danger,.btn-inverse
按钮大小：.btn-large,.btn-small,.btn-mini
通过添加.btn-block类，可使按钮变为块级元素，同时会填充整个父级元素。
.btn-link简化一个按钮, 使它看起来像一个链接，同时保持按钮的行为

为<img>元素添加相应的类.img-rounded,.img-circle,img-polaroid就可以很容易的给图片设置样式。
<img src="..." class="img-rounded">
<img src="..." class="img-circle">
<img src="..." class="img-polaroid">
