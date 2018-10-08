#### css笔记

##### 常用CSS初始化样式

重新设置浏览器的*样式*,但是因浏览器兼容的问题,不同的浏览器对标签的默认属性有不同的默认值

常见参考：https://www.jianshu.com/p/beabef833bfe

**腾讯官网 样式初始化**

```css
body,ol,ul,h1,h2,h3,h4,h5,h6,p,th,td,dl,dd,form,fieldset,legend,input,textarea,select{margin:0;padding:0} 
body{font:12px"宋体","Arial Narrow",HELVETICA;background:#fff;-webkit-text-size-adjust:100%;} 
a{color:#2d374b;text-decoration:none} 
a:hover{color:#cd0200;text-decoration:underline} 
em{font-style:normal} 
li{list-style:none} 
img{border:0;vertical-align:middle} 
table{border-collapse:collapse;border-spacing:0} 
p{word-wrap:break-word} 
```

