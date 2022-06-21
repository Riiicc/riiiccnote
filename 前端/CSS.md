https://zh.learnlayout.com/no-layout.html
### CSS布局

#### display

display 是CSS中最重要的用于控制布局的属性.每个元素都有一个默认的 display 值，这与元素的类型有关。对于大多数元素它们的默认值通常是 block 或 inline 。一个 block 元素通常被叫做**块级元素**。一个 inline 元素通常被叫做**行内元素**。   

#### block
> div 是一个标准的**块级元素**。一个块级元素会新开始一行并且尽可能撑满容器。其他常用的块级元素包括 p 、 form 和HTML5中的新元素： header 、 footer 、 section 等等。

#### inline
>span 是一个标准的**行内元素**。一个行内元素可以在段落中 <span> 像这样 </span> 包裹一些文字而不会打乱段落的布局。 a 元素是最常用的行内元素，它可以被用作链接。

#### none 
> 另一个常用的display值是 none 。一些特殊元素的默认 display 值是它，例如 script 。 display:none 通常被 JavaScript 用来在不删除元素的情况下隐藏或显示元素。

> display:none 不占据空间，visibility:hidden;还会占据空间  

#### width
>设置块级元素的 width 可以防止它从左到右撑满整个容器。然后你就可以设置左右外边距为 auto 来使其水平居中
```css
#main {
  width: 600px;
  margin: 0 auto; 
}
```
> max-width 替代 width 可以使浏览器更好地处理小窗口的情况
```css
#main {
  max-width: 600px;
  margin: 0 auto; 
}
```

#### 盒子模型
> box-sizing F12右下角 

#### position 
> 默认为 position: static;  

> position: relative; relative 表现的和 static 一样，除非你添加了一些额外的属性,在一个相对定位（position属性的值为relative）的元素上设置 top 、 right 、 bottom 和 left 属性会使其偏离其正常位置。   

> fixed 一个固定定位（position属性的值为fixed）元素会相对于视窗来定位，这意味着即便页面滚动，它还是会停留在相同的位置。和 relative 一样， top 、 right 、 bottom 和 left 属性都可用

> absolute  absolute 与 fixed 的表现类似，但是它不是相对于视窗而是相对于最近的“positioned”祖先元素。如果绝对定位（position属性的值为absolute）的元素没有“positioned”祖先元素，那么它是相对于文档的 body 元素，并且它会随着页面滚动而移动。记住一个“positioned”元素是指 position 值不是 static 的元素


#### float
> float 可以用户实现文字环绕   

> clear 属性可以用于控制浮动,可以用 left right 或 both 来清除向右浮动或同时清除向左向右浮动,使用overflow:auto 可以防止浮动元素溢出

```css
.box {
  float: left;
  width: 200px;
  height: 100px;
  margin: 1em;
}
.after-box {
  clear: left;
  /* 
  clear: right;
  clear: both;
  */
}

.clearfix {
  overflow: auto;
}
```