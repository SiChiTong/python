# python django微信公众平台

## 验证微信支付结果通知
由于微信发送的数据格式是xml的，所以为了处理方便，我将xml格式的数据转换成json格式的数据进行处理
在python中可以使用xmltodict这个库进行转换，
1. 转换后的结果是一个dict，应该是一个dict的子类
2. 也可以转换成标准的dict：
    先转换成python的dict， 再用json.dumps()转换成字符串，
    再用json.loads()转换成标准的json格式的数据，用法如下:

    xml_dict = xmltodict.parse(xmlstr, encoding='utf-8')
    dict_str = json.dumps(xml_dict)
    json_str = json.dumps(dict_str)

    def weixin_verify(params):
        """
        验证微信支付结果通知
        :param params:
        :return:
        """
        sign = params['sign']
        if sign:
            message = weixin_params_filter(params)
            my_sign = md5(message.encode()).upper()
            if sign == my_sign:
                return True
            else:
                return False
        else:
            return False

    def weixin_params_filter(params):
        """
        将微信非空参数按照参数名ASCII码从小到大排序（字典序）
        并拼接成url键值对的形式，并在最后添加key
        :param params:
        :param key:
        :return:
        """
        results = ''
        if params is None or len(params) <= 0:
            return results
        ks = sorted(params)
        for k in ks:
            v = params[k]
            k = smart_str(k, charset)
            if v and v != '' and k != 'sign':
                results += '%s=%s&' % (k, v)
        return results+'key='+key

测试代码:

    def weixin_main():
        xmlstr = """
        <xml>
           <appid><![CDATA[wx2421b1c4370ec43b]]></appid>
           <attach><![CDATA[支付测试]]></attach>
           <bank_type><![CDATA[CFT]]></bank_type>
           <fee_type><![CDATA[CNY]]></fee_type>
           <is_subscribe><![CDATA[Y]]></is_subscribe>
           <mch_id><![CDATA[10000100]]></mch_id>
           <nonce_str><![CDATA[5d2b6c2a8db53831f7eda20af46e531c]]></nonce_str>
           <openid><![CDATA[oUpF8uMEb4qRXf22hE3X68TekukE]]></openid>
           <out_trade_no><![CDATA[1409811653]]></out_trade_no>
           <result_code><![CDATA[SUCCESS]]></result_code>
           <return_code><![CDATA[SUCCESS]]></return_code>
           <sign><![CDATA[B552ED6B279343CB493C5DD0D78AB241]]></sign>
           <sub_mch_id><![CDATA[10000100]]></sub_mch_id>
           <time_end><![CDATA[20140903131540]]></time_end>
           <total_fee>1</total_fee>
           <trade_type><![CDATA[JSAPI]]></trade_type>
           <transaction_id><![CDATA[1004400740201409030005092168]]></transaction_id>
        </xml>
        """
        params = json.dumps(xmltodict.parse(xmlstr, encoding='utf-8'))
        params = json.loads(params)
        print(params)
        print(weixin_verify(params['xml']))

## 使用python代码模拟微信服务端发送xml格式的数据
只需要将xml格式的字符串进行utf-8编码后直接赋给requests的data即可, 代码如下:

    def weixin_server_test():
        xmlstr = """
        <xml>
           <appid><![CDATA[wx2421b1c4370ec43b]]></appid>
           <attach><![CDATA[支付测试]]></attach>
           <bank_type><![CDATA[CFT]]></bank_type>
           <fee_type><![CDATA[CNY]]></fee_type>
           <is_subscribe><![CDATA[Y]]></is_subscribe>
           <mch_id><![CDATA[10000100]]></mch_id>
           <nonce_str><![CDATA[5d2b6c2a8db53831f7eda20af46e531c]]></nonce_str>
           <openid><![CDATA[oUpF8uMEb4qRXf22hE3X68TekukE]]></openid>
           <out_trade_no><![CDATA[14374760130773875]]></out_trade_no>
           <result_code><![CDATA[SUCCESS]]></result_code>
           <return_code><![CDATA[SUCCESS]]></return_code>
           <sign><![CDATA[B3C623BB9DF7B08E8F580CC6A3109A6A]]></sign>
           <sub_mch_id><![CDATA[10000100]]></sub_mch_id>
           <time_end><![CDATA[20140903131540]]></time_end>
           <total_fee>100</total_fee>
           <trade_type><![CDATA[JSAPI]]></trade_type>
           <transaction_id><![CDATA[1004400740201409030005092168]]></transaction_id>
        </xml>
        """
        resp = requests.post(url, data=xmlstr.encode())
        print(resp.text)

## 接收微信服务器发送的xml消息
微信服务器发送的xml消息，在django中可以通过request.body来获取，好像是通过post来发送的，不过又和普通的文件传输不太一样

示例如下:

    params = smart_str(request.body)
其中smart_str这个方法是django里面一个处理请求的关于编码转换的方法，你也可以自己写