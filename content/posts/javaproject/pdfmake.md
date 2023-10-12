---
title: "Pdfmake"
date: 2023-10-12T21:33:55+08:00
description: "pdfmake生成pdf指南"
tags: [pdfmake]
categories: javascript
draft: false
---


##

最近有个需求需要根据一些字段生成pdf，准备在前端做这个功能，使用[pdfmake](https://pdfmake.github.io/docs/0.1/)生成pdf。简单记录一下使用过程

## 安装
```shell
npm install pdfmake
```

## 简单使用
### 导入
直接在js文件中导入
```javascript
import pdfMake from "pdfmake/build/pdfmake";
import pdfFonts from "pdfmake/build/vfs_fonts";
pdfMake.vfs = pdfFonts.pdfMake.vfs;
```
内容部分（示例）：
```javascript
var docDefinition = {
	content: [
		'第一段',
		'另一段，这次要长一点，这一行至少要分成两行。Another paragraph, this time a little bit longer to make sure, this line will be divided into at least two lines',
        {
			text: '这是一个header，使用header样式',
			style: 'header'
		},
        'Lorem ipsum dolor sit amet, consectetur adipisicing elit. Confectum ponit legam, perferendis nomine miserum, animi. Moveat nesciunt triari naturam.\n\n',
		{
			text: 'Subheader 1 - using subheader style',
			style: 'subheader'
		},
        {
			style: 'tableExample',
			table: {
				body: [
					['Column 1', 'Column 2', 'Column 3'],
					['One value goes here', 'Another one here', 'OK?']
				]
			}
		},
        'pdfmake (since it\'s based on pdfkit) supports JPEG and PNG format',
		'If no width/height/fit is provided, image original size will be used',
        '如果没有提供宽度/高度/适合度，将使用图像原始尺寸',
		{
			image: 'sampleImage.jpg',
		},
		'If you specify width, image will scale proportionally',
		{
			image: 'sampleImage.jpg',
			width: 150
		},
        'You can also fit the image inside a rectangle',
		{
			image: 'sampleImage.jpg',
			fit: [100, 100],
			pageBreak: 'after'
		},
        'Images can be also provided in dataURL format...',
	],
    styles: {
		header: {
			fontSize: 18,
			bold: true
		},
		subheader: {
			fontSize: 15,
			bold: true
		}，
        tableExample: {
			margin: [0, 5, 0, 15]
		},
    }
	
}

```
效果如下：  
第一页：
![](/img/pdfmake/example_1.jpg)
第二页：
![](/img/pdfmake/example_2.jpg)


### 常用方法
下载
```javascript
pdfMake.createPdf(docDefinition).download();
```

新窗口打开
```javascript
pdfMake.createPdf(docDefinition).open();
```

打印
```javascript
pdfMake.createPdf(docDefinition).print();
```



## 格式
详见[官方示例](http://pdfmake.org/playground.html)



## 进阶

### 自定义字体
1. 需要进入到前端项目的**node_modules**目录下，找到*pdfmake*的目录，如果该目录下没有*examples/fronts*目录，新建一个。将你的字体文件粘贴到这个目录中(*xxx字体.ttf*).
2. 命令行进入到**node_modules/pdfmake**目录，执行命令
   ```bash
   node build-vfs.js "./examples/fonts"
   ```
3. 在你的js文件中引入该字体，只需要填写名称即可。代码示例（导入的是微软雅黑字体）：
   ```javascript
    pdfMake.fonts = {
        微软雅黑: {
          normal: "微软雅黑.ttf",
          bold: "微软雅黑.ttf",
          italics: "微软雅黑.ttf",
          bolditalics: "微软雅黑.ttf",
        },
      }
    <!-- 之后在你的docDefinition中使用它 -->
    defaultStyle: {
        font: "微软雅黑",
      },
   ```


