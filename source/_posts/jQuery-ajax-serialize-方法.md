---
title: jQuery-ajax-serialize()方法
date: 2019-02-21 15:44:07
tags: [Ajax、笔记]
categories: [JS]
toc: true
---

#### html代码段：

```html
<form action="index" method="post">
  <div><input type="text" name="a" value="1" id="a" /></div>
  <div><input type="text" name="b" value="2" id="b" /></div>
  <div><input type="hidden" name="c" value="3" id="c" /></div>
  <div>
    <textarea name="d" rows="8" cols="40">4</textarea>
  </div>
  <div><select name="e">
    <option value="5" selected="selected">5</option>
    <option value="6">6</option>
    <option value="7">7</option>
  </select></div>
  <div>
    <input type="checkbox" name="f" value="8" id="f" />
  </div>
  <div>
    <input type="submit" name="g" value="Submit" id="g" />
  </div>
</form>
```
#### 普通ajax提交:

```js
$.ajax(function(){
    url:'/index/index?_t=' + Math.random(),
    type:'post',
    dataType:'json',
    data:{
        a:$("input[name='a']").val,
        b:$("input[name='b']").val,
        c:$("input[name='c']").val,
        d:$("input[name='d']").val,
        e:$("input[name='e']").val,
        f:$("input[name='f']").val,
        
            },
    success:function(data){
        // TODO...
    },
    error:function(){
        // TODO...
    }
});
```

#### 使用jQuery ajax - serialize() 方法

```js
 $.ajax(function(){

    url:'/index/index?_t=' + Math.random(),

    type:'post',

    dataType:'json',

    data:$("form").serialize(),
    success:function(data){
        // TODO...
    },

    error:function(){
        // TODO...
    }
});
```
可以看到，使用jQuery `serialize()` 序列化 ，使得 表单数据的提交要方便得多。
