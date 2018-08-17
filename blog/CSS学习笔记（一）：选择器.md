### 选择器
    用与定位到具体的html元素上，并将样式应用于元素上面
###### 1.元素选择器 
***
* 元素选择器：
文档(HTML、XML)的元素就是最基本的选择器,html所有元素如html 、h1、h2、p都能作为选择器

```css
代码示例：html {color: red}; h1 {color: black}; h2 {color: green}
```
* 选择器分组：
用于将一组html元素应用同一个样式
```css
代码示例：html,h1,h2{color: red}; 
```
* 类选择器：.+class，可以指定给多个元素
```css
代码示例：
p.success {font-size: 10px};   <!--含有类名success的p元素-->
.success {font-size: 10px};
.success.fail {font-size: 10px};  <!--同时含有类名success、fail的元素-->
```
* id选择器：#+id，只能够指定给一个元素
```css
代码示例：
#success {font-size: 10px};
```
###### 2.属性选择器

1.简单的属性选择器：当某个元素具有某个属性时选择
```css
代码示例： 
h1[class] {color: black};     <!--当h1元素具有class属性时应用此样式-->
img[alt] {border: 1px solid #999}；
a[href][title] {font-size: 12px}; 
p[class="success"] {font-size: 12px}; <!--当p的class属性为"success"时应用此样式-->
p[class~="success"] {font-size: 12px}; <!--当p的class属性包含"success"类名时应用此样式-->
p[class*="success"] {font-size: 12px}; <!--当p的class属性包含"success"时应用此样式-->
p[class^="success"] {font-size: 12px}; <!--当p的class属性以"success"开头时应用此样式-->
p[class$="success"] {font-size: 12px}; <!--当p的class属性以"success"结尾时应用此样式-->
```
2.特殊属性选择器
```css
代码示例：*[lang|="en"] {color: white} <!--匹配lang属性等于en或以en-开头的所有元素-->
```
###### 3.基于文档树结构的选择器
1.  后代选择器：从祖先元素找到后代元素，其中从父元素找到子元素是一个特例情况,元素之间用空格隔开
```css
代码示例：ul em {font-size: 12px}; <!--匹配ul的后代元素em,em不一定是ul的子元素-->
```
2.子元素选择器：只能从父元素找到子元素，元素之间用 '>'
```css
代码示例：h1 > p {font-size: 13px}; <!--匹配所有h1元素的子元素p-->
```
3.相邻兄弟选择器：元素之间是同辈的，同一个父元素
```css
代码示例：h1 + p {font-size: 13px}; <!--匹配h1后的所有p元素，却h1和p具有相同的父元素-->
```
###### 4.伪类和伪元素
1.伪类选择器：
>超链接伪类：具有href属性的a(锚)标签
```css
  a:link   未访问的a标签
  a:visited   已访问的a标签
```
>动态伪类：可以根据用户行为应用不同的样式
```css
  a:hover  鼠标停留在a标签时应用样式
  a:active   被用户激活的a标签应用该样式
  input:focus   元素获得焦点时应用样式
```
>静态伪类：选择第一个子元素
```css
  p:first-child {font-weight: bold}
```
2.伪元素选择器
```css
  设置首字母样式
  p:first-letter {color: red}
  设置第一行样式
  p:first-line {color: red}
  设置之前和之后的样式
  p:after {content: 'The Begin'}
  p:before {content: 'The End'}
```
