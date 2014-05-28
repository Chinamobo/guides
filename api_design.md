Chinamobo HTTP API 设计参考
=======

时间格式
----
时间需要统一使用 [ISO 8601](//en.wikipedia.org/wiki/ISO_8601) UTC 格式。

### 参考代码

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


参考
-----
* https://beta.wikiversity.org/wiki/School:软件开发/API_Design
* [ISO 8601 : 1988 (E) - W3C](http://www.w3.org/TR/NOTE-datetime)