[PHP android ios相互兼容的AES加密算法](https://blog.csdn.net/moqidian/article/details/51006670)

[AES加密CBC模式兼容互通四种编程语言平台【PHP、Javascript、Java、C#】](https://my.oschina.net/Jacker/blog/86383)

[IOS参考这里](https://www.cnblogs.com/leotangcn/p/4248414.html)

[PHP 7以后](https://blog.csdn.net/u014231144/article/details/78774788)

[PHP7  NODE.JS aes-256-cbc](https://blog.csdn.net/luo200618/article/details/52878523)

[JAVA  NODE.JS aes-256-cbc](https://github.com/fukata/AES-256-CBC-Example)




```

<?php
class Security
{
    public static function encrypt($input, $key, $iv) {
        $localIV = $iv;
        $module = mcrypt_module_open(MCRYPT_RIJNDAEL_128, '', MCRYPT_MODE_CBC, $localIV);
        mcrypt_generic_init($module, $key, $localIV);
        $size = mcrypt_get_block_size(MCRYPT_RIJNDAEL_128, MCRYPT_MODE_CBC);
        $input = Security::pkcs5_pad($input, $size);
        $data = mcrypt_generic($module, $input);

        mcrypt_generic_deinit($module);
        mcrypt_module_close($module);
        $data = base64_encode($data);
        return $data;
    }

    private static function pkcs5_pad ($text, $blocksize) {
        $pad = $blocksize - (strlen($text) % $blocksize);
        return $text . str_repeat(chr($pad), $pad);
    }

    public static function decrypt($sStr, $key, $iv) {
        $localIV = $iv;
        $module = mcrypt_module_open(MCRYPT_RIJNDAEL_128, '', MCRYPT_MODE_CBC, $localIV);
        mcrypt_generic_init($module, $key, $localIV);
        $encryptedData = base64_decode($sStr);
        $encryptedData = mdecrypt_generic($module, $encryptedData);

        $dec_s = strlen($encryptedData);
        $padding = ord($encryptedData[$dec_s-1]);
        $decrypted = substr($encryptedData, 0, -$padding);

        mcrypt_generic_deinit($module);
        mcrypt_module_close($module);
        if(!$decrypted){
            throw new Exception("Decrypt Error,Please Check SecretKey");
        }
        return $decrypted;
    }
}

$key = 'abcdefghijklmnop'; // 16位（也可以不是16位，但当它大于16位时，7.1的openssl函数会截取前16位，有点坑）
$iv = '1234567890123456'; // 16位

echo '用来加密的串：------------------------';
echo $str = '中文数字123字母ABC符号!@#$%^&*()';

echo '<br />用7.1之前的mcrypt加密：------------------------';
echo $strEncode = Security::encrypt($str, $key, $iv);

echo '<br />用7.1之前的mcrypt解密：------------------------';
echo Security::decrypt($strEncode, $key, $iv);









/**
 * [AesSecurity aes加密，支持PHP7.1]
 */
class AesSecurity
{
    /**
     * [encrypt aes加密]
     * @param    [type]                   $input [要加密的数据]
     * @param    [type]                   $key   [加密key]
     * @return   [type]                          [加密后的数据]
     */
    public static function encrypt($input, $key)
    {
        $data = openssl_encrypt($input, 'AES-128-ECB', $key, OPENSSL_RAW_DATA);
        $data = base64_encode($data);
        return $data;
    }
    /**
     * [decrypt aes解密]
     * @param    [type]                   $sStr [要解密的数据]
     * @param    [type]                   $sKey [加密key]
     * @return   [type]                         [解密后的数据]
     */
    public static function decrypt($sStr, $sKey)
    {
        $decrypted = openssl_decrypt(base64_decode($sStr), 'AES-128-ECB', $sKey, OPENSSL_RAW_DATA);
        return $decrypted;
    }
}








7.1


echo '<br />用PHP 7.1的openssl加密：------------------------';
echo base64_encode(openssl_encrypt($str, 'aes-128-cbc', $key, true, $iv));

echo '<br />用PHP 7.1的openssl解密：------------------------';
echo openssl_decrypt(base64_decode($strEncode), 'aes-128-cbc', $key, true, $iv);


```








