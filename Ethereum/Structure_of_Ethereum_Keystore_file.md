# 详解以太坊 Keystore 文件

## 文件结构

下面是以太坊 `1.7.3` 稳定版客户端 Geth 在私有链上生成的 Keystore 文件内容（与公链文件结构相同）：

``` JSON
{
	"address": "74aadcda1c35fb5a4461eba5cbe2ffaff37a044c",
	"crypto": {
		"cipher": "aes-128-ctr",
		"ciphertext": "9e32071c06f766798ba5617808397e7f6e6f8a3b463dd126eef5e850a7d06bd3",
		"cipherparams": {
			"iv": "23f0e5e9b559805169ccfa23368f530a"
		},
		"kdf": "scrypt",
		"kdfparams": {
			"dklen": 32,
			"n": 262144,
			"p": 1,
			"r": 8,
			"salt": "0476c69012ee4a84af322fe84af9deff931a4ed1d1b275026224112829f49a08"
		},
		"mac": "b2b999956c831f81484e79554427e6d2b5351af1f63435863588002c9920e13e"
	},
	"id": "6f9b3263-448c-48de-b1ab-951255eaba45",
	"version": 3
}
```

一个包涵了许多参数的 JSON 文件，分为一下四部分：

- 地址（address）；
- 加密信息（crypto）；
- id；
- 版本（version）；

## 加密信息（crypto）

Keystore文件中的关键信息都存储在该部分，解析来将对各个Key进行详细介绍。

### cipher

一种对称加密的AES算法，这里用的是 aes-128-ctr 加密模式

### ciphertext
通过 cipher 加密算法对密钥加密后得到的文本，即密文。

### cipherparams
cipher 算法（aes-128-ctr）加密时所需的参数，这里只有一个参数：iv

### kdf
密钥生成函数，这里使用的是`scrypt`算法。

### kdfparams
scrypt算法需要用到的参数，简单来说就是 dklen、n、p、r、salt都是`kdf`函数需要的参数。有关scrypt函数的详细介绍请参见[这里](https://tools.ietf.org/html/rfc7914)

### mac
用于对我们设置的密码进行校验。详细介绍请参见[这里](https://github.com/hashcat/hashcat/issues/1228)