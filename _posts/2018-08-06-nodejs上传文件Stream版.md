---
layout: post
title:  "nodejs上传文件Stream版"
date:   2018-08-06
categories: [front-end]
---

1. 读入流  fs.createReadStream  ,  req
2. 写入流 fs.createWriteStream , res
3. 读写流 gz

操作流时常用的事件和方法：

![stream1](/Users/cail/Downloads/stream1.png)



可以使用管道pipe连接，pipe的入口必须是读入流或者读写流，出口必须是写入流或者读写流



stream.js

```javascript
const http = require('http');
const fs = require('fs');
const zlib = require('zlib');

let server = http.createServer((req, res) => {
	let file = fs.createReadStream('www/jquery-1.11.3.min.js');
	let savedFile = fs.createWriteStream('www/savedfile');

	res.setHeader('content-encoding', 'gzip');

	// pipe的输入是读入流，输出是写入流
	rs = file.pipe(zlib.createGzip()).pipe(savedFile);

	rs.on('error', err => {
		res.writeHeader(404);
		res.write('failed');

		res.end();
	})
})

server.listen(8080);
```



reference:

1. 一篇很好的“流”的入门文章

   https://juejin.im/post/5940a9c3128fe1006a0ab176