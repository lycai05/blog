---
layout: post
title:  "nodejs上传文件Buffer版"
date:   2018-08-05
categories: [front-end]
---

目标：

1. 理解二进制文件上传的原理
2. 学会使用buffer对象



重点：

根据数据的结构一步步的解析得到的buffer对象



使用buffer上传文件的示例：

upload.html

```html
<form action="http://localhost:8080/upload" method="post" enctype="multipart/form-data">
    Name: <input type="text" name="user" id="user" /><br>
    Password: <input type="password" name="password" id="password" /><br>
    <input type="file" name="upload" >
    <input type="submit" value="login" >
</form>
```

server.js

```javascript
const http = require('http');
const uuid = require('uuid/v4');
const fs = require('fs');

// add a split function for Buffer object
Buffer.prototype.split = function(d) {
	let arr = [];
	let cur = 0;

	while(this.indexOf(d, cur) != -1) {
		arr.push(this.slice(cur, this.indexOf(d, cur)));
		cur = this.indexOf(d, cur) + d.length;
	}

	return arr;
}

let server = http.createServer((req, res)=>{
	
	// 将数据暂时储存在数组里
	let arr = [];
	req.on('data', (data) => {
		arr.push(data);
	});

	req.on('end', () => {
		// 将数组转换成buffer对象
		let buff = Buffer.concat(arr);
		// 显示buffer
		//console.log(buff);
		// 二进制文件会显示乱码
		//console.log(buff.toString());

		// 解析buffer
		//  test if content-type exists
		let files = {};
		if (req.headers['content-type']) {

			let d = '--' + req.headers['content-type'].split('=')[1];

			// split based on d;
			let arr1 = buff.split(d);

			// delete head element
			arr1.shift();

			// delete heading ','
			arr1 = arr1.map(function(x){return x.slice(2, x.length - 2)});

			arr1.forEach(function(x) {
				let n = x.indexOf('\n');
				let meta = x.slice(0, n);
				let data = x.slice(n+1);

				meta = meta.toString();

				// file data
				if (meta.match('filename')) {
					let n = data.indexOf('\n');
					data = data.slice(n+1);

					// 设置上传路径
					let path = `upload/${uuid()}`;
					// 保存上传的文件
					fs.writeFile(path, data, err => {
						if (err) {
							console.log("upload failed");
						} else {
							files[meta] = {meta, path};
							console.log("upload success");
						}
					})
        
				// other text data
				} else {
					meta = meta.split('name=')[1];
					data = data;
					let {name, pass} = {name: meta, data: data};
				}

			})

			res.end();
		}

	});

});

server.listen(8080);
```



总结：

通过对buffer进行slice和split操作，将文件和metadata保存到指定路径中



缺陷：

1. 等到所有数据到达后才开始处理