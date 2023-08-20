---
layout:     post
title:      牛客网刷题技巧
subtitle:  
date:       2019-06-17
author:     
header-img: 
catalog: true
tags:
    - < 算法题 >
typora-root-url: ..
---



# 牛客网刷题技巧
#### JavaScript(V8)

```javascript
//单行输入，单行输出
sc=readline()
var arr = sc.split(' ');
n=parseInt(arr[0]);
m=parseInt(arr[1]);
print(m+n)
//输入
3 4
//输出
7
```

```javascript
//多行输入，多行输出
while (sc = readline()) {
     var arr = sc.split(' ');
     n = parseInt(arr[0]);
     m = parseInt(arr[1]);
     print(m+n);
}
//输入
3 4
5 6
//输出
7
11
```

```javascript
//多行输入，多行输出
var n = parseInt(readline());
var ans = 0;
for (var i = 0; i < n; i++) {
    lines = readline().split(" ")
    for (var j = 0; j < lines.length; j++) {
      ans = lines[j];
      print(ans);
      print(' ');
    }
    print('\n')    
}
//输入
2
3 4
5 6
//输出
3 4 
5 6 
```



### 牛客网帮助文档里面复制的

```javascript
	while (line = readline()) {
    //在这里写代码
    var lines = line.split(' ');
    var a = parseInt(lines[0]);
    var b = parseInt(lines[1]);
    print(a + b);
  }
```

```javascript
//如果是函数的话：
  while (line = readline()) {
    var lines = line.split(' ');
    var a = parseInt(lines[0]);
    var b = parseInt(lines[1]);

    function add(m, n) {
      return m + n;
    }
    print(add(a, b));
  }
```

```javascript
  //多行输入举例
  //打印一个多行矩阵
  var n = parseInt(readline());
  var ans = 0;
  for (var i = 0; i < n; i++) {
    lines = readline().split(" ")
    for (var j = 0; j < lines.length; j++) {
      ans += 
          (lines[j]);
    }
    print(ans);
  }
```

```javascript
  //JavaScript(Node)模拟输入
  var readline = require('readline');
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
  });
  rl.on('line', function (line) {
    //在这里写代码
    var tokens = line.split(' ');
    console.log(parseInt(tokens[0]) + parseInt(tokens[1]));
  });
```

```javascript
  //node实现多行输入（ 固定行数）
  var readline = require('readline');
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
  });
  var countLine = 1;
  var tokens = [];
  rl.on('line', function (line) {
    tokens.push(line);
    if (countLine == 2) {
      var arr1 = tokens[0].split('');
      var arr2 = tokens[1].split('');
      for (var i = 0; i < arr2.length; i++) {
        for (var j = 0; j < arr1.length; j++) {
          if (arr1[j] == arr2[i]) {
            arr1.splice(j, 1);
          }
        }
      }
      console.log(arr1.join(''));
    } else {
      countLine++;
    }
  });

  var readline = require('readline');

  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
  });

  var K = 2;
  var arr = [];
  rl.on('line', function (data) {
    arr.push(data);
    if (K == arr.length) {
      var result = deal(arr);
      console.log(result);
      arr.length = 0;

    }
  });

  function deal(inputs) {
    //直接根据目标字符分割字符串成数组，计算数组长度减一就是所求。注意不区分大小写。
    return inputs[0].toLowerCase().split(inputs[1].toLowerCase()).length - 1;
  }
```

```javascript
 Node实现多行输入（ 行数不固定）
  process.stdin.resume();
  process.stdin.setEncoding('ascii');

  var input = "";
  var input_array = "";

  process.stdin.on('data', function (data) {
    input += data;
  });

  process.stdin.on('end', function () {
    input_array = input.split("\n"); //示例代码
    var len = input_array.length;
    var result = [];
    for (var i = 0; i < len; i++) {
      var temp = input_array[i].trim().split(' ');
      for (var j = 0; j < temp.length; j++) {
        if (temp[j] !== '' && result.indexOf(temp[j]) == -1) {
          result.push(temp[j]);
        }
      }
    }
    console.log(result.length);
  });
```
