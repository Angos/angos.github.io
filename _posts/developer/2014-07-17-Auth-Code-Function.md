---
layout: post
title: Auth Code Function
date: 13:14 2014/7/17
categories:
 - developer
tags:
 - php
description: 一个对称加密的简单方法。
---

<pre class="prettyPrint">
function authCode($string, $auth_key, $operation='ENCODE') {  
    $key = md5($auth_key);  
    $key_length = strlen($key);  
  
    //base64_decode 默认是 iso-8859-1 转 UTF-8 用 utf8_decode(base64_decode($string))  
    $string = $operation == 'DECODE' ? utf8_decode(base64_decode($string)) : substr(md5($string.$key), 0, 8).$string;  
    $string_length = strlen($string);  
  
    $rndkey = $box = array();  
    $result = '';  
  
    for($i = 0; $i <= 255; $i++) {  
        $rndkey[$i] = ord($key[$i % $key_length]);  
        $box[$i] = $i;  
    }   
   
    for($j = $i = 0; $i < 256; $i++) {  
        $j = ($j + $box[$i] + $rndkey[$i]) % 256;  
        $tmp = $box[$i];  
        $box[$i] = $box[$j];  
        $box[$j] = $tmp;  
    }  
  
    for($a = $j = $i = 0; $i < $string_length; $i++) {  
        $a = ($a + 1) % 256;  
        $j = ($j + $box[$a]) % 256;  
        $tmp = $box[$a];  
        $box[$a] = $box[$j];  
        $box[$j] = $tmp;  
        $result .= chr(ord($string[$i]) ^ ($box[($box[$a] + $box[$j]) % 256]));  
    }  
  
    if($operation == 'DECODE') {  
        if(substr($result, 0, 8) == substr(md5(substr($result, 8).$key), 0, 8)) {  
            return substr($result, 8);  
        } else {  
            return '';  
        }  
    } else {  
        //base64_encode 默认是 iso-8859-1 转 UTF-8 用 base64_encode(utf8_encode($result))  
        return str_replace('=', '',base64_encode(utf8_encode($result)));  
    }  
}  
</pre>
