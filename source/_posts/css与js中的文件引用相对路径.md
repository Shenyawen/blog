---
title: css与js中的文件引用相对路径
date: 2018-08-30 17:45:42
tags: H5
---

javascript和css文件中采用相对路径引用图片等静态资源时其基准路径是完全不同的。

1.在javascript文件中引用静态资源相对路径是以宿主路径(即应用该js文件的html文件)所处的位置为相对位置基准的

2.在css文件引用静态资源(background-image url等)相对路径是以css文件所处位置为基准的

假设项目目录如下
-app
——images
————index.png
——js
————index.js
——css
————index.css
——index.html

### index.css
```css
// index.css

.image {
	background-image: url(../images/index.png)
}
```

### index.js
```js
// index.js

document.write("<img src='./images/index.png' />"); 
```

### index.html
```html
<!DOCTYPE html PUBLIC “-//W3C//DTD XHTML 1.0 Transitional//EN” “http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd”> 
<html > 
	<head> 
	<title>test</title> 
	<script type=”text/javascript” src=”../js/index.js”></script> 
	<link href=”../css/index.css” rel=”stylesheet” type=”text/css” /> 
	</head> 
	<body> 
	</body> 
</html> 
```
很明显看出区别：***在js文件中引入图片等静态资源时相对路径的基准路径是引用其的html文件，在css文件中则是按其本身为基准路径***