---
title: JavaScript常用脚本
date: 2014-02-13 21:28:01
comments: true
categories: JavaScript
toc: true 
---

## 字符串函数

### 是否包含
```javascript
String.prototype.Contains = function (A) {
    return (this.indexOf(A) > -1);
};
```
<!--more-->
### 是否相等
```javascript
String.prototype.Equals = function () {
    var A = arguments;
    if (A.length == 1 && A[0].pop)
        A = A[0];
    for (var i = 0; i < A.length; i++) {
        if (this == A[i])
            return true;
    };
    return false;
};
```

### 是否相等(不区分大小写)
```javascript
String.prototype.IEquals = function () {
    var A = this.toUpperCase();
    var B = arguments;
    if (B.length == 1 && B[0].pop)
        B = B[0];
    for (var i = 0; i < B.length; i++) {
        if (A == B[i].toUpperCase())
            return true;
    };
    return false;
};
```

### 替换所有
```javascript
String.prototype.ReplaceAll = function (A, B) {
    var C = this;
    for (var i = 0; i < A.length; i++) {
        C = C.replace(A[i], B[i]);
    };
    return C;
}; 
或
String.prototype.ReplaceAll = function(A,B){
       var C = this;
       C=C.replace(/A/g,B);
       return C;  
}
```

### StartsWith
```javascript
String.prototype.StartsWith = function (A) {
    return (this.substr(0, A.length) == A);
};
```

### EndsWith
```javascript
String.prototype.EndsWith = function (A, B) {
    var C = this.length;
    var D = A.length;
    if (D > C)
        return false;
    if (B) {
        var E = new RegExp(A + '$', 'i');
        return E.test(this);
    } else
        return (D == 0 || this.substr(C - D, D) == A);
};
```

### Remove
```javascript
String.prototype.Remove = function (A, B) {
    var s = '';
    if (A > 0)
        s = this.substring(0, A);
    if (A + B < this.length)
        s += this.substring(A + B, this.length);
    return s;
};
```

### Trim
```javascript
String.prototype.Trim = function () {
    return this.replace(/(^[ \t\n\r]*)|([ \t\n\r]*$)/g, '');
};
```

### LTrim
```javascript
String.prototype.LTrim = function () {
    return this.replace(/^[ \t\n\r]*/g, '');
};
```

### RTrim
```javascript
String.prototype.RTrim = function () {
    return this.replace(/[ \t\n\r]*$/g, '');
}
```

### ReplaceNewLineChars
```javascript
String.prototype.ReplaceNewLineChars = function (A) {
    return this.replace(/\n/g, A);
};
```

### Replace
```javascript
String.prototype.Replace = function (A, B, C) {
    if (typeof B == 'function') {
        return this.replace(A, function () {
            return B.apply(C || this, arguments);
        });
    } else
        return this.replace(A, B);
};
```

## 获取浏览器信息
```javascript
 var gecko = /gecko\/\d/i.test(navigator.userAgent);
 var ie = /MSIE \d/.test(navigator.userAgent);
 var ie_lt8 = ie && (document.documentMode == null || document.documentMode < 8);
 var ie_lt9 = ie && (document.documentMode == null || document.documentMode < 9);
 var webkit = /WebKit\//.test(navigator.userAgent);
 var qtwebkit = webkit && /Qt\/\d+\.\d+/.test(navigator.userAgent);
 var chrome = /Chrome\//.test(navigator.userAgent);
 var opera = /Opera\//.test(navigator.userAgent);
 var safari = /Apple Computer/.test(navigator.vendor);
 var khtml = /KHTML\//.test(navigator.userAgent);
 var mac_geLion = /Mac OS X 1\d\D([7-9]|\d\d)\D/.test(navigator.userAgent);
 var mac_geMountainLion = /Mac OS X 1\d\D([8-9]|\d\d)\D/.test(navigator.userAgent);
 var phantom = /PhantomJS/.test(navigator.userAgent);
 ```
 
 ## 获取操作系统信息
 ```javascript
 var ios = /AppleWebKit/.test(navigator.userAgent) && /Mobile\/\w+/.test(navigator.userAgent);
 // This is woefully incomplete. Suggestions for alternative methods welcome.
 var mobile = ios || /Android|webOS|BlackBerry|Opera Mini|Opera Mobi|IEMobile/i.test(navigator.userAgent);
 var mac = ios || /Mac/.test(navigator.platform);
 var windows = /win/i.test(navigator.platform);
 ```