![](https://xzfile.aliyuncs.com/media/upload/picture/20230623063842-8a65274a-114d-1.png) 讲这个思路之前我要先说一下SQL注入的原理

 				前端黑客发现输入点(username,密码,cookie,token,agent，HTTP头信息等等)，输入字符串（ASCII码流）后端接收字符串（ASCII码流），然后和自己代码里的select等字符串（在传给MySQL时也是ASCII码流）进行拼接，执行SQL语句，MySQL执行语句的时候无法区分哪些是程序员写的，哪些是黑客写的，所以这是问题根源
 	
 				还有一点，SQL数据库会自动解析0x开头的比如（0x2CA335434长度不限直到非0-F的字节）对应的ASCII码流解析成字符串，次操作只进行一次，就算解析后的字符串长成这个样子 (admin' 和 1=2 #),SQL 引擎也不会把他当成控制字符了，就是单纯的字符串，用汇编的说法就是这个数据在DS段，不在CS段了

因此sqlwaf的代码实现如下：



SQL-WAF

```php
<?php
比如
    $a='sad'时;
var_dump(bin2hex($a));	//	结果是 string(6) "736164"
因此
当 $a=$_GET/POST['uername或密码或cookie或token等']时,
function sqlwaf($xx){
    $xx = bin2hex($xx);
    return '0x'.$xx;
}
$a = sqlwaf($a);

可以之间select查询

function sqlwaf($xxxx){

    $a = bin2hex($xxxx);
    return '0x'.$a;

}


function create_connection() {
    $conn = mysqli_connect('localhost', 'root', '', 'learn') or die("数据库连接不成功.");
    mysqli_query($conn, "set names utf8"); //等同于mysqli_set_charset($conn, "utf8");
    return $conn;
}


function query($sql) {
            $conn = create_connection();
            $result = mysqli_query($conn, $sql);
            $rows = mysqli_fetch_all($result, MYSQLI_ASSOC);
            return $rows;
        }

// WAF 的$sql语句
$a = sqlwaf($_GET['xxx']);
$sql ='select * from  user where username='.$a;
var_dump(query($sql));

不加防范的$sql语句
$a = $_GET['xxx'];
$sql ="select * from user where username=''$a'";
var_dump(query($sql));


?>
```

<h1 style="color:yellow;text-align:center">普通版</h1>
![](https://xzfile.aliyuncs.com/media/upload/picture/20230623063714-55c63042-114d-1.png)
![](https://xzfile.aliyuncs.com/media/upload/picture/20230623063757-6f8ea1e4-114d-1.png)


<h1 style="color:yellow;text-align:center">waf版本只有正确输入才可以，无法注入</h1>

![](https://xzfile.aliyuncs.com/media/upload/picture/20230623063830-8363a43a-114d-1.png)
