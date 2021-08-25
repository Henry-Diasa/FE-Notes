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

