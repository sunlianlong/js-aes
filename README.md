# 基于cryptoJS的aes加密

其实就是使用cryptoJS中的方法写的；


## 说明 ：与java进行加密通信，后端使用的是AES的PKCS5Padding填充方式;但是前端在使用时需要使用PKCS7Padding，这两个是等价的
## 步骤 ：
发送服务端的数据

1.	由于Java就是按照128bit给的，但是由于是一个字符串，需要先在前端将其转为128bit(CryptoJS.enc.Utf8.parse);

2.	进行数据偏移 plaintText为第一步执行结果，key为密钥字符串CryptoJS.enc.Latin1.parse()生成
CryptoJS.AES.encrypt(plaintText, key, { 
   	mode: CryptoJS.mode.ECB,
   	padding: CryptoJS.pad.Pkcs7
 })

3.	将上面的对象encryptedData.toString()为字符串

接收服务端的数据

1.	拿到字符串类型的密文需要先将其用Hex方法parse一下CryptoJS.enc.Hex.parse();

2.	使用CryptoJS.AES.decrypt方法解密
		CryptoJS.AES.decrypt(encryptedBase64Str, key, { 
			mode: CryptoJS.mode.ECB,
			padding: CryptoJS.pad.Pkcs7
		});

3.	经过CryptoJS解密后，依然是一个对象，将其变成明文就需要按照Utf8格式转为字符串
	decryptedData.toString(CryptoJS.enc.Utf8); 





#### 下面方法将加密、解密、加base64、解base64方法声明。并在CryptoJS上扩展两个方法，CryptoJS.aeson(需要加密字符串)---向服务端发送的加密字符串；CryptoJS.aesoff(需要解密的字符串)---前端拿到服务端的字符串解密用
```
;(function(CryptoJS){
  var b = "0123456789abcdef",
  c = function(a, b) {
      var c = CryptoJS.enc.Utf8.parse(a),
          d = CryptoJS.enc.Latin1.parse(b),
          e = CryptoJS.AES.encrypt(c, d, {
              mode: CryptoJS.mode.ECB,
              padding: CryptoJS.pad.Pkcs7
          }),
          f = e.ciphertext.toString();
      return f
  },
  d = function(a, b) {
      var c = CryptoJS.enc.Base64.stringify(CryptoJS.enc.Hex.parse(a)),
          d = CryptoJS.enc.Latin1.parse(b),
          e = CryptoJS.AES.decrypt(c, d, {
              mode: CryptoJS.mode.ECB,
              padding: CryptoJS.pad.Pkcs7
          });
      return e.toString(CryptoJS.enc.Utf8)
  },
  e = function(a) {
      return a = CryptoJS.enc.Utf8.parse(a), CryptoJS.enc.Base64.stringify(a)
  },
  f = function(a) {
      return CryptoJS.enc.Base64.parse(a).toString(CryptoJS.enc.Utf8)
  };
  CryptoJS.aeson = function(a){
    return c(e(a), b)
  };
  CryptoJS.aesoff = function(a){
    return f(d(a, b))
  };
})(CryptoJS)
```

#使用方法：
1.在页面引入 aesCrypto.js

2. 使用CryptoJS.aeson()和CryptoJS.aesoff()




## 前端开发环境与生产环境
一、开发环境
请求函数，默认使用jquery的1.6以上版本
```
function GetUserInfo(data,add) {
	var url = "nginx上配置的地址";
	return $.ajax({
	        type: "get",
	        url: url+add,
	        dataType: "json",
	        contentType: "application/json;utf-8",
	        data: data,
	        timeout: 6000
	    });
};
```
调用函数，传入参数、请求url并使用.done().fail()写回调函数(promise对象)
```
GetUserInfo("参数字符串","地址")
.done(function (response) {
    console.log(response);
})
.fail(function (res) {
	console.log(res.statusText);
    //TODO
});
```


二、生产环境
请求函数，默认使用jquery的1.6以上版本
```
function GetUserInfo(data,add) {
	var url = "nginx上配置的地址";
	return $.ajax({
	        type: "get",
	        url: url+add,
	        dataType: "text",
	        contentType: "application/json;utf-8",
	        data: CryptoJS.aeson(data),
	        timeout: 6000,
	        dataFilter:function(res,type){
	          return JSON.parse(CryptoJS.aesoff(res));
	        }
	    });
};
```
调用函数，传入参数、请求url并使用.done().fail()写回调函数(promise对象)
```
GetUserInfo("参数字符串","地址")
.done(function (response) {
    console.log(response);
})
.fail(function (res) {
	console.log(res.statusText);
    //TODO
});
```
		