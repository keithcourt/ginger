## css 清除浮动

```css
.clearfix:after {

content: ".";

display: block;

height: 0;

clear: both;

visibility: hidden;

}

/* Hides from IE-mac \*/

* html .clearfix {height: 1%;}

/* End hide from IE-mac */
```
