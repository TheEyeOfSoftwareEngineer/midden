## Grid

### 介绍
CSS Grid Layout (aka “Grid” or “CSS Grid”), is a two-dimensional grid-based layout system that, compared to any web layout system of the past, completely changes the way we design user interfaces. CSS has always been used to layout our web pages, but it’s never done a very good job of it. First, we used tables, then floats, positioning and inline-block, but all of these methods were essentially hacks and left out a lot of important functionality (vertical centering, for instance). Flexbox is also a very great layout tool, but its one-directional flow has different use cases — and they actually work together quite well! Grid is the very first CSS module created specifically to solve the layout problems we’ve all been hacking our way around for as long as we’ve been making websites.
> table float position inline-bolck都比较黑客的方式去解决布局问题，总会有各种各样的问题（比如居中）；grid相比于flex是一种二维布局的方式

### Grid术语
- 网格容器: `display: grid;`
- 网格线
- 网格单元
- 网格行
- 网格列
- 网格轨道
- 网格区
- 网格项

### 定义行和列
```css
.wrapper {
  display: grid;
  grid-template-rows: 300px 300px;
  grid-template-columns: 1fr 1fr 1fr 1fr;
}
```
这段代码定义了一个2行4列的网格，行高300像素，4列等宽，而且该网格中每行和每列的边缘都会生成网格线。其中`fr`意思是可用空间的部分(fraction of available space)。可用空间就是网格轨道（通过明确指定的长度值或根据自己的内容）确定尺寸后的剩余空间。每个fr单位表示网格可用空间的1/4.假如再添加一个1fr则每个fr单位表示的就是可用空间的1/5.

指定行和列的数量及大小时，可以混用不同的长度单位，比如声明列的时候可以写为`200px 20% 1fr 200%`，表示靠两边的列宽度固定为200像素，左起第二列的宽度是总空间的20%，而第三列则占据剩下的全部空间。fr，和flexbox一样，其大小会在计算完其他长度值之后再确定。






### References
![CSS Trick - Grid](https://css-tricks.com/snippets/css/complete-guide-grid/#introduction)