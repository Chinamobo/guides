Chinamobo HTTP API 设计参考
=======

时间格式
----
时间需要统一使用 [ISO 8601](//en.wikipedia.org/wiki/ISO_8601) UTC 格式。

### 实现参考

Android 可以使用 DateFormat 类进行时间的转换，DateFormat 的设置可参考下列代码：
```Java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm'Z'");
sdf.setTimeZone(TimeZone.getTimeZone("UTC"));
```

iOS 可以使用 NSDateFormatter 进行时间的转换，其设置可参考下列代码：
```Objective-C
NSDateFormatter *standDateFormatter = [NSDateFormatter new];
standDateFormatter.dateFormat = @"yyyy'-'MM'-'dd'T'HH':'mm':'ss'Z'";
standDateFormatter.timeZone = [NSTimeZone timeZoneForSecondsFromGMT:0];
```

PHP 的处理方式可以参考下列实现：
```PHP
// 假设当前时间是北京时间 2014-05-27 09:54:47，PHP 默认时区是 +8
// 则 UTC 时间是 2014-05-27 01:54:47

// 简单的转换方法
strtotime("2014-05-27T01:54:47Z");    // int(1401155687)
gmdate("Y-m-d\TH:i:s\Z", time());     // string(20) "2014-05-27T01:54:47Z"

// 不指明时区则视为当前时区时间，结果为 1401126867，比上面的 1401155687 早 8 个小时
strtotime("2014-05-27 01:54:27");

// 使用 DateTime 类
// 将 string 转成时间
$dateTime = DateTime::createFromFormat(DateTime::ISO8601, "2014-05-27T01:54:47Z");

// 将时间转为 string
// 下面是个参考，还有其他方法
$dateTime = new DateTime("", new DateTimezone("UTC"));
$dateTime->setTimestamp(1401155687);
var_dump($dateTime->format('Y-m-d\TH:i:s\Z'));    // 输出：string(20) "2014-05-27T01:54:47Z"
```

JavaScript 中的实现：
```JavaScript
// 字符串转日期
Date("2014-05-27T01:54:47Z");

// 反转参见：http://stackoverflow.com/q/2573521
```

### 动机
 
对于时间，一切不考虑时区的实现都是错误的（[A joke](http://xkcd.com/1179/)）。在多种标准时间格式中，ISO 8601 是广泛使用格式之一。规范中，像 `1994-11-05T08:15:30-05:00`，`1994-11-05T08:15:30-0500`，`1994-11-05T13:15:30Z` 这集中时区的格式都是正确的，为了简化时区的处理，我们约定统一用 UTC 时间，即一个 Z 结尾的格式。


考虑使用 UUID 或类似物作为标识
-----
我们经常会在数据库中使用自增的整形作为资源的唯一标识，并在外部使用它来访问资源，比如：用 `/user/123` 访问 123 号用户的信息；用 `/news?id=123` 访问第 123 号新闻。多处情况，这没有什么问题，但是当资源是比较敏感或有价值时这么做可能就有问题了。

举例来说，真事，某手机运营商某系统，会给每个用户自动分配一个账号，账号的默认密码是 123456，登录名是手机号，这个设计有什么问题吗？首先，这个系统大部分人都不会用，上去改密码的更是寥寥无几，而手机号又是很容易遍历的。那么用手机号+默认密码遍历一下能成功登入系统的几率会非常高，登进去能拿到什么呢？也许拿不到什么，因为没人用么（但真能拿到什么真是非常严重的漏洞），但至少遍历一遍至少能拿到一个真实、可用的号码库。

再举一个，你见过任何一个主流网盘服务可以用文件的序号访问文件吗？没有的，大都是一个随机字符串，或者是至少两组数字，都是没法轻易遍历的。为什么？文件是一个网盘服务最重要的资源，随随便便就让人遍历个遍跟拖库也没什么区别了。

不容易在外部遍历资源，这是使用 UUID 的一个好处，另一个理论上的好处是可以分布式的创建 ID，而不需要一个需要随时在线的中心。另见：[Advantages and disadvantages of GUID / UUID database keys](http://stackoverflow.com/q/45399)

但如果能提供友好的 URI 总比使用 ID 要好，比如 `/user/alice` 总比 `/user/123` 或者 `/user/0fde5a1c` 要好。

### UDID 实现参考

Android 可使用 [UUID](https://developer.android.com/reference/java/util/UUID.html) 类，iOS 可使用 NSUUID。

PHP 实现待考，Windows 上推荐用 com_create_guid()，其他实现目前见到最佳的是 http://stackoverflow.com/a/15875555/945906。其他 PHP Manual 评论中的实现和 PECL uuid extension 都不推荐。

JavaScript 实现待考，目前见到最佳的实现是 http://stackoverflow.com/a/8809472/945906


### 不利面

个人比较赞同 The Old New Thing [GUIDs are globally unique, but substrings of GUIDs aren't](http://blogs.msdn.com/b/oldnewthing/archive/2008/06/27/8659071.aspx) 这篇文章中的观点，UUID 应该与设备有关的。网上能找到的多数 UUID 实现都是 RFC 4122 v4 （随机数）的实现，多数是很有问题的，一个随机数的范围有多大？拿 PHP 来说，如果用 mt_rand() 生成随机数，它能生成的范围有多大？我用 mt_getrandmax() 获得的结果是 2,147,483,647，跟理论上的 2^122 （约 5.317e36）差的不是一点半点吧。用时间戳，能精确到毫秒就不错了吧，乘以1000 换算成整形，2000 年的第一天直到 2100 年的第一天可用的整形有多少？946656000000 到 4102416000000，约 3.16e12，够用一百年吗，也许。只用时间戳的另一个问题是可以在多个设备上获得相同的 ID 是很容易的吧。


参考
-----
* https://beta.wikiversity.org/wiki/School:软件开发/API_Design
* [ISO 8601 : 1988 (E) - W3C](http://www.w3.org/TR/NOTE-datetime)
* [RFC2616: HTTP/1.1 - W3C](http://www.w3.org/Protocols/rfc2616/rfc2616.html)
* [RFC4122: A Universally Unique IDentifier (UUID) URN Namespace](http://tools.ietf.org/html/rfc4122)