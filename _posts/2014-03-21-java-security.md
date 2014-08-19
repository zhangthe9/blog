---
layout: post
title: java RSA
description: 
- java RSA
categories:
- rsa
tags:
- rsa
---
##生成&转换
####生成无密码明文私钥
openssl genrsa -out rsa_private_key.pem 1024
####生成有密码明文私钥
openssl genrsa -des3 -out rsa_private_key.key 1024
####从生成的私钥生成公钥
openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem
####从生成的私钥生成二进制文件
openssl   base64  -d  -in rsa_private_key.pem -out private.key

####msysgit中ssh-keygen生成的公钥转换
ssh-keygen -t rsa -C "admin@ddatsh.com"
生成id_rsa(私钥)和id_rsa.pub(公钥)
将id_rsa.pub转换成openssl的公钥
ssh-keygen -f id_rsa.pub -e > id_rsa.pem


<pre class="prettyprint">
public static PrivateKey initPrivateKey(String path, String pwd) {
	try {
		KeyFactory keyFactory = KeyFactory.getInstance("RSA");
		File file = new File(path);
		byte[] b = null;
		InputStream in = new FileInputStream(file);
		PKCS8Key pkcs8 = new PKCS8Key(in, pwd.toCharArray());
		b = pkcs8.getDecryptedBytes();
		PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(b);
		PrivateKey prikey = keyFactory.generatePrivate(keySpec);
		return prikey;
	} catch (Exception e) {
		e.printStackTrace();
	}
	return null;
}
</pre>
 