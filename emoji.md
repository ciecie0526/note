
前言

emoji介绍

Emoji （絵文字，词义来自日语えもじ，e-moji，moji在日语中的含义是字符）是一套起源于日本的12x12像素表情符号，由栗田穣崇(Shigetaka Kurit)创作，最早在日本网络及手机用户中流行，自苹果公司发布的iOS 5输入法中加入了emoji后，这种表情符号开始席卷全球，目前emoji已被大多数现代计算机系统所兼容的Unicode编码采纳，普遍应用于各种手机短信和社交网络中。近期，更是有不少网友用emoji图案玩猜字游戏，享受这种表情文化带来的乐趣。
关于emoji的发音：很多人第一眼见到emoji便会下意识将其误读作“一磨叽”，其实不然，emoji音译过来大概读作“诶磨叽”，当中“e”的发音颇似字母abc的a的发音。
最初日本的三大电信运营商各自有不同的字符定义，分别是DoCoMo、KDDI和Softbank。随着ios内置了Softbank的版本，emoji在全球范围内风靡（iOS5版本以前）。而Google又自己定义了一套emoji字符。iOS5以后，apple采用了unicode定义的emoji字符（iOS5版本以后）。
unicode定义的emoji是四个字符，softbank为3个字符，emoji的四个字符从存储到展示对应没有做过考虑的系统来说，简直就是灾难。

面临问题：

插入Emoji表情，保存到数据库时报错：
SQLException: Incorrect string value: '\xF0\x9F\x98\x84' for column 'review' at row 1
UTF-8编码有可能是两个、三个、四个字节。Emoji表情是4个字节，而MySQL的utf8编码最多3个字节，所以数据插不进去。

emoji表情的存放对于日常的开发还是比较经常遇到的。不管是留言还是昵称多多少少都会用到emoji。 
有没有发现emoji是没办法直接放到数据库中？ 
那么该如何以正确的姿势来存放和使用emoji呢？ 

因为emoji符号实际上是文本，并不是图片，它们仅仅显示为图片而已。而且，emoji符号已经被标准化并编码到最新的Unicode标准中了，所以，要支持emoji，只需要底层软件系统支持就可以了。
服务器端要正确存储emoji符号，只需要确保Web程序和底层数据库能支持最新的Unicode标准就可以了

1.数据库层面出发  

2.转译层面出发 





正文

# 数据库层面#
为什么我们设置表的的字符类型为utf8却不能存放emoji呢？

原来utf8可能是2或3或4个字节，而mysql的utf8是3个字节，存放一个emoji是需要4个字节的，自然不够。
解决办法： 
将Mysql的编码从utf8转换成utf8mb4

1. 修改my.ini [mysqld] character-set-server=utf8mb4
2. 在Connector/J的连接参数中，不要加characterEncoding参数。 不加这个参数时，默认值就时autodetect。
3. 将已经建好的表也转换成utf8mb4 
   
命令：ALTER TABLE `TABLE_NAME` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci; （将TABLE_NAME替换成你的表名）

4. 将需要使用emoji的字段设置类型为： 
   
命令：ALTER TABLE `TABLE_NAME`MODIFY COLUMN `COLUMN_NAME`  text CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

**如果使用TP框架开发则将database.php中charset也改为utf8mb4**

# 转义层面 #

如果嫌上述方案麻烦，还要操作数据库最初的类型。那么这种方法适合你。

原理:转义成字符串放入到数据库，使用的时候反转义可以直接转义成表情，再把内容传进去就。
//对emoji表情转义

    function emoji_encode($str){
    $strEncode = '';   
    $length = mb_strlen($str,'utf-8');  
    for ($i=0; $i < $length; $i++) {
    $_tmpStr = mb_substr($str,$i,1,'utf-8');
    if(strlen($_tmpStr) >= 4){
    $strEncode .= '[[EMOJI:'.rawurlencode($_tmpStr).']]';
    }else{
    $strEncode .= $_tmpStr;
    }
    }   
    return $strEncode;
    }
//对emoji表情转反义
    
    function emoji_decode($str){
    
    $strDecode = preg_replace_callback('|\[\[EMOJI:(.*?)\]\]|', function($matches){  
       
     return rawurldecode($matches[1]);
    }, $str);
    
    return $strDecode;
    }







