# python中RSA模块

RSA模块是一个纯python编写的rsa模块，它支持加密解密，签名和验证，包括公钥和私钥的生成

## 1. 生成一对公钥和私钥
    >>> pub_key, priv_key = rsa.newkeys(512)

## 2. 使用私钥签名
    >>> sign = rsa.sign(message, priv_key, hash)
    message: 需要bytes类型，
    priv_key: 就是上一步生成的私钥，
    hash: 'MD5', 'SHA-1', 'SHA-256', 'SHA-384', 'SHA-512'.

## 3. 加密
    >>> crypto = rsa.encrypt('hello', pub_key)
    使用公钥加密

## 4. 解密
    >>> result = decrypt(crypto, priv_key)
    使用私钥解密

## 5. 签名验证
    >>> rsa.verify(message, signature, pub_key)
    message: 需要bytes类型的数据
    signature: 待验证签名(需要注意待验证的签名是否进行了base64编码，如果有进行编码，则需要先解码)
    pub_key: 公钥

验证如果通过，则返回Ture, 否则抛出 VerificationError 异常
所以验证过程需要捕捉这个异常

## 6. 根据本地的公钥文本生成公钥
如果本地的文本是base64编码的，则需要先进行解码
下面以支付宝的公钥为例：

    >>> der = base64.decodebytes(ali_public_key.encode(charset))
    >>> pub_key = key.PublicKey.load_pkcs1_openssl_der(der)
还可以使用pem格式文件的形式获取：

    >>> pub_key = key.PublicKey.load_pkcs1_openssl_pem(keyfile)
    keyfile: pem的文件，可以先读取，也可以将读取放在一起进行，
            这个格式文件中包括前后一段：
            -----BEGIN PUBLIC KEY-----
            -----END PUBLIC KEY-----

读取私钥的方式是类似的

## 7. python中支付宝异步回调验证的例子：
在这里需要特别注意的是：支付宝的签名(sign)是经过base64编码过的，所以在验证的时候需要先解码：

    HTTPS_VERIFY_URL = "https://mapi.alipay.com/gateway.do?service=notify_verify&"

    def params_filter(params):
        """
        对数组排序并除去数组中的空值和签名参数
        :param params: 支付宝回调的参数
        :return: 去掉空值与签名参数后的字符串
        """
        results = ''
        if params is None or len(params) <= 0:
            return results
        ks = sorted(params)
        for k in ks:
            v = params[k]
            k = smart_str(k, AlipayConf.input_charset)
            if v and v != '' and k.lower() not in ('sign', 'sign_type'):
                results += '%s=%s&' % (k, v)
        return results[:-1]


    def verify_sign(message, sign, ali_public_key, charset):
        """
        验证签名
        :param message: 待签名数据
        :param sign: 签名值
        :param ali_public_key: 支付宝公钥
        :param charset: 编码格式
        :return: 布尔值，签名验证通过则为True,否则为False
        """
        der = base64.decodebytes(ali_public_key.encode(charset))
        pub_key = key.PublicKey.load_pkcs1_openssl_der(der)
        try:
            flag = rsa.verify(message.encode(charset), base64.b64decode(sign), pub_key)
            print(flag)
        except VerificationError:
            flag = False
        return flag


    def verify(params):
        """
        验证消息是否是支付宝发出的合法消息
        :param params:
        :return:
        """
        # 初级验证--签名
        message = params_filter(params)
        sign = params.get('sign', None)
        if sign:
            if not verify_sign(message, sign, AlipayConf.ali_public_key, AlipayConf.input_charset):
                return False
        else:
            return False
        # 二级验证--查询支付宝服务器此条信息是否有效
        data = dict()
        data['partner'] = AlipayConf.partner
        data['notify_id'] = params.get('notify_id')

        verify_result = urlopen(HTTPS_VERIFY_URL, urlencode(data).encode()).read().decode()
        if verify_result.lower().strip() == 'true':
            return True
        return False

配置文件：
配置文件中需要注意的是: 商户的私钥和支付宝的公钥不可以有空格，也不要换行，如果觉得难看，可以使用pem格式的文件保存

    class AlipayConf:
        # ------------请在这里配置您的基本信息--------------

        # 合作身份者ID，以2088开头由16位纯数字组成的字符串
        partner = ""

        # 商户的私钥
        private_key = ""

        # 支付宝的公钥，无需修改该值
        ali_public_key = "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCnxj/9qwVfgoUh/y2W89L6BkRAFljhNhgPdyPuBV64bfQNN1PjbCzkIM6qRdKBoLPXmKKMiFYnkd6rAoprih3/PrQEB/VsW8OoM8fxn67UDYuyBTqA23MML9q1+ilIZwBC2AQ2UBVOrFXfFl75p6/B5KsiNG9zpgmLCUYuLkxpLQIDAQAB"

        # -----------请在这里配置您的基本信息--------------

        # 字符编码格式 目前支持 gbk 或 utf-8
        input_charset = "utf-8"

        # 签名方式 不需修改
        sign_type = "RSA"
        
## 8. rsa中方法的封装
下面的方法可以直接放在一个模块中，也可以作为一个class的方法

    def generate_rsa():
        """
        先生成一对密钥，然后保存.pem格式文件，当然也可以直接使用
        :return:
        """
        (p_key, s_key) = rsa.newkeys(1024)
    
        pub = p_key.save_pkcs1()
        p_key_file = open("washer_pub.pem", 'w+')
        p_key_file.write(pub.decode("utf-8"))
        p_key_file.close()
        print("公钥生成完成")
    
        pri = s_key.save_pkcs1()
        s_key_file = open('washer.pem', 'w+')
        s_key_file.write(pri.decode("utf-8"))
        s_key_file.close()
        print("私钥生成完成")


    def get_pk():
        """
        获取公钥
        :return:
        """
        with open(settings.BASE_DIR + '/util/rsa/washer_pub.pem') as p_key_file:
            p = p_key_file.read()
            p_key = rsa.PublicKey.load_pkcs1(p.encode("utf-8"))
            return p_key
    
    
    def get_sk():
        """
        获取私钥
        :return:
        """
        with open(settings.BASE_DIR + '/util/rsa/washer.pem') as s_key_file:
            p = s_key_file.read()
            s_key = rsa.PrivateKey.load_pkcs1(p.encode("utf-8"))
            return s_key
    
    
    def rsa_encrypt(message):
        """
        用公钥加密
        :param message:
        :return:
        """
        p_key = get_pk()
        crypto = rsa.encrypt(message.encode("utf-8"), p_key)
        # print("加密的内容", crypto)
        return crypto
    
    
    def rsa_subsection_encrypt(message, size):
        """
        分段加密
        :param message: 待加密字符串
        :param size: 字符串分割大小（生成密钥时候的模长，如1024/8）
                size = rsa_size(rsa)-11
        :return:
        """
        n = len(message) // size+1
        crypto = b''
        p_key = get_pk()
        for i in range(n):
            crypto += rsa.encrypt(message[i*size: (i+1)*size].encode("utf-8"), p_key)
        return crypto
    
    
    def rsa_decrypt(crypto):
        """
        再用私钥解密
        :param crypto:
        :return:
        """
        s_key = get_sk()
        message = rsa.decrypt(crypto, s_key)
        # print("解密的内容", message)
        return message
    
    
    def rsa_subsection_decrypt(crypto, size):
        """
        分段解密
        :param crypto: 待解密字符串
        :param size: 生成密钥时候的模长, 例如1024//8=128
                size = rsa_size(rsa)
        :return:
        """
        n = len(crypto) // size
        message = ""
        s_key = get_sk()
        for i in range(n):
            c = crypto[i*size: (i+1)*size]
            message += rsa.decrypt(c, s_key).decode()
        return message
    
    
    def rsa_sign(message):
        """
        用私钥签名认证
        :param message:
        :return:
        """
        s_key = get_sk()
        signature = rsa.sign(message, s_key, 'SHA-1')
        print("签名", signature)
        return signature
    
    
    def rsa_verify(signature):
        """
        用公钥验证签名是否正确
        :param signature:
        :return:
        """
        p_key = get_pk()
        ret = rsa.verify('hello'.encode("utf-8"), signature, p_key)
        print("验证签名是否正确", ret)
        return ret

测试代码如下

    if __name__ == "__main__":
        r = rsa_encrypt("这是我的RSA加密字符串啊")
        # 生成base64字符串
        r1 = base64.b64encode(r).decode("utf-8")
        print(r1)
        # base64解码
        r2 = base64.b64decode(r1.encode("utf-8"))
        print(r2)
        print(type(r2))
        # 私钥解密
        print(rsa_decrypt(r2).decode("utf-8"))
    
        doc = {
            'a': 1234,
            'b': 'asasasasaaas',
            'c': 'asas'
        }
        doc = json.dumps(doc)
        print(doc)
        # rsa_doc = rsa_encrypt(doc)
        # print(rsa_doc)
        # b64_doc = base64.b64encode(rsa_doc).decode()
        # print(b64_doc)
        rsa_doc = rsa_subsection_encrypt(doc, 117)
        print('rsa_doc', rsa_doc)
        b64_doc = base64.b64encode(rsa_doc).decode()
        print('b64_doc:', b64_doc)
        print(len(b64_doc))
        b64_decode = base64.b64decode(b64_doc.encode())
        print('b64_decode:', b64_decode)
        msg = rsa_subsection_decrypt(b64_decode, 128)
        print(msg)
        