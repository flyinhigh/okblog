title: 算法的乐趣
date: 2015-11-13 21:15:58
tags: 现实
---

写代码时碰到的一些小而美的算法，就喜欢这样的思路，来分享一下。

## BKMGTPEZYBND
### 背景
可能有人会担心，硬盘要是超过 T 了怎么办，表紧，计算机存储单位已经目前准备到了2的110次方。不过在Unix系统中已经采用十进制，Windows则还保留二进制的方式，这一点会导致同一个文件在Windows和Unix上的大小不同，但这种差异已然可以忽略不计。

下表的冥显示用了[MathJax](https://www.mathjax.org/)，它的CDN不如我们阿里云的强大，刷不出来可以多刷几次。


| 单位 | 换算 | 二进制 | 十进制 |
| --------   | -----  | -----  |-----  |
| 1B (byte) | 1B | $$2^{0}$$ | $$10^{0}$$ |
| 1KB(Kilobyte) | 1024B | $$2^{10}$$ | $$10^{3}$$ |
| 1MB(Megabyte) | 1024KB |$$2^{20}$$ |$$10^{4}$$ |
| 1GB(Gigabyte) | 1024MB |$$2^{30}$$ |$$10^{5}$$ |
| 1TB(Trillionbyte) | 1024GB |$$2^{40}$$ |$$10^{6}$$ |
| 1PB(Petabyte) | 1024TB |$$2^{50}$$ |$$10^{7}$$ |
| 1EB(Exabyte) | 1024PB |$$2^{60}$$ |$$10^{8}$$ |
| 1ZB(Zettabyte) | 1024EB |$$2^{70}$$ |$$10^{9}$$ |
| 1YB(YottaByte) | 1024ZB |$$2^{80}$$ |$$10^{10}$$ |
| 1BB(Brontobyte) | 1024YB |$$2^{90}$$ |$$10^{11}$$ |
| 1NB(NonaByte） | 1024BB |$$2^{100}$$ |$$10^{12}$$ |
| 1DB(DoggaByte) | 1024NB |$$2^{110}$$ |$$10^{13}$$ |
据百度百科所言，最后一级基本上已经可以标注地球上所有的分子了，短时间应该够用了。

### 问题
如果接口只会吐字节数，比如12123123123 bit，而这个数是不确定的，你需要将其换算成对应的单位，怎么做？
假设这里采用十进制，最大单位到YB。

### 答案
普通的写法:

```
function transfer(bits){
  if(bits === 0) return '0 B';
  bits /= 8;
  var x = '';
  
  if(Math.round(bits/(Math.pow(10,3))) == 0){
    return x + bits.toFixed(2) + 'B';
  }
  else if(Math.round(bits/(Math.pow(10,3))) < 10){
    return x + (bits/(Math.pow(10,3))).toFixed(2) + 'KB';
  }
  else if(Math.round(bits/(Math.pow(10,4))) < 10){
    return x + (bits/(Math.pow(10,4))).toFixed(2) + 'MB';
  }
  else if(Math.round(bits/(Math.pow(10,5))) < 10){
    return x + (bits/(Math.pow(10,5))).toFixed(2) + 'GB';
  }
  else if(Math.round(bits/(Math.pow(10,6))) < 10){
    return x + (bits/(Math.pow(10,6))).toFixed(2) + 'TB';
  }
  else if(Math.round(bits/(Math.pow(10,7))) < 10){
    return x + (bits/(Math.pow(10,7))).toFixed(2) + 'PB';
  }
  else if(Math.round(bits/(Math.pow(10,8))) < 10){
    return x + (bits/(Math.pow(10,7))).toFixed(2) + 'EB';
  }
  else if(Math.round(bits/(Math.pow(10,9))) < 10){
    return x + (bits/(Math.pow(10,7))).toFixed(2) + 'ZB';
  }
  else{
    return x + (bits/(Math.pow(10,7))).toFixed(2) + 'YB';
  }
}
```

然后我们看下文艺的写法。

```
function transfer(bits){
  if(bits === 0) return '0 B';
  bits /= 8;
  var x = '';
  var k = 1000;
  var units = ['B', 'KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB'];
  //通过对数函数计算出属于哪个数量级
  var i = Math.floor(Math.log10(bits) / Math.log10(k));
  //再通过冥函数计算出该数量级，除以数量级即得到结果
  return x + (bits / Math.pow(k, i)).toFixed(2) + ' ' + units[i];

}
```
运行效果：
<a class="jsbin-embed" href="http://jsbin.com/zilesezeze/embed?html,js,console">JS Bin on jsbin.com</a><script src="http://static.jsbin.com/js/embed.min.js?3.35.3"></script>

看不见？请[戳这里](http://jsbin.com/zilesezeze/edit?html,js,console)

如果想换成二进制把变量`k`设为1024，然后对数函数换成`Math.log`就ok了，你想到什么级别都可以。是不是很惊艳很想去找本高等数学来翻？

你哭着对我说，数学是有用的！！

## 补零
### 背景
现在后端一起吐出三段数据：
```
var data = {
  a: [[1,1],[3,3],[7,7]],
  b: [[2,2],[3,3],[8,8]],
  c: [[1,1],[4,4],[8,8]]
}
```
每段是个二维数组，元素是[index,value]
### 问题
现在要求需要给每段数据中缺失的那块补零，并且从小到大排列，意思就是要得到：
```
var data = {
  a: [[1,1],[2,0],[3,3],[4,0],[7,7],[8,0]],
  b: [[1,0],[2,2],[3,3],[4,0],[7,0],[8,8]],
  c: [[1,1],[2,0],[3,0],[4,4],[7,0],[8,8]]
}
```
程序该怎么写？

### 答案
```
function zero(data){
  //第一步，收集所有的Index并排序
  var xIndex = [];
  $.each(data, function(key, value){
    $.each(value, function(idx, item){
      if ($.inArray(item[0], xIndex) === -1) {
        xIndex.push(item[0]);
      }
    });
  });
  xIndex.sort(function (a, b) {
    return a - b;
  });
  //第二步，遍历遍历。
  $.each(data, function(key, value){
    var thisIndex = {}, cache = $.extend([],value);
    //先搜集每段数据的Index
    $.each(value, function(idx,item){
      thisIndex[item[0]] = idx;
    })
    //再将每段数据的Index与汇总的Index比较，没有的补零
    $.each(xIndex, function(k, v){
      if(thisIndex[v] === undefined){
        cache.push([v, 0]);
      }
    })
    //排序
    cache.sort(function (a, b) {
      return a[0] - b[0];
    });
    //赋值，注意数组是对象类型，不能直接赋值
    data[key] = $.extend([], cache);
  }); 
  return data;
}
```

思路清晰，代码运行效果：
<a class="jsbin-embed" href="http://jsbin.com/wotuzukasu/1/embed?html,js,console">JS Bin on jsbin.com</a><script src="http://static.jsbin.com/js/embed.min.js?3.35.3"></script>

看不见？请[戳这里](http://jsbin.com/holotilije/edit?html,js,console)

肯定会有更好的解法，请不吝赐教。

好了就写两个，太长了也懒得看，碰到新的再补充~

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>
