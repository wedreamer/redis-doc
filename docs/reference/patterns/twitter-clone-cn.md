
本文介绍[非常简单的推特克隆](https://github.com/antirez/retwis)使用 PHP 编写, Redis 是唯一的数据库。传统上, 编程社区将键值存储视为一种特殊用途的数据库, 不能用作用于开发Web应用程序的关系数据库的直接替代品。本文将尝试表明, 键值层之上的 Redis 数据结构是实现多种应用程序的有效数据模型。

注意：本文的原始版本写于2009年, 当时Redis是
释放。当时还不清楚Redis数据模型是
适合编写整个应用程序。现在5年后有很多案例
使用Redis作为其主商店的应用程序, 因此本文的目标
是Redis新手的教程。您将学习如何设计一个简单的
使用 Redis 的数据布局, 以及如何应用不同的数据结构。

我们的推特克隆, 称为[Retwis](https://github.com/antirez/retwis), 结构简单, 性能非常好, 可以毫不费力地分布在任意数量的Web和Redis服务器中。[查看 Retwis 源代码](https://github.com/antirez/retwis).

我使用PHP作为例子, 因为它可以被每个人阅读。使用Ruby, Python, Erlang等可以获得相同 (或更好) 的结果。
存在一些克隆 (但并非所有克隆都使用与
本教程的当前版本, 所以请坚持使用官方PHP
为了实现为了更好地遵循文章) 。

*   [Retwis-RB](https://github.com/danlucraft/retwis-rb)是Retwis到Ruby和Sinatra的移植, 由Daniel Lucraft编写。
*   [Retwis-J](https://docs.spring.io/spring-data/data-keyvalue/examples/retwisj/current/)是 Retwis 到 Java 的一个端口, 使用 Spring Data Framework, 由[科斯汀·劳](http://twitter.com/costinl).它的源代码可以在[GitHub](https://github.com/SpringSource/spring-data-keyvalue-examples), 并且有全面的文档, 请访问[springsource.org](http://j.mp/eo6z6I).

## 什么是键值存储？

键值存储的本质是能够存储一些数据, 称为*价值*, 位于键内。只有当我们知道该值的存储位置的特定键时, 才能在以后检索该值。没有直接的方法可以按值搜索键。从某种意义上说, 它就像一个非常大的哈希/字典, 但它是持久的, 即当你的应用程序结束时, 数据不会消失。因此, 例如, 我可以使用命令`SET`以存储值*酒吧*在键中*foo*:

    SET foo bar

Redis永久存储数据, 所以如果我以后问”*密钥 foo 中存储的值是什么？*“ Redis 将回复*酒吧*:

    GET foo => bar

键值存储提供的其他常见操作包括`DEL`, 删除给定键及其关联值, SET-if-不存在 (称为`SETNX`), , 仅当某个键尚不存在时才为该键赋, , 以及`INCR, , 以原子方式递增存储在给定键中的数字：

    SET foo 10
    INCR foo => 11
    INCR foo => 12
    INCR foo => 13

## 原子操作

有一些特别之处`INCR`.您可能想知道, 如果我们可以用一些代码自己完成, 为什么Redis提供这样的操作？毕竟, 它就像：

    x = GET foo
    x = x + 1
    SET foo x

问题是, 只要只有一个客户端使用密钥, 这种方式就会起作用。*foo*曾经。查看如果两个客户端同时访问此密钥会发生什么情况：

    x = GET foo (yields 10)
    y = GET foo (yields 10)
    x = x + 1 (x is now 11)
    y = y + 1 (y is now 11)
    SET foo x (foo is now 11)
    SET foo y (foo is now 11)

出了点问题！我们将该值递增了两倍, 但不是从 10 增加到 12, 而是将键保持为 11。这是因为增量完成`GET / increment / SET` *不是原子操作*.相反, Redis提供的INKR, Memcached, ..., 是原子实现, 服务器将在完成增量所需的时间内负责保护密钥, 以防止同时访问。

Redis 与其他键值存储的不同之处在于, 它提供了类似于 INCR 的其他操作, 可用于对复杂问题进行建模。这就是为什么你可以使用Redis来编写整个Web应用程序, 而无需使用另一个数据库 (如SQL数据库), , 也不会发疯。

## 超越键值存储：列表

在本节中, 我们将看到构建Twitter克隆所需的Redis功能。首先要知道的是, Redis 值可以不止字符串。Redis支持列表, 集, 哈希, 排序集, 位图和HyperLogLog类型作为值, 并且有原子操作可以对它们进行操作, 因此即使多次访问同一密钥, 我们也是安全的。让我们从列表开始：

    LPUSH mylist a (now mylist holds 'a')
    LPUSH mylist b (now mylist holds 'b','a')
    LPUSH mylist c (now mylist holds 'c','b','a')

`LPUSH`方法*左推*, 即, 将元素添加到 存储在 的列表的左侧 (或头部) *我的列表*.如果密钥*我的列表*不存, , 它会在 PUSH 操作之前自动创建为空列表。可以想, , 还有一个`RPUSH`将元素添加到列表右侧 (在尾部) 的操作。这对我们的Twitter克隆非常有用。可以将用户更新添加到存储在`username:updates`例如。

当然, 有从列表获取数据的操作。例如, LRANGE 从列表中返回一个范围, 或整个列表。

    LRANGE mylist 0 1 => c,b

LRANGE使用从零开始的索引 - 即第一个元素为0, 第二个元素为1, 依此类推。命令参数为`LRANGE key first-index last-index`.这*上一个索引*参数可以是负数, 具有特殊含义：-1 是列表的最后一个元素, -2 是倒数第二个元素, 依此类推。因此, 要获取整个列表, 请使用：

    LRANGE mylist 0 -1 => c,b,a

其他重要操作是返回列表中元素数量的LLEN, 以及类似于LRANGE但不返回指定范围的LTRIM*装饰*列表, 所以它就像*从 mylist 获取范围, 将此范围设置为新值*但这样做是原子的。

## “集”数据类型

目前, 我们在本教程中不使用 Set 类型, 但由于我们使用
排序集, 这是一种功能更强大的集版本, 它更好
首先开始引入集合 (这是一个非常有用的数据结构) 
本身), , 以及后来的排序集。

数据类型比列表更多。Redis 还支持 Sets, 它们是元素的未排序集合。可以添加、删除和测试成员是否存在, 并执行不同集之间的交集。当然, 可以得到一个集合的元素。一些例子会更清楚地说明这一点。请记住, `SADD`是*添加到集合*操作`SREM`是*从集中删除*操作`SISMEMBER`是*测试成员*操作, 以及`SINTER`是*执行交叉*操作。其他操作包括`SCARD`获取集合的基数 (元素数), , 以及`SMEMBERS`以返回集合的所有成员。

    SADD myset a
    SADD myset b
    SADD myset foo
    SADD myset bar
    SCARD myset => 4
    SMEMBERS myset => bar,a,foo,b

请注意, `SMEMBERS`不以我们添加元素的相同顺序返回元素, 因为 Set 是*排序*元素的集合。当您想要按顺序存储时, 最好改用列表。针对集的更多操作：

    SADD mynewset b
    SADD mynewset foo
    SADD mynewset hello
    SINTER myset mynewset => foo,b

`SINTER`可以返回集合之间的交集, 但不限于两个集合。您可以要求 4、5 或 10000 组的交集。最后, 让我们检查一下如何`SISMEMBER`工程：

    SISMEMBER myset foo => 1
    SISMEMBER myset notamember => 0

## “已排序集”数据类型

排序集类似于集：元素的集合。但是, 在排序中
设置每个元素都与浮点值相关联, 称为
*元素分数*.由于分数的原因, 排序集中的元素是
有序, 因为我们总是可以按分数比较两个元素 (如果分数
碰巧是相同的, 我们将两个元素比较为字符串) 。

与排序集中的集合一样, 不可能添加重复的元素, 每个
元素是唯一的。但是, 可以更新元素的分数。

排序的 Set 命令以`Z`.下面是一个示例
排序集的用法：

    ZADD zset 10 a
    ZADD zset 5 b
    ZADD zset 12.55 c
    ZRANGE zset 0 -1 => b,a,c

在上面的示例中, 我们添加了一些元素`ZADD`, 并在以后检索
具有`ZRANGE`.如您所见, 元素按顺序返回
根据他们的分数。为了检查给定元素是否存在, 以及
如果存在, 也要检索其分数, 我们使用`ZSCORE`命令：

    ZSCORE zset a => 10
    ZSCORE zset non_existing_element => NULL

排序集是一种非常强大的数据结构, 您可以通过以下方式查询元素
分数范围, 字典, 反向顺序, 依此类推。
了解更多[请查看官方 Redis 命令文档中的排序集部分](https://redis.io/commands/#sorted_set).

## 哈希数据类型

这是我们在程序中使用的最后一个数据结构, 并且非常简单
喘口气, 因为几乎每种编程语言中都有一个等价物
那里：哈希。Redis Hashes基本上就像Ruby或Python哈希一样, 
与值关联的字段的集合：

    HMSET myuser name Salvatore surname Sanfilippo country Italy
    HGET myuser surname => Sanfilippo

`HMSET`可用于设置哈希中的字段, 可以使用
`HGET`后。可以检查字段是否存在`HEXISTS`或
以递增哈希字段`HINCRBY`等等。

哈希是要表示的理想数据结构*对象*.例如, 我们
使用哈希值来表示我们的 Twitter 克隆中的用户和更新。

好的, 我们刚刚公开了 Redis 主数据结构的基础知识, 
我们准备开始编码！

## 先决条件

如果您尚未下载[Retwis 源代码](https://github.com/antirez/retwis)已经请现在抓住它。它包含一些PHP文件, 以及[普雷迪斯](https://github.com/nrk/predis), 我们在此示例中使用的 PHP 客户端库。

您可能想要的另一件事是工作Redis服务器。只需获取源代码, 构建`make`, 运行方式`./redis-server`, 然后您就可以开始了。无需任何配置即可在计算机上玩或运行Retwis。

## 数据布局

使用关系数据库时, 必须设计一个数据库架构, 以便我们知道数据库将包含的表、索引等。我们在 Redis 中没有表, 那么我们需要设计什么呢？我们需要确定需要哪些键来表示我们的对象, 以及这些键需要保存什么样的值。

让我们从用户开始。当然, 我们需要用用户的用户名、userid、密码、用户组跟随给定用户的用户集、给定用户关注的用户集来表示用户, 等等。第一个问题是, 我们应该如何识别用户？与关系数据库一样, 一个好的解决方案是使用不同的数字识别不同的用户, 这样我们就可以将唯一的ID与每个用户相关联。对此用户的所有其他引用都将通过 id 完成。使用我们的原子创建唯一 ID 非常简单`INCR`操作。当我们创建一个新用户时, 我们可以做这样的事情, 假设该用户被称为“antirez”：

    INCR next_user_id => 1000
    HMSET user:1000 username antirez password p1pp0

*注意：为简单起见, 您应该在实际应用程序中使用散列密码
我们以明文形式存储密码。*

我们使用`next_user_id`键, 以便始终为每个新用户获取唯一 ID。然后, 我们使用此唯一 ID 来命名包含用户数据的哈希的密钥。*这是一种常见的设计模式*与键值存储！请记住这一点。
除了已经定义的字段之外, 我们还需要更多的东西来完全定义用户。例如, 有时能够从用户名中获取用户 ID 可能很有用, 因此每次添加用户时, 我们还会填充`users`key, 它是一个哈希, 用户名作为字段, 其 ID 作为值。

    HSET users antirez 1000

乍一看, 这可能看起来很奇怪, 但请记住, 我们只能直接访问数据, 而无需二级索引。无法告诉 Redis 返回包含特定值的密钥。这也是*我们的实力*.这种新范式迫使我们组织数据, 以便所有内容都可以通过以下方式访问。*主键*, 用关系数据库术语说话。

## 关注者、关注者和更新

在我们的系统中还有另一个核心需求。用户可能有关注他们的用户, 我们将他们称为关注者。用户可能会关注其他用户, 我们称之为关注者。为此, 我们有一个完美的数据结构。那是。。。集。
Set 元素的唯一性, 以及我们可以在恒定时间内测试
存在, 是两个有趣的特征。但是, 还要记住呢？
给定用户开始关注另一个用户的时间？在增强的
我们简单的Twitter克隆版本这可能很有用, 所以不要使用
一个简单的集合, 我们使用排序集, 使用以下或追随者的用户ID
用户作为元素, 以及用户之间关系的unix时间
被创建, 作为我们的分数。

因此, 让我们定义我们的键：

    followers:1000 => Sorted Set of uids of all the followers users
    following:1000 => Sorted Set of uids of all the following users

我们可以通过以下方式添加新的关注者：

    ZADD followers:1000 1401267618 1234 => Add user 1234 with time 1401267618

我们需要的另一件重要的事情是, 我们可以添加更新以显示在用户主页中的位置。稍后, 我们需要按时间顺序访问此数据, 从最近的更新到最旧的更新, 因此完美的数据结构是 List。基本上每个新的更新都会`LPUSH`在用户更新键中, 并感谢`LRANGE`, 我们可以实现分页等等。请注意, 我们使用这些词*更新*和*职位*可以互换, 因为更新实际上在某种程度上是“小帖子”。

    posts:1000 => a List of post ids - every new post is LPUSHed here.

此列表基本上是用户时间线。我们将推送她/他自己的ID
帖子, 以及由以下用户创建的所有帖子的 ID。
基本上, 我们将实现一个写入扇出。

## 认证

好吧, 除了身份验证之外, 我们或多或少都有关于用户的所有内容。我们将以一种简单但健壮的方式处理身份验证：我们不想使用PHP会话, 因为我们的系统必须准备好轻松地分布在不同的Web服务器中, 因此我们将整个状态保留在Redis数据库中。我们所需要的只是一个随机的**不可猜测**字符串设置为经过身份验证的用户的 Cookie, 以及将包含保存该字符串的客户端的用户 ID 的密钥。

我们需要两件事才能使这个东西以一种强大的方式工作。
第一：当前身份验证*秘密* (随机不可猜测的字符串) 
应该是User对象的一部分, 因此在创建用户时, 我们还设置
一`auth`字段在其哈希中：

    HSET user:1000 auth fea5e81ac8ca77622bed1c2132a021f9

此外, 我们需要一种方法将身份验证密钥映射到用户ID, 因此
我们还采取`auths`键, 其值为哈希类型映射
用户 ID 的身份验证机密。

    HSET auths fea5e81ac8ca77622bed1c2132a021f9 1000

为了对用户进行身份验证, 我们将执行这些简单的步骤 (请参阅`login.php`文件在 Retwis 源代码中) ：

*   通过登录表单获取用户名和密码。
*   检查`username`字段实际存在于`users`散 列。
*   如果它存在, 我们有用户ID (即1000) 。
*   检查 user：1000 密码是否匹配, 如果没有, 则返回错误消息。
*   好的, 已通过身份验证！设置“fea5e81ac8ca77622bed1c2132a021f9” (用户值：1000`auth`字段) 作为“身份验证”Cookie。

这是实际代码：

    include("retwis.php");

    # Form sanity checks
    if (!gt("username") || !gt("password"))
        goback("You need to enter both username and password to login.");

    # The form is ok, check if the username is available
    $username = gt("username");
    $password = gt("password");
    $r = redisLink();
    $userid = $r->hget("users",$username);
    if (!$userid)
        goback("Wrong username or password");
    $realpassword = $r->hget("user:$userid","password");
    if ($realpassword != $password)
        goback("Wrong username or password");

    # Username / password OK, set the cookie and redirect to index.php
    $authsecret = $r->hget("user:$userid","auth");
    setcookie("auth",$authsecret,time()+3600*24*365);
    header("Location: index.php");

每次用户登录时都会发生这种情况, 但我们还需要一个函数`isLoggedIn`以检查给定用户是否已通过身份验证。这些是`isLoggedIn`功能：

*   从用户处获取“身份验证”Cookie。当然, 如果没有cookie, 用户就没有登录。让我们调用 Cookie 的值`<authcookie>`.
*   检查是否`<authcookie>`字段中的字段`auths`哈希存在, 以及值 (用户 ID) 是什么 (示例中为 1000) 。
*   为了使系统更加可靠, 还要验证 user：1000 auth 字段是否也匹配。
*   好的, 用户已通过身份验证, 我们在`$User`全局变量。

代码比描述更简单, 可能：

    function isLoggedIn() {
        global $User, $_COOKIE;

        if (isset($User)) return true;

        if (isset($_COOKIE['auth'])) {
            $r = redisLink();
            $authcookie = $_COOKIE['auth'];
            if ($userid = $r->hget("auths",$authcookie)) {
                if ($r->hget("user:$userid","auth") != $authcookie) return false;
                loadUserInfo($userid);
                return true;
            }
        }
        return false;
    }

    function loadUserInfo($userid) {
        global $User;

        $r = redisLink();
        $User['id'] = $userid;
        $User['username'] = $r->hget("user:$userid","username");
        return true;
    }

有`loadUserInfo`作为一个单独的函数, 对于我们的应用程序来说是过度的, 但在复杂的应用程序中, 这是一个好方法。所有身份验证中唯一缺少的是注销。注销后我们该怎么办？这很简单, 我们只需更改user：1000中的随机字符串`auth`字段中, 从 中删除旧的身份验证密钥`auths`哈希, 然后添加新的。

*重要：*注销过程解释了为什么我们不只是在`auths`哈希, 但根据 user：1000 进行双重检查`auth`田。真正的身份验证字符串是后者, 而`auths`哈希只是一个甚至可能是易失性的身份验证字段, 或者, 如果程序中存在错误或脚本被中断, 我们甚至可能以`auths`指向同一用户 ID 的键。注销代码如下 (`logout.php`):

    include("retwis.php");

    if (!isLoggedIn()) {
        header("Location: index.php");
        exit;
    }

    $r = redisLink();
    $newauthsecret = getrand();
    $userid = $User['id'];
    $oldauthsecret = $r->hget("user:$userid","auth");

    $r->hset("user:$userid","auth",$newauthsecret);
    $r->hset("auths",$newauthsecret,$userid);
    $r->hdel("auths",$oldauthsecret);

    header("Location: index.php");

这正是我们所描述的, 应该很容易理解。

## 更新

更新 (也称为帖子) 甚至更简单。为了在数据库中创建新帖, , 我们执行如下操作：

    INCR next_post_id => 10343
    HMSET post:10343 user_id $owner_id time $time body "I'm having fun with Retwis"

如您所见, 每个帖子仅由具有三个字段的哈希表示。拥有帖子的用户的 ID、帖子的发布时间, 最后是帖子的正文, 即实际状态消息。

在我们创建帖子并获得帖子ID后, 我们需要在关注帖子作者的每个用户的时间轴中LPUSH该ID, 当然在作者本身的帖子列表中 (每个人实际上都在关注她自己) 。) 。这是文件`post.php`显示了如何执行此操作：

    include("retwis.php");

    if (!isLoggedIn() || !gt("status")) {
        header("Location:index.php");
        exit;
    }

    $r = redisLink();
    $postid = $r->incr("next_post_id");
    $status = str_replace("\n"," ",gt("status"));
    $r->hmset("post:$postid","user_id",$User['id'],"time",time(),"body",$status);
    $followers = $r->zrange("followers:".$User['id'],0,-1);
    $followers[] = $User['id']; /* Add the post to our own posts too */

    foreach($followers as $fid) {
        $r->lpush("posts:$fid",$postid);
    }
    # Push the post on the timeline, and trim the timeline to the
    # newest 1000 elements.
    $r->lpush("timeline",$postid);
    $r->ltrim("timeline",0,1000);

    header("Location: index.php");

该功能的核心是`foreach`圈。我们使用`ZRANGE`获取当前用户的所有关注者, 则循环将`LPUSH`在每个关注者时间轴列表中推送帖子。

请注意, 我们还为所有帖子维护了全局时间轴, 以便在Retwis主页中, 我们可以轻松显示每个人的更新。这只需要做一个`LPUSH`到`timeline`列表。让我们面对现实吧, 你是不是开始觉得必须使用时间顺序对添加的东西进行排序有点奇怪。`ORDER BY`使用 SQL？我认为如此。

在上面的代码中有一件有趣的事情要注意：我们使用了一个新的
命令调用`LTRIM`在我们执行之后`LPUSH`在全球运营
时间轴。这是为了将列表修剪为仅 1000 个元素。这
全局时间线实际上仅用于在
主页, 没有必要拥有所有帖子的完整历史记录。

基本上`LTRIM`+`LPUSH`是创建*封顶收藏*在雷迪斯。

## 分页更新

现在应该很清楚我们可以如何使用`LRANGE`为了获取帖子的范围, 并在屏幕上呈现这些帖子。代码很简单：

    function showPost($id) {
        $r = redisLink();
        $post = $r->hgetall("post:$id");
        if (empty($post)) return false;

        $userid = $post['user_id'];
        $username = $r->hget("user:$userid","username");
        $elapsed = strElapsed($post['time']);
        $userlink = "<a class=\"username\" href=\"profile.php?u=".urlencode($username)."\">".utf8entities($username)."</a>";

        echo('<div class="post">'.$userlink.' '.utf8entities($post['body'])."<br>");
        echo('<i>posted '.$elapsed.' ago via web</i></div>');
        return true;
    }

    function showUserPosts($userid,$start,$count) {
        $r = redisLink();
        $key = ($userid == -1) ? "timeline" : "posts:$userid";
        $posts = $r->lrange($key,$start,$start+$count);
        $c = 0;
        foreach($posts as $p) {
            if (showPost($p)) $c++;
            if ($c == $count) break;
        }
        return count($posts) == $count+1;
    }

`showPost`将简单地转换并打印HTML中的帖子, 而`showUserPosts`获取一系列帖子, 然后将其传递给`showPosts`.

*注意：`LRANGE`如果帖子列表开始非常有效, 则效率不高
big, 我们希望访问位于列表中间的元素, 因为Redis列表由链接列表支持。如果系统设计为
百万个项目的深度分页, 最好求助于排序集
相反。*

## 关注用户

这并不难, 但我们还没有检查我们如何创建追随者/追随者关系。如果用户 ID 1000  (antirez)  想要关注用户 ID 5000  (pippo,  , 我们需要创建关注者和关注者关系。我们只需要`ZADD`调用：

        ZADD following:1000 5000
        ZADD followers:5000 1000

一次又一次地注意相同的模式。从理论上讲, 对于关系数据库, 关注者和关注者的列表将包含在一个表中, 其字段如下：`following_id`和`follower_id`.您可以使用 SQL 查询提取每个用户的关注者或关注者。使用键值数据库, 事情有点不同, 因为我们需要将两者都设置为`1000 is following 5000`和`5000 is followed by 1000`关系。这是要付出的代价, 但另一方面, 访问数据更简单, 速度非常快。将这些东西作为单独的集合可以让我们做一些有趣的事情。例如, 使用`ZINTERSTORE`我们可以有交叉点`following`两个不同的用户, 因此我们可能会在我们的Twitter克隆中添加一个功能, 以便当您访问其他人的个人资料时, 它能够非常快速地告诉您, “您和Alice有34个共同的关注者”, 以及类似的事情。

您可以在`follow.php`文件。

## 使其水平可扩展

温柔的读者, 如果你读到这一点, 你已经是一个英雄了。谢谢。在谈论水平扩展之前, 值得检查单个服务器上的性能。Retwis is*极快*, 而不带任何类型的缓存。在速度非常慢且负载很大的服务器上, 具有100个并行客户端发出100000个请求的Apache基准测试测量了平均页面浏览量, 即5毫秒。这意味着您每天只需一个Linux盒子就可以为数百万用户提供服务, 而这个盒子是猴子屁股慢的......想象一下使用更新的硬件的结果。

但是, 您不能永远使用单个服务器, 您如何扩展键值
商店？

Retwis不执行任何多键操作, 因此使其可扩展
简单：您可以使用客户端分片, 或类似分片代理的东西
比如Twemproxy, 或者即将到来的Redis Cluster。

要了解有关这些主题的更多信息, 请阅读
[我们关于分片的文档](/topics/partitioning).但是, 这里的要点
要强调的是, 在键值存储中, 如果您谨慎设计, 数据集
在**许多独立的小键**.分发这些密钥
与使用相比, 到多个节点更直接, 更可预测
语义上更复杂的数据库系统。
