#### 超出文本省略号显示

**单行**

```
p{
	width: 200px;
	overflow:hidden;
	white-space:nowrap;
	text-overflow:ellipsis
}
```

**多行**

```
p {
	-webkit-line-clamp: 2;
  display: -webkit-box;
  overflow: hidden;
  -webkit-box-orient: vertical;
  text—overflow:ellipse
}
```

#### 自适应正方形

```css
<div class="placeholder"></div>

// 方案1
.placeholder {
	width: 100%
	height: 100vw
}

// 方案2
.placeholder{
  height: 0
	width: 100%
	padding-bottom: 100%
}

// 方案3

.placeholder {
  width: 100%;
  overflow: hidden
}

.placeholder:after {
  content: '';
  display: block;
  margin-top: 100%; /* margin 百分比相对父元素宽度计算 */
}

```

