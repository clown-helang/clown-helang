---
title: Javascript常用工具方法
catalog: true
date: 2018-11-03 13:17:58
subtitle: 日常工具
header-img: "/img/article_header/article_header.png"
tags:
- Javascript
---

## 常用工具方法
记录工作中常用的JavaScript工具方法

### 判空、判真

```
export function isEmpty(obj){
  if(typeof obj === 'object'){
    if(!obj){
      return true;
    }
    return !(JSON.stringify(obj) !== '{}' && JSON.stringify(obj) !== '[]' && Boolean(obj))
  } else if(typeof obj === 'number'){
      return false
  }
  return !(Boolean(obj) && obj !== '')
}

```

### session 操作

```
export function setSession(_key,_value){
  sessionStorage.setItem(_key, JSON.stringify(_value));
}

export function getSession(_key){
  return JSON.parse(sessionStorage.getItem(_key));
}

```

### localStorage 操作

```
export function setLocalStorage(_key,_value){
  localStorage.setItem(_key, JSON.stringify(_value));
}

export function getLocalStorage(_key){
  return JSON.parse(localStorage.getItem(_key));
}

```

### 字符串操作
```
//获取指定字符
export function getBytes(_str){
	var bytes = [];
    for(var i=0;i<_str.length;i++){
		bytes[i] = _str.charCodeAt(i);
	}
	return bytes;
}

//比较字符串长度
export function compareLength(rule, value, callback){
  if (value) {
    let len;
    if (value == null)
    {
      len=0;
    }
    if (typeof value != "string"){
      value += "";
    }
    len = value.replace(/[^\x00-\xff]/g,"01").length;
    if (len>rule.len){
      callback(rule.message);
    }else{
      callback();
    }
  }else{
    callback();
  }
}

//获取字符串长度 中文占一个 英文占0.5个
export function get_length(s){ 
  if(isEmpty(s))
    return 0;
  else {
    var char_length = 0;
    for (var i = 0; i < s.length; i++){
      var son_char = s.charAt(i);
      if(son_char == ' '){
        char_length += 0.5;
      }else{
        encodeURI(son_char).length > 2 ? char_length += 1 : char_length += 0.5;
      }
    }
    return char_length;
  }
}

//截取字符串 从str 中截取len 个字符，中文占一个 英文占0.5个
export function cut_str(str, len){
  var char_length = 0;
  for (var i = 0; i < str.length; i++){
    var son_str = str.charAt(i);
    if(son_str == ' '){
      char_length += 0.5;
    }else{
      encodeURI(son_str).length > 2 ? char_length += 1 : char_length += 0.5;
    }
    if (char_length >= len){
      var sub_len = char_length == len ? i+1 : i;
      return str.substr(0, sub_len);
      break;
    }
  }
}

//去掉左边的空白
export function trimLeft(s){
  if(s == null) {
    return "";
  }
  var whitespace = new String(" \t\n\r");
  var str = new String(s);
  if (whitespace.indexOf(str.charAt(0)) != -1) {
    var j=0, i = str.length;
    while (j < i && whitespace.indexOf(str.charAt(j)) != -1){
      j++;
    }
    str = str.substring(j, i);
  }
  return str;
}

//去掉右边的空白
export function trimRight(s){
  if(s == null) return "";
  var whitespace = new String(" \t\n\r");
  var str = new String(s);
  if (whitespace.indexOf(str.charAt(str.length-1)) != -1){
    var i = str.length - 1;
    while (i >= 0 && whitespace.indexOf(str.charAt(i)) != -1){
      i--;
    }
    str = str.substring(0, i+1);
  }
  return str;
}

//去掉前后空格
export function trim(s){
  return trimRight(trimLeft(s));
}

```
### 数组去重

```
export function unique(arr){//数组去重
  let res = [];
  let json = {};
  for(var i = 0; i < arr.length; i++){
    if(!json[arr[i]]){
      res.push(arr[i]);
      json[arr[i]] = 1;
    }
  }
  return res;
}
```
### 时间格式化

```

Date.prototype.format = function (fmt) {
  var o = {
    "M+": this.getMonth() + 1, //月份
    "d+": this.getDate(), //日
    "H+": this.getHours(), //小时
    "m+": this.getMinutes(), //分
    "s+": this.getSeconds(), //秒
    "q+": Math.floor((this.getMonth() + 3) / 3), //季度
    "S": this.getMilliseconds() //毫秒
  };
  if (/(y+)/.test(fmt)) fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 - RegExp.$1.length));
  for (var k in o)
    if (new RegExp("(" + k + ")").test(fmt)) fmt = fmt.replace(RegExp.$1, (RegExp.$1.length === 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
  return fmt;
};

```
使用方式：
```
alert(new Date().format("yyyy-MM-dd HH:mm:ss")); 
```



### 树形结构格式化

```
/*
* data: 带有 parent_id 的数组对象，如下所示：
*/

function toTree(data) {
  var val = [];
  // 删除 所有 children,以防止多次调用
  if(data!==undefined&&data.length>0){
    data.forEach(function (item) {
      delete item.children;
    });
    // 将数据存储为 以 id 为 KEY 的 map 索引数据列
    var map = {};
    let _data = data.sort(compare("show_order"))
    _data.forEach(function (item) {
      map[item.id] = item;
    });
    _data.forEach(function (item) {
      // 以当前遍历项的parent_id,去map对象中找到索引的id
      var parent = map[item.parent_id];
      // 如果找到索引，那么说明此项不在顶级当中,那么需要把此项添加到，他对应的父级中
      if (parent) {
        (parent.children || ( parent.children = [] )).push(item);
      } else {
        //如果没有在map中找到对应的索引ID,那么直接把 当前的item添加到 val结果集中，作为顶级
        val.push(item);
      }
    });
  }
  return val;
}

function compare(property){
  return function(obj1,obj2){
    let value1 = obj1[property];
    let value2 = obj2[property];
    return value1 - value2;     // 升序
  }
}
```
使用方式：

```
var data = [
    {
        "id": 29,
        "name": "电视",
        "parent_id": -1,
    },
    {
        "id": 30,
        "name": "景点",
        "parent_id": -1,
    },
    {
        "id": 49,
        "name": "餐饮",
        "parent_id": 30,
    },
  ];
  
toTree(data);
```

返回结果：

```
[
    {
        "id": 29,
        "name": "电视",
        "parent_id": -1
    },
    {
        "id": 30,
        "name": "景点",
        "parent_id": -1,
        "children": [
            {
                "id": 49,
                "name": "餐饮",
                "parent_id": 30
            }
        ]
    }
]

```


