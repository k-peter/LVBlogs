#### 目录介绍

- 1.定义
- 2.工作原理
- 3.Https建立过程
- 4.HTTPS 连接建立的过程 简要
- 5.抓包软件可以抓到加密信息的原理
### 1.定义
- HTTP over SSL 的简称，即工作在 SSL(安全套接字层) 或TLS（SSL的升级版）上的 HTTP。说白了就是加密通信的HTTP。

- 一定注意**HTTPS不是协议**。



### 2.工作原理 
- **在客户端和服务器之间协商出一套对称密钥，每次发送信息之前将内容加密，收到之后解密，达到内容的加密传输。**

加密解密数据用的对称加密,传输Pre master sercert和证书验证使用的非对称加密。

- **为什么不直接使用非对称加密？** 

非对称加密由于使用了复杂的数学原理，因此计算相当复杂，如果完全使用非对称加密来加密通信内容，会严重影响网络通信的性能。



### 3.Https建立过程

![](https://user-gold-cdn.xitu.io/2019/3/26/169b9f5a42ce6ab0?w=1423&h=663&f=png&s=94373)

#### Step 1. Client Hello C端通知S端建立连接 同时会带上客户端支持的:

	1. 对称加密算法
	2. 非对称加密算法
	3. Hash算法
	4. SSL或者TLS版本号
	5. 一个随机数

#### Step 2.Server Hello S端通知C端建立连接 并带上Server端选择的:

	1. 对称加密算法
	2. 非对称加密算法
	3. Hash算法
	4. SSL或者TLS版本号
	5. 一个随机数

那么此时客户端和服务端就拥有了相同的

	1. 对称加密算法
	2. 非对称加密算法
	3. Hash算法
	4. SSL或者TLS版本号
	5. 两个随机数

#### Step 3.S端发给客户端被第三方机构信任的一个证书公钥，客户端收到证书公钥后对证书进行验证。
看图:

![](https://user-gold-cdn.xitu.io/2019/3/26/169b9f93230fa1a5?w=330&h=280&f=png&s=30279)


![](https://user-gold-cdn.xitu.io/2019/3/26/169b9f95c32f7fa5?w=632&h=826&f=png&s=44579)
如图发来的信息主体是证书的公钥,但是它同时会附加上证书的签名和HOST主机名称。

	1. 首先我们通过发来证书的根证书在本地找寻相同的根证书（本地根证书一般是操作系统自身已经带着的）。
	2. 取出根证书的公钥对证书的签名(用证书私钥对发来公钥进行加密的一个Hash值)做认证（公钥解密）。
	3. 比对验证后的数据(Hash)和发来证书公钥的Hash值是否相同,如果相同则认证通过,说明是一个受信任的证书。

#### Step 4.生成Pre Master-sercert（一个随机数）,并用发来的证书公钥加密传给Server端。
#### Step 5.C端和S端用Pre Master-sercert 和1,2部的两个随机数生成Master-sercert。
#### Step 6.C端和S端用相同的Master-sercert和相同的对称加密算法生成秘钥和sercert key（用于做Hash摘要）,这时因为生成对称加密密钥的条件两方相同，所以最后生成的秘钥虽然没有通过传输，两方也是相同的(避免了对称加密过程中的秘钥泄露问题)。
#### Step 7.开始加密数据 C端加密数据发给S端同时会带上用sercert key为介质做的Hash值(为了防止数据被篡改)。
#### Step 8.S端解密数据后用sercert key对数据做Hash并同发送来的Hash作对比,如果相同则通过验证。
### 4.HTTPS 连接建立的过程 简要
	1. Client Hello 
	2. Server Hello 
	3. 服务器证书 信任建立
	4. Pre-master Secret 
	5. 客户端通知：将使用加密通信 
	6. 客户端发送：Finished 
	7. 服务器通知：将使用加密通信 
	8. 服务器发送：Finished

### 5.抓包软件可以抓到加密信息的原理Fiddler为什么可以抓取到信息？
**因为它要求你手动信任了它的根证书**。此时它就可以篡改证书为它自己的证书了,HOST NAME直接篡改为目标服务器不变,**`证书公钥是它自身的假机构签发的而你本地又信任了它的根证书,那么此时证书的验证是可以通过的,后续的Pre Master-Sercert它也可以获取到了,那么最后它是持有密钥的`**。Fiddler相当于当了中间人，客户端服务端两头骗，所以我们平时不要信任未知的证书。

可参考:https://blog.csdn.net/hbdatouerzi/article/details/71440206


![](https://user-gold-cdn.xitu.io/2019/3/26/169b9fb363c37e78?w=870&h=248&f=png&s=15901)



