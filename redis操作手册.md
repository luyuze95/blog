# Redis命令手册

[TOC]

## 字符串
-----

**SET**

- SET key value [EX seconds] [PX milliseconds] [NX|XX]
- 将字符串值value关联到key。
- 如果key已经持有其他值，set就覆写旧值，无视类型。
- 当set命令对一个带有生存时间(TTL)的键进行设置之后，该键原有的TTL将被清除
- NX：只在键不存在时，才对键进行设置操作，等同于setnx。
- XX：只在键已经存在时，才对键进行设置操作。

-----

**SETNX**

- SETNX key value
- 只在键key不存在的情况下，将键key的值设置为value。
- 若键key已经存在，则setnx命令不做任何动作。
- setnx是set if not exist的简写。
- 命令在设置成功时返回1，设置失败时返回0。

-----

**SETEX** 

- SETEX key seconds value
- 将键key的值设置为value，并将键key的生存时间设置为seconds秒钟。
- 如果键key已经存在，那么setex命令将覆盖已有的值。
- 等同于`set key value`, `expire key seconds`两个命令的组合，不同的是setex是一个原子操作
- 命令在设置成功时返回OK，当seconds参数不合法时，命令将返回一个错误。

-----

**PSETEX**

- PSETEX key milliseconds value
- 这个命令和setex命令相似，但它以毫秒为单位设置key的过期时间。
- 命令在设置成功时返回OK。

-----

**GET**

- GET key
- 返回与键key相关联的字符串值。
- 如果key不存在，那么返回特殊值nil；否则返回key的值。
- 如果key的值并非字符串类型，那么返回一个错误，因为get命令只能用于字符串值。

-----

**GETSET**

- GETSET key value
- 将key的值设为value，并返回key在被设置之前的旧值。
- 返回给定key的旧值。
- 如果key没有旧值，那么返回nil。
- 当key存在但不是字符串类型时，命令返回一个错误。

-----

**STRLEN**

- STRLEN key
- 返回key储存的字符串值的长度。
- 当key不存在时，命令返回0。
- 当key储存的不是字符串值时，返回一个错误。

-----

**APPEND**

- APPEND key value
- 如果key已经存在并且它的值是一个字符串，append命令将把value追加到key现有值的末尾。
- 如果key不存在，append就简单地将key的值设为value，等同于`set key value`命令。
- 返回追加value之后，键key的值的长度。

-----

**SETRANGE**

- SETRANGE key offset value
- 从偏移量offset开始，用value参数覆写键key储存的字符串值。
- 不存在的键key当作空白字符串处理。
- setrange命令会确保字符串足够长以便将value设置到指定的偏移量上，如果键key原来储存的字符串长度比偏移量小，那么原字符和偏移量之间的空白将用零字节(zerobytes，"\x00")进行填充。
- redis字符串大小最多512M，所以用户能够使用的最大偏移量为2^29-1。

-----

**GETRANGE**

- GETRANGE key start end
- 返回键key储存的字符串值的指定部分，字符串的截取范围由start和end两个偏移量决定，包括start和end在内。
- 负数偏移量表示从字符串的末尾开始计数，-1表示最后一个字符，-2表示倒数第二个字符，以此类推。
- getrange通过保证子字符串的值域不超过实际字符串的值域来处理超出范围的值域请求。

-----

**INCR**

- INCR key
- 为键key储存的数字值加上一。
- 如果键key不存在，那么它的值会先被初始化为0，然后再执行INCR命令。
- 如果键key储存的值不能被解释为数字，那么INCR命令将返回一个错误。
- 本操作的值限制在64位有符号数字表示之内。
- INCR命令会返回键key在执行加一操作之后的值。

-----

**INCRBY**

- INCRBY key increment
- 为键key储存的数字值加上增量increment。
- 如果键key不存在，那么键key的值会先被初始化为0，然后再执行INCRBY命令。
- 如果键key储存的值不能被解释为数字，那么INCRBY命令将返回一个错误。
- 本操作的值限制在64位有符号数字表示之内。
- 返回加上增量increment之后，键key当前的值。

-----

**INCRBYFLOAT**

- INCRBYFLOAT key increment
- 为键key储存的值加上浮点数增量increment。
- 如果键key不存在，那么INCRBYFLOAT会先将键key的值设为0，然后再执行加法操作。
- 如果命令执行成功，那么键key的值会被更新为执行加法计算之后的新值，并且新值会以字符串的形式返回给调用者。
- 无论加法计算所得的浮点数的实际精度有多长，INCRBYFLOAT命令的计算结果最多只保留小数点的后十七位。
- 当以下任意一个条件发生时，命令返回一个错误：
    - 键key的值不是字符串类型。
    - 键key当前的值或者给定的增量increment不能被解释为双精度浮点数。

-----

**DECR**

- DECR key
- 为键key储存的数字值减去一。
- 如果键key不存在，那么键key的值会先被初始化为0，然后再执行DECR操作。
- 如果键key储存的值不能被解释为数字，那么DECR命令将返回一个错误。
- 本操作的值限制在64位有符号数字表示之内。
- DECR命令会返回键key在执行减一操作之后的值。

-----

**DECRBY**

- DECRBY key decrement
- 将键key储存的整数值减去减量decrement。
- 如果键key不存在，那么键key的值会先被初始化为0，然后再执行DECRBY命令。
- 如果键key储存的值不能被解释为数字，那么DECRBY命令将返回一个错误。
- 本操作的值限制在64位有符号数字表示之内。
- DECRBY命令会返回键在执行减法操作之后的值。

-----

**MSET**

- MSET key value [key value …]
- 同时为多个键设置值。
- 如果某个给定键已经存在，那么MSET将使用新值去覆盖旧值，如果这不是你所希望的效果，请考虑使用MSETNX。
- MSET是一个原子性操作，所有给定键都会在同一时间内被设置，不会出现某些键被设置了但是另一些键没有被设置的情况。
- MSET命令总是返回OK。

-----

**MSETNX**

- MSETNX key value [key value …]
- 当且仅当所有给定键都不存在时，为所有给定键设置值。
- 即使只有一个给定键已经存在，MSETNX命令也会拒绝执行对所有键的设置操作。
- MSETNX是一个原子性操作，所有给定键要么全部都被设置，要么就全部都不设置，不可能出现第三种状态。
- 当所有给定键都设置成功时，命令返回1；否则返回0。

-----

**MGET**

- MGET key [key …]
- 返回给定的一个或多个字符串键的值。
- 如果给定的字符串键里面，有某个键不存在，那么这个键的值将以特殊值nil表示。
- MGET命令将返回一个列表，列表中包括了所有给定键的值。

## 哈希表

-----

**HSET**

- HSET hash field value
- 将哈希表hash中域field的值设置为value。
- 如果给定的哈希表并不存在，那么一个新的哈希表将被创建并执行HSET操作。
- 如果域field已经存在于哈希表中，那么它的旧值将被新值value覆盖。
- 当HSET命令在哈希表中新创建field域并成功为它设置值时，命令返回1；如果域field已经存在于哈希表，并且HSET命令成功使用新值覆盖了它的旧值，那么命令返回0。

-----

**HSETNX**

- HSETNX hash field value
- 当且仅当域field尚未存在于哈希表的情况下，将它的值设置为value。
- 如果给定域已经存在于哈希表当中，那么命令将放弃执行设置操作。
- 如果哈希表hash不存在，那么一个新的哈希表将被创建并执行HSETNX命令。
- HSETNX命令在设置成功时返回1，在给定域已经存在而放弃执行设置操作时返回0。

-----

**HGET**

- HGET hash field
- 返回哈希表中给定域的值。
- 如果给定域不存在域哈希表中，又或者给定的哈希表并不存在，那么命令返回nil。

-----

**HEXISTS**

- HEXISTS hash field
- 检查给定域field是否存在于哈希表hash当中。
- HEXISTS命令在给定域存在时返回1，在给定域不存在时返回0。

-----

**HDEL**

- HDEL key field [field …]
- 删除哈希表key中的一个或多个指定域，不存在的域将被忽略。
- 返回被成功移除的域的数量，不包括被忽略的域。

-----

**HLEN**

- HLEN key
- 返回哈希表key中域的数量。
- 当key不存在时，返回0。

-----

**HSTRLEN**

- HSTRLEN key field
- 返回哈希表key中，与给定域field相关联的值的字符串长度。
- 如果给定的键或者域不存在，那么命令返回0。

-----

**HINCRBY**

- HINCRBY key field increment
- 为哈希表key中的域field的值加上增量increment。
- 增量也可以为负数，相当于给给定域进行减法操作。
- 如果key不存在，一个新的哈希表被创建并执行HINCRBY命令。
- 如果field不存在，那么在执行命令前，域的值被初始化为0。
- 对一个储存字符串值的域field执行HINCRBY命令将造成一个错误。
- 本操作的值被限制在64位有符号数字表示之内。
- 返回执行HINCRBY命令之后，哈希表key中域field的值。

-----

**HINCRBYFLOAT**

- HINCRBYFLOAT key field increment
- 为哈希表key中的域field加上浮点数增量increment。
- 如果哈希表中没有域field，那么HINCRBYFLOAT会先将域field的值设为0，然后再执行加法操作。
- 如果键key不存在，那么HINCRBYFLOAT会先创建一个哈希表，再创建域field，最后再执行加法操作。
- 当以下任意一个条件发生时，返回一个错误：
    - 域field的值不是字符串类型。
    - 域field当前的值或给定的增量increment不能解释为双精度浮点数。
- 返回执行加法操作之后field域的值。

-----

**HMSET**

- HMSET key field value [field value …]
- 同时将多个field-value对设置到哈希表key中。
- 此命令会覆盖哈希表中已存在的域。
- 如果key不存在，一个空哈希表被创建并执行HMSET操作。
- 如果命令执行成功，返回OK。
- 当key不是哈希表类型时，返回一个错误。

-----

**HMGET**

- HMGET key field [field …]
- 返回哈希表key中，一个或多个给定域的值。
- 如果给定的域不存在于哈希表，那么返回一个nil值。
- 因为不存在的key被当作一个空哈希表来处理，所以对一个不存在的key进行HMGET操作将返回一个只带有nil的表。
- 返回一个包含多个给定域的关联值的表，表值的排列顺序和给定域参数的请求顺序一样。

-----

**HKEYS**

- HKEYS key
- 返回哈希表key中的所有域field的表。
- 当key不存在时，返回一个空表。

-----

**HVALS**

- HVALS key
- 返回哈希表key中所有值的表。
- 当key不存在时，返回一个空表。

-----

**HGETALL**

- HGETALL key
- 返回哈希表key中，所有的域和值的表。
- 在返回值里，紧跟每个域名field之后是域的值value，所以返回值的长度是哈希表大小的两倍。
- 若key不存在，返回空列表。

## 列表

-----

**LPUSH**

- LPUSH key value [value …]
- 将一个或多个值value插入到列表key的表头
- 如果有多个value值，那么各个value值按从左到右的顺序依次插入到表头。
- 如果key不存在，一个空列表会被创建并执行LPUSH操作。
- 当key存在但不是列表类型时，返回一个错误。
- 返回执行LPUSH命令后，列表的长度。

-----

**LPUSHX**

- LPUSHX key value
- 将值value插入到列表key的表头，当且仅当key存在并且是一个列表。
- 当key不存在时，LPUSHX命令什么也不做。
- 返回LPUSHX命令执行之后，表的长度。

-----

**RPUSH**

- RPUSH key value [value …]
- 将一个或多个值value插入到列表key的表尾。
- 如果有多个value值，那么各个value值按从左到右的顺序依次插入到表尾。
- 如果key不存在，一个空列表会被创建并执行RPUSH操作。
- 当key存在但不是列表类型时，返回一个错误。
- 返回执行RPUSH操作后，表的长度。

-----

**RPUSHX**

- RPUSHX key value
- 将值value插入到列表key的表尾，当且仅当key存在并且是一个列表。
- 当key不存在时，RPUSHX命令什么都不做。
- 返回RPUSHX命令执行之后，表的长度。

-----

**LPOP**

- LPOP key
- 移除并返回列表key的头元素。
- 当key不存在时，返回nil。

-----

**RPOP**

- RPOP key
- 移除并返回列表key的尾元素。
- 当key不存在时，返回nil。

-----

**RPOPLPUSH**

- RPOPLPUSH source destination
- 命令RPOPLPUSH在一个原子时间内，执行以下两个动作：
    - 将列表source中的尾元素弹出，并返回给客户端。
    - 将source弹出的元素插入到列表destination，作为destination列表的头元素。
- 如果source不存在，值nil被返回，并且不执行其他动作。
- 如果source和destination相同，则列表中的表尾元素被移动到表头，并返回该元素，可以把这种特殊情况视作列表的旋转操作。

-----

**LREM**

- LREM key count value
- 根据参数count的值，移除列表中与参数value相等的元素。
- count的值可以是以下几种：
    - count > 0：从表头开始向表尾搜索，移除与value相等的元素，数量为count。
    - count < 0：从表尾开始向表头搜索，移除与value相等的元素，数量为count的绝对值。
    - count = 0：移除表中所有与value相等的值。
- 返回被移除元素的数量，因为不存在的key被视作空表，所以当key不存在时，LREM命令总是返回0。

-----

**LLEN**

- LLEN key
- 返回列表key的长度。
- 如果key不存在，则key被解释为一个空列表，返回0。
- 如果key不是列表类型，返回一个错误。

-----

**LINDEX**

- LINDEX key index
- 返回列表key中，下标为index的元素。
- 下标index参数start和stop都以0为底，也就是说，以0表示列表的第一个元素，以1表示列表的第二个元素，以此类推。
- 下标-1表示列表的最后一个元素，-2表示列表的倒数第二个元素，以此类推。
- 如果key不是列表类型，返回一个错误。
- 如果index参数的值不在列表的区间范围内，返回nil。

-----

**LINSERT**

- LINSERT key BEFORE|AFTER pivot value
- 将值value插入到列表key当中，位于值pivot之前或之后。
- 当pivot不存在于列表key时，不执行任何操作。
- 当key不存在时，key被视为空列表，不执行任何操作。
- 如果key不是列表类型，返回一个错误。
- 如果命令执行成功，返回插入操作完成之后，列表的长度。如果没有找到pivot，返回-1。如果key不存在或为空列表，返回0。

-----

**LSET**

- LSET key index value
- 将列表key下标为index的元素设置为value。
- 当index参数超出范围时，或对一个空列表进行LSET时，返回一个错误。
- 操作成功返回ok，否则返回错误信息。

-----

**LRANGE**

- LRANGE key start stop
- 返回列表key中指定区间内的元素，区间以偏移量start和stop指定。
- 下标index参数start和stop都以0为底，也就是说，以0表示列表的第一个元素，以1表示列表的第二个元素，以此类推。
- 也可以使用负数下标，以-1表示列表的最后一个元素，-2表示列表的倒数第二个元素，以此类推。
- start和stop下标都包含在内，是一个闭区间。
- 超出范围的下标值不会引起错误。
    - 如果start下标比列表的最大下标end还要大，那么LRANGE返回一个空列表。
    - 如果stop下标比end下标还要大，Redis将stop的值设置为end。

-----

**LTRIM**

- LTRIM key start stop
- 对一个列表进行修剪，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
- start和stop下标都被包含在内，闭区间。
- 当key不是列表类型时，返回一个错误。
- 下标index参数start和stop都以0为底，也就是说，以0表示列表的第一个元素，以1表示列表的第二个元素，以此类推。
- 也可以使用负数下标，以-1表示列表的最后一个元素，-2表示列表的倒数第二个元素，以此类推。
- start和stop下标都包含在内，是一个闭区间。
- 超出范围的下标值不会引起错误。
    - 如果start下标比列表的最大下标end还要大，那么LRANGE返回一个空列表。
    - 如果stop下标比end下标还要大，Redis将stop的值设置为end。
- 命令执行成功时，返回ok。

-----

**BLPOP**

- BLPOP key [key …] timeout
- BLPOP是列表的阻塞式弹出原语。
- 它式LPOP key命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被BLPOP命令阻塞，直到等待超时或发现可弹出元素为止。
- 当给定多个key参数时，按参数key的先后顺序依次检查各个列表，弹出第一个非空列表的头元素。
- 非阻塞行为
    - 当BLPOP被调用时，如果给定key内至少有一个非空列表，那么弹出遇到的第一个非空列表的头元素，并和被弹出元素所属的列表的名字一起，组成结果返回给调用者（[key, result]。
    - 当存在多个给定key时，BLPOP按给定key参数排列的先后顺序，依次检查各个列表。
- 阻塞行为
    - 如果所有给定key都不存在或包含空列表，那么BLPOP命令将阻塞连接，直到等待超时，或有另一个客户端对给定key的任意一个执行LPUSH或RPUSH命令为止。
    - 超时参数timeout接受一个以秒为单位的数字作为值。超时参数设为0表示阻塞时间可以无限期延长。
- 相同key被多个客户端同时阻塞
    - 相同key可以被多个客户端同时阻塞。
    - 不同的客户端被放进一个队列中，按先阻塞先服务的顺序为key执行BLPOP命令。
- 在MULTI/EXEC事务中的BLPOP
    - BLPOP可以用于流水线pipeline，但把它用在MULTI/EXEC块当中没有意义。因为这要求整个服务器被阻塞以保证块执行时的原子性，该行为阻止了其他客户端执行LPUSH或RPUSH命令。
    - 因此，一个被包裹在MULTI/EXEX块内的BLPOP命令，行为表现得就像LPOP key一样，对空列表返回nil，对非空列表弹出列表元素，不进行任何阻塞操作。
- 返回值
    - 如果列表为空，返回一个nil。否则，返回一个含有两个元素的列表，第一个元素时被弹出元素所属的key，第二个元素是被弹出元素的值。

-----

**BRPOP**

- BRPOP key [key …] timeout
- BRPOP是列表的阻塞式弹出原语。
- 用法同BLPOP，不过弹出的是尾元素。

-----

**BRPOPLPUSH**

- BRPOPLPUSH source destination timeout
- BRPOPLPUSH是RPOPLPUSH的阻塞版本，当给定列表source不为空时， BRPOPLPUSH的表现和RPOPLPUSH一样。
- 当列表source为空时，BRPOPLPUSH命令将阻塞连接，直到等待超时，或有另一个客户端对source执行LPUSH或RPUSH命令为止。
- 超时参数timeout接受一个以秒为单位的数字作为值。超时参数设为0表示阻塞时间可以无限期延长。
- 假如在指定时间内没有任何元素被弹出，则返回一个nil和等待时长。反之，返回一个含有两个元素的列表，第一个元素是被弹出元素的值，第二个元素是等待时长。

## 集合

-----

**SADD**

- SADD key member [member …]
- 将一个或多个member元素加入到集合key当中，已经存在于集合的member元素将被忽略。
- 假如key不存在，则创建一个只包含member元素作成员的集合。
- 当key不是集合类型时，返回一个错误。
- 返回被添加到集合中的新元素的数量，不包括被忽略的元素。

-----

**SISMEMBER**

- SISMEMBER key member
- 判断member元素是否集合key的成员。
- 如果member元素是集合的成员，返回1。如果member元素不是集合的成员，或key不存在，返回0。

-----

**SPOP**

- SPOP key
- 移除并返回集合中的一个随机元素。
- 如果只想获取一个随机元素，但不想该元素从集合中被移除的话，可以使用SRANDMEMBER key [count]命令。
- 当key不存在或key是空集时，返回nil。

-----

**SRANDMEMBER**

- SRANDMEMBER key [count]
- 如果命令执行时，只提供了key参数，那么返回集合中的一个随机元素。
- 从Redis2.6版本开始，SRANDMEMBER命令接受可选的count参数：
    - 如果count为正数，且小于集合基数，那么命令返回一个包含count个元素的数组，数组中的元素各不相同。如果count大于等于集合基数，那么返回整个集合。
    - 如果count为负数，那么命令返回一个数组，数组中的元素可能会重复出现多次，而数组的长度为count的绝对值。
- 该操作和SPOP key相似，但SPOP key将随机元素从集合中移除并返回，而SRANDMEMBER则仅仅返回随机元素，而不对集合进行任何改动。
- 只提供key参数时，返回一个元素；如果集合为空，返回nil。如果提供了count参数，那么返回一个数组；如果集合为空，返回空数组。

-----

**SREM**

- SREM key member [member …]
- 移除集合key中的一个或多个member元素，不存在的member元素会被忽略。
- 当key不是集合类型时，返回一个错误。
- 返回被成功移除的元素的数量，不包括被忽略的元素。

-----

**SMOVE**

- SMOVE source destination member
- 将member元素从source集合移动到destination集合。
- SMOVE是原子性操作。
- 如果source集合不存在或不包含指定的member元素，则SMOVE命令不执行任何操作，仅返回0。否则，member元素从source集合中被移除，并添加到destination集合中去。
- 当destination集合已经包含member元素时，SMOVE命令只是简单地将source集合中的member元素删除。
- 当source或destination不是集合类型时，返回一个错误。
- 如果member元素被成功移除，返回1。如果member元素不是source集合的成员，并且没有任何操作对destination集合执行，那么返回0。

-----

**SCARD**

- SCARD key
- 返回集合key的基数（集合中元素的数量）。
- 当key不存在时，返回0。

-----

**SMEMBERS**

- SMEMBERS key
- 返回集合key中的所有成员。
- 不存在的key被视为空集合。

-----

**SINTER**

- SINTER key [key …]
- 返回一个集合的全部成员，该集合是所有给定集合的交集。
- 不存在的key被视为空集。
- 当给定集合当中有一个空集时，结果也为空集。

-----

**SINTERSTORE**

- SINTERSTORE destination key [key …]
- 这个命令类似于SINTER key [key …]命令，但它将结果保存到destination集合，而不是简单地返回结果集。
- 如果destination集合已经存在，则将其覆盖。
- destination可以是key本身。
- 返回结果集中的成员数量。

-----

**SUNION**

- SUNION key [key …]
- 返回所有给定集合的并集。
- 不存在的key被视为空集。

-----

**SUNIONSTORE**

- SUNIONSTORE destination key [key …]
- 这个命令类似于SUNION，但它将结果保存到destination集合，而不是简单地返回结果集。
- 如果destination已经存在，则将其覆盖。
- destination可以是key本身。
- 返回集中的元素数量。

-----

**SDIFF**

- SDIFF key [key …]
- 返回所有给定集合之间的差集。
- 不存在的key被视为空集。

-----

**SDIFFSTORE**

- SDIFFSTORE destination key [key …]
- 这个命令的作用和SDIFF类似，但它将结果保存到destination集合，而不是简单地返回结果集。
- 如果destination集合已经存在，则将其覆盖。
- destination可以是key本身。
- 返回结果集中的元素数量。

## 有序集合

-----

**ZADD**

- ZADD key score member [[score member] [score member] …]
- 将一个或多个member元素及其score值加入到有序集key当中。
- 如果某个member已经是有序集的成员，那么更新这个member的score值，并通过重新插入这个member元素，来保证该member在正确的位置上。
- score值可以是整数值或双精度浮点数。
- 如果key不存在，则创建一个空的有序集并执行ZADD操作。
- 当key存在但不是有序集类型时，返回一个错误。
- 返回被成功添加的新成员的数量，不包括那些被更新的、已经存在的成员。

-----

**ZSCORE**

- ZSCORE key member
- 返回有序集key中，成员member的score值，以字符串形式表示。
- 如果member元素不是有序集key的成员，或key不存在，返回nil。

-----

**ZINCRBY**

- ZINCRBY key increment member
- 为有序集key的成员member的score值加上增量increment。
- 可以通过传递一个负数值increment，让score减去相应的值。
- 当key不存在，或member不是key的成员时，ZINCRBY key increment member等同于ZADD key increment member。
- 当key不是有序集类型时，返回一个错误。
- score值可以是整数值或双精度浮点数。
- 返回member成员的新score值，以字符串形式表示。

-----

**ZCARD**

- ZCARD key
- 当key存在且是有序集类型时，返回有序集的基数。当key不存在时，返回0。

-----

**ZCOUNT**

- ZCOUNT key min max
- 返回有序集key中，score值在min和max之间（默认包括等于）的成员的数量。

-----

**ZRANGE**

- ZRANGE key start stop [WITHSCORES]
- 返回有序集key中，指定区间内的成员。
- 其中成员的位置按score值递增来排序。
- 具有相同score值的成员按字典序来排列。
- 如果需要成员按score值递减来排列，请使用ZREVRANGE key start stop [WITHSCORES]命令。
- 下标参数start和stop都以0为底，也就是说，以0表示有序集第一个成员，以1表示有序集第二个成员，以此类推。你也可以使用负数下标，以-1表示最后一个成员，-2表示倒数第二个成员，以此类推。
- 超出范围的下标并不会引起错误。比如说，当start的值比有序集的最大下标还要大，或是start>stop时，ZRANGE命令只是简单地返回一个空列表。另一方面，假如stop参数的值比有序集的最大下标还要大，那么Redis将stop当作最大下标来处理。
- 可以通过使用WITHSCORES选项，来让成员和它的score值一并返回，返回列表以value1、socre1、value2、score2...的格式表示。客户端库可能会返回一些更复杂的数据类型，比如数组、元组等。
- 返回指定区间内，带有score值（可选）的有序集成员的列表。

-----

**ZREVRANGE**

- ZREVRANGE key start stop [WITHSCORES]
- 除了成员按score值递减的次序排列这一点外，ZREVRANGE命令的其他方面和ZRANGE命令一样。

-----

**ZRANGEBYSCORE**

- ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
- 返回有序集key中，所有score值介于min和max之间（包括等于min或max）的成员。有序集成员按score值递增次序排列。
- 具有相同score值的成员按字典序来排列（该属性是有序集提供的，不需要额外的计算）。
- 可选的LIMIT参数指定返回结果的数量及区间，注意当offset很大时，定位offset的操作可能需要遍历整个有序集，此过程最坏复杂度为O(N)时间。
- 可选的WITHSCORES参数决定结果集是单单返回有序集的成员，还是将有序集成员及其socre值一起返回。
- 区间及无限
    - min和max可以是-inf和+inf，这样一来，你就可以在不知道有序集的最低和最高score值的情况下，使用ZRANGEBYSCORE这类命令。
    - 默认情况下，区间的取值使用闭区间（小于等于或大于等于），你也可以通过给参数前增加(符号来使用可选的开区间（小于或大于）。
- 返回指定区间内，带有score值（可选）的有序集成员的列表。

-----

**ZREVRANGEBYSCORE**

- ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
- 除了成员按score值递减的次序排列这一点外，ZREVRANGEBYSCORE命令的其他方面和ZRANGEBYSCORE命令一样。

-----

**ZRANK**

- ZRANK key member
- 返回有序集key中成员member的排名。其中有序集成员按score值递增顺序排列。
- 排名以0为底，也就是说，score值最小的成员排名为0。
- 使用ZREVRANK命令可以获得成员按score值递减排列的排名。
- 如果member是有序集key的成员，返回member的排名。如果member不是有序集key的成员，返回nil。

-----

**ZREVRANK**

- ZREVRANK key member
- 返回有序集key中成员member的排名。其中有序集成员按score值递减顺序排列。
- 排名以0为底，也就是说，score值最大的成员排名为0。
- 使用ZRANK命令可以获得成员按score值递增排列的排名。
- 如果member是有序集key的成员，返回member的排名。如果member不是有序集key的成员，返回nil。

-----

**ZREM**

- ZREM key member [member …]
- 移除有序集key中的一个或多个成员，不存在的成员将被忽略。
- 当key存在但不是有序集类型时，返回一个错误。
- 返回被成功移除的成员的数量，不包括被忽略的成员。

-----

**ZREMRANGEBYRANK**

- ZREMRANGEBYRANK key start stop
- 移除有序集key中，指定排名区间内的所有成员。
- 区间分别以下标参数start和stop指出，包含start和stop在内。
- 下标参数start和stop都以0为底，也就是说，以0表示有序集第一个成员，以1表示有序集第二个成员，以此类推。你也可以使用负数下标，以-1表示最后一个成员，-2表示倒数第二个成员，以此类推。
- 返回被移除成员的数量。

-----

**ZREMRANGEBYSCORE**

- ZREMRANGEBYSCORE key min max
- 移除有序集key中，所有score值介于min和max之间（包括等于min或max）的成员。
- score值等于min或max的成员也可以不包括在内，同ZRANGEBYSCORE命令。
- 返回被移除成员的数量。

-----

**ZRANGEBYLEX**

- ZRANGEBYLEX key min max [LIMIT offset count]
- 当有序集合的所有成员都具有相同的分值时，有序集合的元素会根据成员的字典序来进行排序，而这个命令则可以返回给定的有序集合键key中，值介于min和max之间的成员。
- 如果有序集合里面的成员带有不同的分值，那么命令返回的结果是未指定的。
- 命令会使用C语言的memcmp()函数，对集合中的每个成员进行逐个字节的对比，并按照从低到高的顺序，返回排序后的集合成员。如果两个字符串有一部分内容是相同的话，那么命令会认为较长的字符串比较短的字符串要大。
- 可选的LIMIT offset count参数用于获取指定范围内的匹配元素。如果offset参数的值非常大的话，那么命令在返回结果之前，需要先遍历至offset所指定的位置，这个操作会为命令加上最多O(N)复杂度。
- 合法的min和max参数必须包含(或者[，其中(表示开区间，而[则表示闭区间。
- 特殊值+和-在min参数以及max参数中具有特殊的意义，其中+表示正无限，而-表示负无限。因此，向一个所有成员的分值都相同的有序集合发送命令`ZRANGEBYLEX <zset> - +`，命令将返回有序集合中的所有元素。
- 返回一个列表，包含了有序集合在指定范围内的成员。

-----

**ZLEXCOUNT**

- ZLEXCOUNT key min max
- 对于一个所有成员的分值都相同的有序集合键key来说，这个命令会返回该集合中，成员介于min和max范围内的元素数量。
- 这个命令的min参数和max参数的意义和ZRANGEBYLEX命令一样。
- 返回指定范围内的元素数量。

-----

**ZREMRANGEBYLEX**

- ZREMRANGEBYLEX key min max
- 对于一个所有成员的分值都相同的有序集合键key来说，这个命令会移除该集合中，成员介于min和max范围内的所有元素。
- 这个命令的min参数和max参数的意义和ZRANGEBYLEX命令一样。
- 返回被移除的元素数量。

-----

**ZUNIONSTORE**

- ZUNIONSTORE destination numkeys key [key …] [WEIGHTS weight [weight …]] [AGGREGATE SUM|MIN|MAX]
- 计算给定的一个或多个有序集的并集，其中给定key的数量必须以numkeys参数指定，并将该并集储存到destination。
- 默认情况下，结果集中某个成员的score值是所有给定集下该成员score值之和。
- WEIGHTS
    - 使用WEIGHTS选项，你可以为每个给定有序集分别指定一个乘法因子，每个给定有序集的所有成员的score值在传递给集合函数之前都要先乘以该有序集的因子。
    - 如果没有指定WEIGHTS选项，乘法因子默认设置为1。
- AGGREGATE
    - 使用AGGREGATE选项，你可以指定并集的结果集的聚合方式。
    - 默认使用的参数SUM，为加法；使用MIN，用最小的那个值；使用MAX，用最大的那个值。
- 返回保存到destination结果集的基数。

-----

**ZINTERSTORE**

- ZINTERSTORE destination numkeys key [key …] [WEIGHTS weight [weight …]] [AGGREGATE SUM|MIN|MAX]
- 计算给定的一个或多个有序集的交集，其中给定key的数量必须以numkeys参数指定，并将该交集储存到destination。
- 可选参数同ZUNIONSTORE
- 返回保存到destination结果集的基数。

## 数据库

-----

**EXISTS**

- EXISTS key
- 检查给定key是否存在，若key存在，返回1，否则返回0。

-----

**TYPE**

- TYPE key
- 返回key所储存的值的类型：
    - none（key不存在）
    - string（字符串）
    - list（列表）
    - set（集合）
    - zset（有序集）
    - hash（哈希表）
    - stream（流）

-----

**RENAME**

- RENAME key newkey
- 将key改名为newkey。
- 当key和newkey相同，或者key不存在时，返回一个错误。
- 当newkey已经存在时，RENAME命令将覆盖旧值。
- 改名成功时提示OK，失败时候返回一个错误。

-----

**RENAMENX**

- RENAMENX key newkey
- 当且仅当newkey不存在时，将key改名为newkey。
- 当key不存在时，返回一个错误。
- 修改成功时，返回1；如果newkey已经存在，返回0。

-----

**MOVE**

- MOVE key db
- 将当前数据库的key移动到给定的数据库db当中。
- 如果当前数据库和给定数据库有相同名字的给定key，或者key不存在于当前数据库，那么MOVE没有任何效果。
- 因此也可以利用这一特性，将MOVE当作锁原语。
- 移除成功返回1，失败则返回0。

-----

**DEL**

- DEL key [key …]
- 删除给定的一个或多个key。
- 不存在的key会被忽略。
- 返回被删除key的数量。

-----

**RANDOMKEY**

- RANDOMKEY
- 从当前数据库中随机返回（不删除）一个key。
- 当数据库不为空时，返回一个key。当数据库为空时，返回nil。

-----

**DBSIZE**

- DBSIZE
- 返回当前数据库的key的数量。

-----

**KEYS**

- KEYS pattern
- 查找所有符号给定模式pattern的key，比如说：
    - KEYS *匹配数据库中所有key。
    - kEYS h?llo匹配hello，hallo和hxllo等。
    - kEYS h*llo匹配hllo和heeeello等。
    - KEYS h[ae]llo匹配hello和hallo，但不匹配hillo。
- 特殊符号用\隔开。
- 返回给定模式的key列表。

-----

**SCAN**

- SCAN cursor [MATCH pattern] [COUNT count]
- SCAN命令及其相关的SSCAN命令、HSCAN命令和ZSCAN命令都用于增量地迭代一集元素：
    - SCAN命令用于迭代当前数据库中的数据库键。
    - SSCAN命令用于迭代集合键中的元素。
    - HSCAN命令用于迭代哈希键中的键值对。
    - ZSCAN命令用于迭代有序集合中的元素（包括元素成员和元素分值）。
- 以上列出的四个命令都支持增量式迭代，它们每次执行都只会返回少量元素，所以这些命令可以用于生产环境，而不会出现像KEYS命令、SMEMBERS命令带来的问题——当KEYS命令被用于处理一个大的数据库时，又或者SMEMBERS命令被用于处理一个大的集合键时，它们可能会阻塞服务器达数秒之久。
- 不过，增量式迭代命令也不是没有缺点的：举个例子，使用SMEMBERS命令可以返回集合键当前包含的所有元素，但是对于SCAN这类增量式迭代命令来说，因为在对键进行增量式迭代的过程中，键可能会被修改，所以增量式迭代命令只能对被返回的元素提供有限的保证。
- 因为SCAN、SSCAN、HSCAN和ZSCAN四个命令的工作方式都非常相似，所以这个文档会一并介绍这四个命令
    - SSCAN命令、HSCAN命令和ZSCAN命令的第一个参数总是一个数据库键。
    - 而SCAN命令则不需要在第一个参数提供任何数据库键——因为它迭代的是当前数据库中的所有数据库键。
- SCAN命令的基本用法
    - SCAN命令是一个基于游标的迭代器：SCAN命令每次被调用之后，都会向用户返回一个新的游标，用户在下次迭代时需要使用这个新游标作为SCAN命令的游标参数，以此来延续之前的迭代过程。
    - 当SCAN命令的游标参数被设置为0时，服务器将开始一次新的迭代，而当服务器向用户返回值为0的游标时，表示迭代已结束。
    - SCAN命令的回复是一个包含两个元素的数组，第一个数组元素是用于进行下一次迭代的新游标，而第二个数组元素则是一个数组，这个数组中包含了所有被迭代的元素。
    - 以0作为游标开始一次新的迭代，一直调用SCAN命令，直到命令返回游标0，我们称这个过程为一次完整遍历。
- SCAN命令的保证
    - SCAN命令，以及其他增量式迭代命令，在进行完整遍历的情况下可以为用户带来以下保证：从完整遍历开始直到完整遍历结束期间，一直存在于数据集内的所有元素都会被完整遍历返回；这意味着，如果有一个元素，它从遍历开始直到遍历结束期间都存在于被遍历的数据集当中，那么SCAN命令总会在某次迭代中将这个元素返回给用户。
    - 然而因为增量式命令仅仅使用游标来记录迭代状态，所以这些命令带有以下缺点：
        - 同一个元素可能会被返回多次。处理重复元素的工作交由应用程序负责，比如说，可以考虑将迭代返回的元素仅仅用于可以安全地重复执行多次的操作上。
        - 如果一个元素是在迭代过程中被添加到数据集的，又或者是在迭代过程中从数据集中被删除的，那么这个元素可能会被返回，也可能不会，这是未定义的。
- SCAN命令每次执行返回的元素数量
    - 增量式迭代命令并不保证每次执行都返回某个给定数量的元素。
    - 增量式命令甚至可能会返回零个元素，但只要命令返回的游标不是0，应用程序就不应该将迭代视作结束。
    - 不过命令返回的元素数量总是符合一定规则的，在实际中：
        - 对于一个大数据集来说，增量式迭代命令每次最多可能会返回数十个元素。
        - 而对于一个足够小的数据集来说，如果这个数据集的底层表示为编码数据结构，那么增量迭代命令将在一次调用中返回数据集中的所有元素。
    - 用户可以通过增量式迭代命令提供的COUNT选项来指定每次迭代返回元素的最大值。
- COUNT选项
    - 虽然增量式迭代命令不保证每次迭代所返回的元素数量，但我们可以使用COUNT选项，对命令的行为进行一定程度上的调整。
    - 基本上，COUNT选项的作用就是让用户告知迭代命令，在每次迭代中应该从数据集里返回多少元素。
    - 虽然COUNT选项只是对增量式迭代命令的一种提示，但是在大多数情况下，这种提示都是有效的。
        - COUNT参数的默认值为10。
        - 在迭代一个足够大的、由哈希表实现的数据库、集合键、哈希键或者有序集合键时，如果用户没有使用MATCH选项，那么命令返回的元素数量通常和COUNT选项指定的一样，或者比COUNT选项指定的数量稍多一些。
        - 在迭代一个编码为整数集合、或者编码为压缩列表时，增量式迭代命令通常会无视COUNT选项指定的值，在第一次迭代就将数据集包含的所有元素都返回给用户。
        - 并非每次迭代都要使用相同的COUNT值。
        - 用户可以在每次迭代中按自己的需要随意改变COUNT值，只要记得将上次迭代返回的游标用到下次迭代里就可以了。
- MATCH选项
    - 和KEYS命令一样，增量式迭代命令也可以通过提供一个glob风格的模式参数，让命令只返回和给定模式相匹配的元素，这一点可以通过在执行增量式迭代命令时，通过给定MATCH <pattern>参数来实现。比如`sscan key 0 match f*`
    - 对元素的模式匹配工作是在命令从数据集中取出元素之后，向客户端返回元素之前的这段时间内进行的，所有如果被迭代的数据集中只有少量元素和模式相匹配，那么迭代命令或许会在多次执行中都不返回任何元素。
- 并发执行多个迭代
    - 在同一时间，可以有任意多个客户端对同一数据集进行迭代，客户端每次执行都需要传入一个游标，并在迭代执行之后获得一个新的游标，而这个游标就包含了迭代的所有状态，因此，服务器无须为迭代记录任何状态。
- 中途停止迭代
    - 因为迭代的所有状态都保存在游标里面，而服务器无须为迭代保存任何状态，所以客户端可以在中途停止一个迭代，而无须对服务器进行任何通知。
    - 即使有任意数量的迭代在中途停止，也不会产生任何问题。
- 使用错误的游标进行增量式迭代
    - 使用间断的、负数、超出范围或者其他非正常的游标来执行增量式迭代并不会造成服务器崩溃，但可能会让命令产生未定义的行为。
    - 未定义行为指的是，增量式命令对返回值所做的保证可能会不再为真。
    - 只有两种游标是合法的：
        - 1、在开始一个新的迭代时，游标必须为0。
        - 2、增量式迭代命令在执行之后返回的，用于延续迭代过程的游标。
- 迭代终结的保证
    - 增量式迭代命令所使用的算法只保证在数据集的大小有界的情况下，迭代才会停止，换句话说，如果被迭代数据集的大小不断地增长的话，增量式迭代命令可能永远也无法完成一次完整迭代。
    - 从直觉上可以看出，当一个数据集不断地变大时，想要访问这个数据集中的所有元素就需要做越来越多的工作，能否结束一个迭代取决于用户执行迭代的速度是否比数据集增长的速度更快。
- 返回值
    - SCAN命令、SSCAN命令、HSCAN命令和ZSCAN命令都返回一个包含两个元素的multi-bulk回复：回复的第一个元素是字符串表示的无符号64位整数，回复的第二个元素是另一个multi-bulk回复，这个multi-bulk回复包含了本次被迭代的元素。
    - SCAN命令返回的每个元素都是一个数据库键。
    - SSCAN命令返回的每个元素都是一个集合成员。
    - HSCAN命令返回的每个元素都是一个键值对，一个键值对由一个键和一个值组成。
    - ZSCAN命令返回的每个元素都是一个有序集合元素，一个有序集合元素由一个成员和一个分值组成。

-----

**SORT**

- SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern …]] [ASC | DESC] [ALPHA] [STORE destination]
- 返回或保存给定列表、集合、有序集合key中经过排序的元素。
- 排序默认以数字作为对象，值被解释为双精度浮点数，然后进行比较。
- 最简单的SORT使用方法是SORT key和SORT key DESC。
    - SORT key返回键值从小到大排序的结果。
    - SORT key DESC返回键值从大到小排序的结果。
- 因为SORT命令默认排序对象为数字，当需要对字符串进行排序时，需要显式地在SORT命令之后添加ALPHA修饰符。
- 排序之后返回元素的数量可以通过LIMIT修饰符进行限制，修饰符接受offset和count两个参数：
    - offset指定要跳过的元素数量。
    - count指定跳过offset个指定的元素之后，要返回多少个对象。
- 可以使用外部key的数据作为权重， 代替默认的直接对比的键值的方式来进行排序。
    - 通过BY选项，可以让key让其他键的元素来排序，比如`SORT key BY other_*`让key键按照other_{key}的大小来排序。other_*是一个占位符，它先取出key中的值，然后在用这个值来查找对应的键。
    - 使用GET选项，可以根据排序的结果来取出相应的键值，比如`SORT key GET other_*`，先排序key，在取出键other_{key}的值

-----

**FLUSHDB**

- FLUSHDB
- 清空当前数据库中的所有key。
- 此命令从不失败。
- 总是返回OK。

-----

**FLUSHALL**

- FLUSHALL
- 清空整个Redis服务器的数据（删除所有数据库的所有key）。
- 此命令从不失败。
- 总是返回OK。

-----

**SELECT**

- SELECT index
- 切换到指定的数据库，数据库索引号index用数字值指定，以0作为起始索引值。
- 默认使用0号数据库。
- 返回OK。

-----

**SWAPDB**

- SWAPDB db1 db2
- 对换指定的两个数据库，使得两个数据库的数据立即互换。
- 返回OK。

## 自动过期

-----

**EXPIRE**

- EXPIRE key seconds
- 为给定key设置生存时间，当key过期时，它会被自动删除。
- 在Redis中，带有生存时间的key被称为易失的。
- 生存时间可以通过使用DEL命令来删除整个key来移除，或者被SET和GETSET命令覆写，这意味着，如果一个命令只是修改一个带生存时间的key的值而不是用一个新的key值来代替它的话，那么生存时间不会被改变。
- 比如说，对一个key执行INCR命令，对一个列表进行LPUSH命令，或者对一个哈希表执行HSET命令，这类操作都不会修改key本身的生存时间。
- 另一方面，如果使用RENAME对一个key进行改名，那么改名后的key的生成时间和改名前一样。
- RENAME命令的另一种可能是，尝试将一个带生存时间的key改名成另一个带生存时间的another_key，这时旧的another_key会被删除，然后旧的key会改名为another_key，因此，新的another_key的生存时间也和原本的key一样。
- 使用PERSIST命令可以在不删除key的情况下，移除key的生存时间，让key重新成为一个持久的key。
- 可以对一个已经带有生存时间的key执行EXPIRE命令，新指定的生存时间会取代旧的生存时间。
- 设置成功返回1。当key不存在或者不能为key设置生存时间时，返回0。

-----

**EXPIREAT**

- EXPIREAT key timestamp
- EXPIREAT的作用和EXPIRE类似，都用于为key设置生存时间。
- 不同在于EXPIREAT命令接受的时间参数是UNIX时间戳。
- 如果生存时间设置成功，返回1；当key不存在或没办法设置生存时间， 返回0。

-----

**TTL**

- TTL key
- 以秒为单位，返回给定key的剩余生存时间。
- 当key不存在时，返回-2。当key存在但没有设置剩余生存时间时，返回-1。否则，以秒为单位，返回key的剩余生存时间。

-----

**PERSIST**

- PERSIST key
- 移除给定key的生存时间，将这个key从易失的转换成持久的。
- 当生存时间移除成功时，返回1，如果key不存在或key没有设置生存时间，返回0。

-----

**PEXPIRE**

- PEXPIRE key milliseconds
- 这个命令和EXPIRE命令的作用类似，但是它以毫秒为单位设置key的生存时间，而不是秒。
- 设置成功，返回1；key不存在或设置失败，返回0。

-----

**PEXPIREAT**

- PEXPIREAT key milliseconds-timestamp
- 这个命令和EXPIREAT命令类似，但它以毫秒为单位设置key的过期unix时间戳，而不是像expireat那样，以秒为单位。
- 如果生存时间设置成功，返回1。当key不存在或没办法设置生存时间时，返回0。

-----

**PTTL**

- PTTL key
- 这个命令类似于TTL命令，但它以毫秒为单位返回key的剩余生存时间，而不是秒。
- 当key不存在时，返回-2；当key存在但是没有设置剩余生存时间时，返回-1；否则，以毫秒为单位，返回key的剩余生存时间。

## 事务

-----

**MULTI**

- MULTI
- 标记一个事务块的开始。
- 事务块内的多条命令会按照先后顺序被放进一个队列当中，最后由EXEC命令原子性地执行。
- 总是返回OK。

-----

**EXEC**

- EXEC
- 执行所有事务块内的命令。
- 假如某个key正处在WATCH命令的监视之下，且事务块中有和这个key相关的命令，那么EXEC命令只在这个key没有被其他命令所改动的情况下执行并生效，否则该事务被打断。
- 返回事务块内所有命令的返回值，按命令执行的先后顺序排列；当操作被打断时，返回空值nil。

-----

**DISCARD**

- DISCARD
- 取消事务，放弃执行事务块内的所有命令。
- 如果正在使用WATCH命令监视某个key，那么取消所有监视，等同于执行命令UNWATCH。
- 总是返回OK。

-----

**WATCH**

- WATCH key [key …]
- 监视一个或多个key，如果在事务执行之前这个或这些key被其他命令所改动，那么事务将被打断。
- 总是返回OK。

-----

**UNWATCH**

- UNWATCH
- 取消WATCH命令对所有key的监视。
- 如果在执行WATCH命令之后，EXEC命令或DISCARD命令先被执行了的话，那么就不需要再执行UNWATCH了。
- 因为EXEC命令会执行事务，因此WATCH命令的效果已经产生了；而DISCARD命令在取消事务的同时也会取消所有对key的监视，因此这两个命令执行之后，就没有必要执行UNWATCH了。
- 总是返回OK。

## HyperLogLog

-----

**PFADD**

- PFADD key element [element …]
- 将任意数量的元素添加到指定的HyperLogLog里面。
- 作为这个命令的副作用，HyperLogLog内部可能会被更新，以便反映一个不同的唯一元素估计数量（也即是集合的基数）。
- 如果HyperLogLog估计的近似基数在命令执行之后出现了变化，那么命令返回1，否则返回0。如果命令执行时给定的键不存在，那么程序将先创建一个空的HyperLogLog结构，然后再执行命令。
- 调用此命令时可以只给定键名而不给定元素
    - 如果给定键已经是一个HyperLogLog，那么这种调用不会产生任何效果。
    - 但如果给定的键不存在，那么命令会创建一个空的HyperLogLog，并向客户端返回1。
- 如果HyperLogLog的内部储存被修改了，那么返回1，否则返回0。

-----

**PFCOUNT**

- PFCOUNT key [key …]
- 当PFCOUNT key [key …]命令作用于单个键时，返回储存在给定键的HyperLogLog的近似基数，如果键不存在，那么返回0。
- 当PFCOUNT key [key …]命令作用于多个键时，返回所有给定HyperLogLog的并集的近似基数，这个近似基数是通过将所有给定HyperLogLog合并至一个临时HyperLogLog来计算得出的。
- 通过HyperLogLog数据结构，用户可以使用少量固定大小的内存，来储存集合中的唯一元素（每个HyperLogLog只需使用12k字节内存，以及几个字节的内存来储存键本身）。
- 命令返回的可见集合基数并不是精确值，而是一个带有0.81%标准错误的近似值。
- 返回给定HyperLogLog包含的唯一元素的近似数量。

-----

**PFMERGE**

- PFMERGE destkey sourcekey [sourcekey …]
- 将多个HyperLogLog合并为一个HyperLogLog，合并后的HyperLogLog的基数接近于所有输入HyperLogLog的可见集合的并集。
- 合并得出的hyperLogLog会被储存在destkey键里面，如果该键并不存在，那么命令在执行之前，会先为该键创建一个空的HyperLogLog。
- 返回OK。

## 地理位置

-----

**GEOADD**

- GEOADD key longitude latitude member [longitude latitude member …]
- 将给定的空间元素（纬度、经度、名字）添加到指定的键里面。这些数据会以有序集合的形式被储存在键里面，从而使得像GEORADIUS和GEORADIUSBYMEMBER这样的命令可以在之后通过位置查询取得这些元素。
- GEOADD命令以标准的x,y格式接受参数，所以用户必须先输入经度，然后再输入纬度。
- GEOADD能够记录的坐标是有限的：非常接近两极的区域是无法被索引的。精确的坐标限制由坐标系统定义，具体如下：
    - 有效的经度介于-180度至180度之间。
    - 有效的纬度介于-85.05112878度至85.05112878度之间。
- 当用户尝试输入一个超出范围的经度或者纬度时，GEOADD命令将返回一个错误。
- 返回新添加到键里面的空间元素数量，不包括那些已经存在但是被更新的元素。

-----

**GEOPOS**

- GEOPOS key member [member …]
- 从键里面返回所有给定位置元素的位置（经度和纬度）。
- 因为GEOPOS命令接受可变数量的位置元素作为输入，所以即使用户只给定了一个位置元素，命令也会返回数组回复。
- 返回一个数组，数组中的每个项都由两个元素组成：第一个元素为给定位置元素的经度，而第二个元素则为给定位置元素的纬度。当给定的位置元素不存在时，对应的数组项为空值。

-----

**GEODIST**

- GEODIST key member1 member2 [unit]
- 返回两个给定位置之间的距离。
- 如果两个位置之间的其中一个不存在，那么命令返回空值。
- 指定单位的参数unit必须是以下单位的其中一个：
    - m表示单位为米
    - km表示单位为千米
    - mi表示单位为英里
    - ft表示单位为英尺
- 如果用户没有显式地指定单位参数，那么GEODIST默认使用米作为单位。
- GEODIST命令在计算距离时会假设地球为完美的球形，在极限情况，这一假设最大会造成0.5%的误差。
- 计算出的距离会以双精度浮点数的形式被返回。如果给定的位置元素不存在，那么命令返回空值。

-----

**GEORADIUS**

- GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [ASC|DESC] [COUNT count]
- 以给定的经纬度为中心，返回键包含的位置元素当中，与中心的距离不超过给定的最大距离的所有位置元素。
- 范围可以使用以下其中一个单位：
    - m表示单位为米
    - km表示单位为千米
    - mi表示单位为英里
    - ft表示单位为英尺
- 在给定以下可选项时，命令会返回额外的信息：
    - WITHDIST：在返回位置元素的同时，将位置元素与中心之间的距离也一并返回。距离的单位和用户给定的范围单位保持一致。
    - WITHCOORD：将位置元素的经度和纬度也一并返回。
    - WITHHASH：以52位有符号整数的形式，返回位置元素经过原始geohash编码的有序集合分值。这个选项主要用于底层应用或者调试，实际中的作用并不大。
- 命令默认返回未排序的位置元素。通过以下两个参数，用户可以指定被返回位置元素的排序方式：
    - ASC：根据中心的位置，按照从近到远的方式返回位置元素。
    - DESC：根据中心的位置，按照从远到近的方式返回位置元素。
- 在默认情况下，GEORADIUS命令会返回所有匹配的位置元素。虽然用户可以使用COUNT <count>选项去获取前N个匹配元素，但是因为命令在内部可能会需求对所有被匹配的元素进行处理，所以在对一个非常大的区域进行搜索时，即使只使用COUNT选项去获取少量元素，命令的执行速度也可能会非常慢。但是从另一个方面来说，使用COUNT选项去减少需求返回的元素数量，对于减少带宽来说仍然是非常有用的。
- GEORADIUS命令返回一个数组，具体来说：
    - 在没有给定任何WITH选项的情况下，命令只会返回一个像`["New York", "Milan", "Paris"]`这样的线性列表。
    - 在指定了WITHCOORD、WITHDIST、WITHHASH等选项的情况下，命令返回一个二层嵌套数组，内层的每个子数组就表示一个元素。
- 在返回嵌套数组时，子数组的第一个元素总是位置元素的名字。至于额外的信息，则会作为子数组的后续元素，按照以下顺序被返回：
    - 1、以浮点数格式返回的中心与位置元素之间的距离，单位与用户指定范围时的单位一致。
    - 2、geohash整数。
    - 3、由两个元素组成的坐标，分别为经度和纬度。

-----

**GEORADIUSBYMEMBER**

- GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [ASC|DESC] [COUNT count]
- 这个命令和GEORADIUS命令一样，都可以找出位于指定范围内的元素，但是GEORADIUSBYMEMBER的中心点是由给定的位置元素决定的，而不是像GEORADIUS那样，使用输入的经度和纬度来决定中心点。
- 返回一个数组，数组中的每个项表示一个范围之内的位置元素。

-----

**GEOHASH**

- GEOHASH key member [member …]
- 返回一个数组，数组的每个项都是一个geohash。命令返回的geohash的位置与用户给定的位置元素的位置一一对应。

## 位图

-----

**SETBIT**

- SETBIT key offset value
- 对key所储存的字符串值，设置或清除指定偏移量上的位（bit）。
- 位的设置或清除取决于value参数，可以是0也可以是1。
- 当key不存在时，自动生成一个新的字符串值。
- 字符串会进行伸展以确保它可以将value保存在指定的偏移量上。当字符串值进行伸展时，空白位置以0填充。
- offset参数必须大于或等于0，小于2^32（bit映射被限制在512MB之内）。
- 返回指定偏移量原来储存的位。

-----

**GETBIT**

- GETBIT key offset
- 对key所储存的字符串值，获取指定偏移量上的位（bit）。
- 当offset比字符串值的长度大，或者key不存在时，返回0。

-----

**BITCOUNT**

- BITCOUNT key [start] [end]
- 计算给定字符串中，被设置为1的比特位的数量。
- 一般情况下，给定的整个字符串都会被进行计数，通过指定额外的start或end参数，可以让计数只在特定的位上进行。
- start和end参数的设置和GETRANGE命令类似，都可以使用负数值：比如-1表示最后一个字节，以此类推。
- 不存在的key被当成是空字符串来处理，因此对一个不存在的key进行BITCOUNT操作，结果为0。

-----

**BITPOS**

- BITPOS key bit [start] [end]
- 返回位图中第一个值为bit的二进制位的位置，返回一个整数。
- 在默认情况下，命令将检测整个位图，但用户也可以通过可选的start参数和end参数指定要检测的范围。

-----

**BITOP**

- BITOP operation destkey key [key …]
- 对一个或多个保存二进制位的字符串key进行位元操作，并将结果保存到destkey上。
- operation可以是AND、OR、NOT、XOR这四种操作中的任意一种：
    - AND：逻辑并
    - OR：逻辑或
    - NOT：逻辑非，只接受一个key
    - XOR：逻辑异或
- 除了NOT操作之外，其他操作都可以接受一个或多个key作为输入。
- 当BITOP处理不同长度的字符串时，较短的那个字符串所缺少的部分会被看作0。
- 空的key也被看作是包含0的字符串序列。
- 返回保存到destkey的字符串的长度，和输入key中最长的字符串长度相等。

-----

**BITFIELD**

- BITFIELD key [GET type offset] [SET type offset value] [INCRBY type offset increment] [OVERFLOW WRAP|SAT|FAIL]
- BITFIELD命令可以将一个Redis字符串看作是一个由二进制位组成的数组，并对这个数组中储存的长度不同的整数进行访问（被储存的整数无需进行对齐）。换句话说，通过这个命令，用户可以执行诸如“对偏移量1234上的5位长有符号整数进行设置”、“获取偏移量4567上的31位长无符号整数”等操作。此外，BITFIELD命令还可以对指定的整数执行加法操作和减法操作，并且这些操作可以通过设置妥善地处理计算时出现的溢出情况。
- BITFIELD命令可以在一次调用中同时对多个位范围进行操作：它接受一系列待执行的操作作为参数，并返回一个数组作为回复，数组中的每个元素就是对应操作的执行结果。
- 注意：
    - 使用GET子命令对超出字符串当前范围的二进制位进行访问（包括键不存在的情况），超出部分的二进制位的值将被当做是0。
    - 使用SET子命令或者INCRBY子命令对超出字符串当前范围的二进制位进行访问将导致字符串被扩大，被扩大的部分会使用值位0的二进制位进行填充。在对字符串进行扩展时，命令会根据字符串目前已有的最远端二进制位，计算出执行操作所需的最小长度。
- 以下是BITFIELD命令支持的子命令：
    - `GET<type><offset>`——返回指定的二进制位范围。
    - `SET<type><offset><value>`——对指定的二进制位范围进行设置，并返回它的旧值。
    - `INCRBY<type><offset><increment>`——对指定的二进制位范围执行加法操作，并返回它的旧值。用户可以通过向increment参数传入负值来实现相应的减法操作。
- 除了以上三个子命令之外，还有一个子命令，它可以改变之后执行的INCRBY子命令在发生溢出情况时的行为：
    - OVERFLOW [WRAP|SAT|FAIL]
- 当被设置的二进制位范围值为整数时，用户可以在类型参数的前面添加i来表示有符号整数，或者使用u来表示无符号整数。比如说，我们可以使用u8来表示8位长的无符号整数，也可以使用i16来表示16位长的有符号整数。
- BITFIELD命令最大支持64位长的有符号整数以及63位长的无符号整数。
- 在二进制位范围命令中，用户有两种方法来设置偏移量：
    - 如果用户给定的是一个没有任何前缀的数字，那么这个数字指示的就是字符串以零为开始的偏移量。
    - 另一方面，如果用户给定的是一个带有#前缀的偏移量，那么命令将使用这个偏移量与被设置的数字类型的位长度相乘，从而计算出真正的偏移量。
    - 比如`BITFIELD mystring SET i8 #0 100 i8 #1 200`，命令会把mystring键里面，第一个i8长度的二进制位的值设置为100，并把第二个i8长度的二进制位的值设置为200。当我们把第一个字符串键当成数组来使用，并且数组中储存的都是同等长度的整数时，使用#前缀可以让我们免去手动计算被设置二进制位所在位置的麻烦。
- 用户可以通过OVERFLOW命令以及以下展示的三个参数，指定BITFIELD命令在执行自增或者自减操作时，碰上向上溢出或者向下溢出情况时的行为：
    - WRAP：使用回绕方法处理有符号整数和无符号整数的溢出情况。对于无符号整数来说，回绕就像使用数值本身与能够被储存的最大无符号整数执行取模计算，这也是C语言的标准行为。对于有符号整数来说，上溢将导致数字重新从最小的负数开始计算，而下溢将导致数字重新从最大的正数开始计算。比如说，如果我们对一个值为127的i8整数执行加一操作，那么将得到结果-128。
    - SAT：使用饱和计算方法处理溢出，也即是说，下溢计算的结果为最小的整数值，而上溢计算的结果为最大的整数值。举个例子，如果我们对一个值为120的i8整数执行加10计算，那么命令的结果将为i8类型所能储存的最大整数值127。与此相反，如果一个针对i8值的计算造成了下溢，那么这个i8值将被设置为-127。
    - FAIL：在这一模式下，命令将拒绝执行那些会导致上溢或者下溢情况出现的计算，并向用户返回空值表示计算未被执行。
- 需要注意的是，OVERFLOW子命令只会对紧随着它之后被执行的INCRBY命令产生效果，这一效果将一直持续到与它一同被执行的下一个OVERFLOW命令为止。在默认情况下，INCRBY命令使用WRAP方式来处理溢出计算。
- BITFIELD命令的作用在于它能够将很多小的整数储存到一个长度较大的位图中，又或者将一个非常庞大的键分割位多个较小的键来进行储存，从而非常高校地使用内存，使得Redis能够得到更多不同的应用——特别是在实时分析领域：BITFIELD能够以指定的方式对计算溢出进行空值的能力，使得它可以被应用于这一领域。
- BITFIELD在一般情况下都是一个快速的命令，需要注意的是，访问一个长度较短的字符串的远端二进制位将引发一次内存分配操作，这一操作花费的时间可能会比命令访问已有的字符串花费的时间要长。
- BITFIELD把位图第一个字节偏移量0上的二进制位看作是most significant位，以此类推。举个例子，如果我们对一个已经预先被全部设置为0的位图进行设置，将它在偏移量7的值设置为5位无符号整数值23（二进制位为10111），那么命令将生产出如下这个位图表示`00000001 01110000`。当偏移量和整数长度与字节边界进行对齐时，BITFIELD表示二进制位的方式跟大端表示法一致，但是在没有对齐的情况下，理解这些二进制位是如何进行排列也是非常重要的。
- BITFIELD命令的返回值是一个数组，数组中的每个元素对应一个被执行的子命令。需要注意的是，OVERFLOW子命令本身并不产生任何回复。

## 持久化

-----

**SAVE**

- SAVE
- SAVE命令执行一个同步保存操作，将当前Redis实例的所有数据快照以RDB文件的形式保存到硬盘。
- 一般来说，在生产环境很少执行SAVE操作，因为它会阻塞所有客户端，保存数据库的任务通常由BGSAVE命令异步地执行。然而，如果负责保存数据的后台子进程不幸出现问题时，SAVE可以作为保存数据的最后手段来使用。
- 保存成功时返回OK。

-----

**BGSAVE**

- BGSAVE
- 在后台异步保存当前数据库的数据到磁盘。
- BGSAVE命令执行之后立即返回OK，然后Redis fork出一个新子进程，原来的Redis进程继续处理客户端请求，而子进程则负责将数据保存到磁盘，然后退出。
- 客户端可以通过LASTSAVE命令查看相关信息，判断BGSAVE命令是否执行成功。

-----

**BGREWRITEAOF**

- BGREWRITEAOF
- 执行一个AOF文件重写操作。重写会创建一个当前AOF文件的体积优化版本。
- 即使BGREWRITEAOF执行失败，也不会有任何数据丢失，因为旧的AOF文件在BGREWRITEAOF成功之前不会被修改。
- 重写操作只会在没有其他持久化工作在后台执行时被触发，也就是说：
    - 如果Redis的子进程正在执行快照的保存工作，那么AOF重写的操作会被预定，等到保存工作完成之后再执行AOF重写。在这种情况下，BGREWRITEAOF的返回值仍然是OK，但还会加上一条额外的信息，说明BGREWRITEAOF要等到保存操作完成之后才能执行。在Redis2.6或以上的版本，可以使用INFO[section]命令查看BGREWRITEAOF是否被预定。
    - 如果已经有别的AOF文件重写在执行，那么BGREWRITEAOF返回一个错误，并且这个新的BGREWRITEAOF请求也不会被预定到下次执行。
- 从Redis2.4开始，AOF重写由Redis自行出发，BGREWRITEAOF仅仅用于手动触发重写操作。

-----

**LASTSAVE**

- LASTSAVE
- 返回最近一次Redis成功将数据保存到磁盘上的时间，以UNIX时间戳格式表示。

## 发布与订阅

-----

**PUBLISH**

- PUBLISH channel message
- 将信息message发送到指定的频道channel。
- 返回接收到信息message的订阅者数量。

-----

**SUBSCRIBE**

- SUBSCRIBE channel [channel …]
- 订阅给定的一个或多个频道的信息。
- 返回接收到的信息，前面是订阅频道成功与否的情况，后面是订阅频道返回的信息，频道顺序按输入顺序。
```shell
# 订阅 msg 和 chat_room 两个频道

# 1 - 6 行是执行 subscribe 之后的反馈信息
# 第 7 - 9 行才是接收到的第一条信息
# 第 10 - 12 行是第二条

redis> subscribe msg chat_room
Reading messages... (press Ctrl-C to quit)
1) "subscribe"       # 返回值的类型：显示订阅成功
2) "msg"             # 订阅的频道名字
3) (integer) 1       # 目前已订阅的频道数量

1) "subscribe"
2) "chat_room"
3) (integer) 2

1) "message"         # 返回值的类型：信息
2) "msg"             # 来源(从那个频道发送过来)
3) "hello moto"      # 信息内容

1) "message"
2) "chat_room"
3) "testing...haha"
```

-----

**PSUBSCRIBE**

- PSUBSCRIBE pattern [pattern …]
- 订阅一个或多个符合给定模式的频道。
- 每个模式以*作为匹配符。
- 返回接收到的信息，先返回订阅频道成功与否的情况，对应模式，后返回订阅频道返回的信息。
```shell
# 订阅 news.* 和 tweet.* 两个模式

# 第 1 - 6 行是执行 psubscribe 之后的反馈信息
# 第 7 - 10 才是接收到的第一条信息
# 第 11 - 14 是第二条
# 以此类推。。。

redis> psubscribe news.* tweet.*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"                  # 返回值的类型：显示订阅成功
2) "news.*"                      # 订阅的模式
3) (integer) 1                   # 目前已订阅的模式的数量

1) "psubscribe"
2) "tweet.*"
3) (integer) 2

1) "pmessage"                    # 返回值的类型：信息
2) "news.*"                      # 信息匹配的模式
3) "news.it"                     # 信息本身的目标频道
4) "Google buy Motorola"         # 信息的内容

1) "pmessage"
2) "tweet.*"
3) "tweet.huangz"
4) "hello"

1) "pmessage"
2) "tweet.*"
3) "tweet.joe"
4) "@huangz morning"

1) "pmessage"
2) "news.*"
3) "news.life"
4) "An apple a day, keep doctors away"
```

-----

**UNSUBSCRIBE**

- UNSUBSCRIBE [channel [channel …]]
- 提示客户端退订给定的频道。
- 如果没有频道被指定，也即是，一个无参数的UNSUBSCRIBE调用被执行，那么客户端使用SUBSCRIBE命令订阅的所有频道都会被退订。在这种情况下，命令会返回一个信息，告知客户端所有被退订的频道。

-----

**PUNSUBSCRIBE**

- PUNSUBSCRIBE [pattern [pattern …]]
- 指示客户端退订所有给定模式。
- 如果没有模式被指定，也即是，一个无参数的PUNSUBSCRIBE调用被执行，那么客户端使用PSUBSCRIBE命令订阅的所有模式都会被退订。在这种情况下，命令会返回一个信息，告知客户端所有被退订的模式。

-----

**PUBSUB**

- PUBSUB `<subcommand>` [argument [argument …]]
- PUBSUB是一个查看订阅与发布系统状态的内省命令，它由数个不同格式的子命令组成，以下将分别对这些子命令进行介绍。
    - PUBSUB CHANNELS [pattern]
        - 列出当前的活跃频道。
        - 活跃频道指的是那些至少有一个订阅者的频道，订阅模式的客户端不计算在内。
        - pattern参数是可选的：
            - 如果不给出pattern参数，那么列出订阅与发布系统中的所有活跃频道。
            - 如果给出pattern参数，那么只列出和给定模式pattern相匹配的那些活跃频道。
        - 返回一个由活跃频道组成的列表。
        ```shell
        client-3> PUBSUB CHANNELS
        1) "news.sport"
        2) "news.internet"
        3) "news.it"

        # 接下来， client-3 打印那些与模式 news.i* 相匹配的活跃频道
        # 因为 news.sport 不匹配 news.i* ，所以它没有被打印

        redis> PUBSUB CHANNELS news.i*
        1) "news.internet"
        2) "news.it"    
        ```
    - PUBSUB NUMSUB [channel-1 … channel-N]
        - 返回给定频道的订阅者数量，订阅模式的客户端不计算在内。
        - 返回一个多条批量回复，回复中包含给定的频道，以及频道的订阅者数量。格式为频道channel-1，channel-1的订阅者数量，频道channel-2，以此类推。回复中频道的排列顺序和执行命令时给定频道的排列顺序一致。不给定任何频道而直接调用这个命令也是可以的，在这种情况下，命令只返回一个空列表。
        ```shell
        client-3> PUBSUB NUMSUB news.it news.internet news.sport news.music
        1) "news.it"    # 频道
        2) "2"          # 订阅该频道的客户端数量
        3) "news.internet"
        4) "1"
        5) "news.sport"
        6) "1"
        7) "news.music" # 没有任何订阅者
        8) "0"
        ```
    - PUBSUB NUMPAT
        - 返回订阅模式的数量
        - 注意，这个命令返回的不是订阅模式的客户端的数量，而是客户端订阅的所有模式的数量总和。
        ```shell
        client-3> PUBSUB NUMPAT
        (integer) 4
        ```

## 复制

-----

**SLAVEOF**

- SLAVEOF host port
- SLAVEOF命令用于在Redis运行时动态地修改复制功能的行为。
- 通过执行SLAVEOF host port命令，可以将当前服务器转变为指定服务器的从属服务器。
- 如果当前服务器已经是某个主服务器的从属服务器，那么执行SLAVEOF host port将使当前服务器停止对旧主服务器的同步，丢弃旧数据集，转而开始对新主服务器进行同步。
- 另外，对一个从属服务器执行命令SLAVEOF NO ONE将使得这个从属服务器关闭复制功能，并从从属服务器转变回主服务器，原来同步所得的数据集不会被丢弃。
- 利用SLAVEOF NO ONE不会丢弃同步所得数据集这个特性，可以在主服务器失败的时候，将从属服务器用作新的主服务器，从而实现无间断运行。
- 总是返回OK。

-----

**ROLE**

- ROLE
- 返回实例在复制中担任的角色，这个角色可以是master、slave或者sentinel。除了角色之外，命令还会返回与该角色相关的其他信息，其中：
    - 主服务器将返回属下从服务器的IP地址和端口。
    - 从服务器将返回自己正在复制的主服务器的IP地址、端口、连接状态以及复制偏移量。
    - Sentinel将返回自己正在监视的主服务器列表。
- ROLE命令将返回一个数组。

## 客户端与服务器

-----

**AUTH**

- AUTH password
- 通过设置配置文件中requirepass项的值（使用命令`CONFIG SET requirepass password`），可以使用密码来保护Redis服务器。
- 如果开启了密码保护的话，在每次连接Redis服务器之后，就要使用AUTH命令解锁，解锁之后才能使用其他Redis命令。
- 如果AUTH命令给定的密码password和配置文件中的密码相符的话，服务器会返回OK并开始接收命令输入。
- 另一方面，加入密码不匹配的话，服务器将返回一个错误，并要求客户端需重新输入密码。

-----

**QUIT**

- QUIT
- 请求服务器关闭与当前客户端的连接。
- 一旦所有等待中的回复（如果有的话）顺利写入到客户端，连接就会被关闭。
- 总是返回OK（但是不会被打印显示，因为当时redis-cli已经退出）。

-----

**INFO**

- INFO [section]
- 以一种易于解释且易于阅读的格式，返回关于Redis服务器的各种信息和统计数值。
- 通过给定可选参数section，可以让命令只返回某一部分的信息：
    - server部分记录了Redis服务器的信息，它包含以下域：
        - redis_version：Redis服务器版本
        - redis_git_sha1：Git SHA1
        - redis_git_dirty：Git dirty flag
        - os：Redis服务器的宿主操作系统
        - arch_bits：架构（32或64位）
        - multiplexing_api：Redis所使用的事件处理机制
        - gcc_version：编译Redis时所使用的GCC版本
        - process_id：服务器进程的PID
        - run_id：Redis服务器的随机标识符（用于Sentinel和集群）
        - tcp_port：TCP/IP监听端口
        - uptime_in_seconds：自Redis服务器启动以来，经过的秒数
        - uptime_in_days：自Redis服务器启动以来，经过的天数
        - lru_clock：以分钟为单位进行自增的时钟，用于LRU管理
    - clients部分记录了已连接客户端的信息，它包含以下域：
        - connected_clients：已连接客户端的数量（不包括通过从属服务器连接的客户端）
        - client_longest_output_list：当前连接的客户端当中，最长的输出列表
        - client_longest_input_buf：当前连接的客户端当中，最大输入缓存
        - blocked_clients：正在等待阻塞命令（BLPOP、BRPOP、BRPOPLPUSH）的客户端的数量
    - memory部分记录了服务器的内存信息，它包含以下域：
        - used_memory：由Redis分配器分配的内存总量，以字节（byte）为单位
        - used_memory_human：以人类可读的格式返回 Redis 分配的内存总量
        - used_memory_rss：从操作系统的角度，返回Redis已分配的内存总量（俗称常驻集大小）。这个值和top、ps等命令的输出一致。
        - used_memory_peak：Redis的内存消耗峰值（以字节为单位）
        - used_memory_peak_human：以人类可读的格式返回Redis的内存消耗峰值
        - used_memory_lua：Lua引擎所使用的内存大小（以字节为单位）
        - mem_fragmentation_ratio：used_memory_rss和 used_memory之间的比率
        - mem_allocator：在编译时指定的，Redis所使用的内存分配器。可以是libc、jemalloc或者tcmalloc
        - 在理想情况下，used_memory_rss的值应该只比used_memory稍微高一点儿。当rss>used，且两者的值相差较大时，表示存在（内部或外部的）内存碎片。内存碎片的比率可以通过mem_fragmentation_ratio的值看出。当used>rss时，表示Redis的部分内存被操作系统换出到交换空间了，在这种情况下，操作可能会产生明显的延迟。当Redis释放内存时，分配器可能会，也可能不会，将内存返还给操作系统。如果Redis释放了内存，却没有将内存返还给操作系统，那么used_memory的值可能和操作系统显示的Redis内存占用并不一致。查看used_memory_peak的值可以验证这种情况是否发生。
    - persistence部分记录了跟RDB持久化和AOF持久化有关的信息，它包含以下域：
        - loading：一个标志值，记录了服务器是否正在载入持久化文件。
        - rdb_changes_since_last_save：距离最近一次成功创建持久化文件之后，经过了多少秒。
        - rdb_bgsave_in_progress：一个标志值，记录了服务器是否正在创建RDB文件。
        - rdb_last_save_time：最近一次成功创建RDB文件的UNIX时间戳。
        - rdb_last_bgsave_status：一个标志值，记录了最近一次创建RDB文件的结果是成功还是失败。
        - rdb_last_bgsave_time_sec：记录了最近一次创建RDB文件耗费的秒数。
        - rdb_current_bgsave_time_sec：如果服务器正在创建RDB文件，那么这个域记录的就是当前的创建操作已经耗费的秒数。
        - aof_enabled：一个标志值，记录了AOF是否处于打开状态。
        - aof_rewrite_in_progress： 一个标志值，记录了服务器是否正在创建AOF文件。
        - aof_rewrite_scheduled：一个标志值，记录了在RDB文件创建完毕之后，是否需要执行预约的AOF重写操作。
        - aof_last_rewrite_time_sec：最近一次创建AOF文件耗费的时长。
        - aof_current_rewrite_time_sec：如果服务器正在创建AOF文件，那么这个域记录的就是当前的创建操作已经耗费的秒数。
        - aof_last_bgrewrite_status：一个标志值，记录了最近一次创建AOF文件的结果是成功还是失败。
        - 如果AOF持久化功能处于开启状态，那么这个部分还会加上以下域：
            - aof_current_size：AOF文件目前的大小。
            - aof_base_size：服务器启动时或者AOF重写最近一次执行之后，AOF文件的大小。
            - aof_pending_rewrite：一个标志值，记录了是否有AOF重写操作在等待RDB文件创建完毕之后执行。
            - aof_buffer_length：AOF缓冲区的大小。
            - aof_rewrite_buffer_length：AOF重写缓冲区的大小。
            - aof_pending_bio_fsync：后台I/O队列里面，等待执行的fsync调用数量。
            - aof_delayed_fsync：被延迟的fsync调用数量。
    - stats部分记录了一般统计信息，它包含以下域：
        - total_connections_received：服务器已接受的连接请求数量。
        - total_commands_processed：服务器已执行的命令数量。
        - instantaneous_ops_per_sec：服务器每秒钟执行的命令数量。
        - rejected_connections：因为最大客户端数量限制而被拒绝的连接请求数量。
        - expired_keys：因为过期而被自动删除的数据库键数量。
        - evicted_keys：因为最大内存容量限制而被驱逐（evict）的键数量。
        - keyspace_hits：查找数据库键成功的次数。
        - keyspace_misses：查找数据库键失败的次数。
        - pubsub_channels：目前被订阅的频道数量。
        - pubsub_patterns：目前被订阅的模式数量。
        - latest_fork_usec：最近一次fork()操作耗费的毫秒数。
    - replication记录了主从复制信息。
        - role：如果当前服务器没有在复制任何其他服务器，那么这个域的值就是master；否则的话，这个域的值就是slave。注意，在创建复制链的时候，一个从服务器也可能是另一个服务器的主服务器。
        - 如果当前服务器是一个从服务器的话，那么这个部分还会加上以下域：
            - master_host：主服务器的IP地址。
            - master_port：主服务器的TCP监听端口号。
            - master_link_status：复制连接当前的状态，up表示连接正常，down表示连接断开。
            - master_last_io_seconds_ago：距离最近一次与主服务器进行通信已经过去了多少秒钟。
            - master_sync_in_progress：一个标志值，记录了主服务器是否正在与这个从服务器进行同步。
        - 如果同步操作正在进行，那么这个部分还会加上以下域：
            - master_sync_left_bytes：距离同步完成还缺少多少字节数据。
            - master_sync_last_io_seconds_ago：距离最近一次因为SYNC操作而进行I/O已经过去了多少秒。
        - 如果主从服务器之间的连接处于断线状态，那么这个部分还会加上以下域：
            - master_link_down_since_seconds：主从服务器连接断开了多少秒。
        - 以下是一些总会出现的域：
            - connected_slaves：已连接的从服务器数量。
        - 对于每个从服务器，都会添加以下一行信息：
            - slaveXXX：ID、IP 地址、端口号、连接状态
    - cpu部分记录了CPU的计算量统计信息，它包含以下域：
        - used_cpu_sys：Redis服务器耗费的系统CPU。
        - used_cpu_user：Redis服务器耗费的用户CPU。
        - used_cpu_sys_children：后台进程耗费的系统CPU。
        - used_cpu_user_children：后台进程耗费的用户CPU。
    - commandstats部分记录了各种不同类型的命令的执行统计信息，比如命令执行的次数、命令耗费的CPU时间、执行每个命令耗费的平均CPU时间等等。对于每种类型的命令，这个部分都会添加一行以下格式的信息：
        - cmdstat_XXX:calls=XXX,usec=XXX,usecpercall=XXX
    - cluster部分记录了和集群有关的信息，它包含以下域：
        - cluster_enabled：一个标志值，记录集群功能是否已经开启。
    - keyspace部分记录了数据库相关的统计信息，比如数据库的键数量、数据库已经被删除的过期键数量等。对于每个数据库，这个部分都会添加一行以下格式的信息：
        - dbXXX:keys=XXX,expires=XXX
    - 除上面给出的这些值以外， section 参数的值还可以是下面这两个：
        - all：返回所有信息
        - default：返回默认选择的信息
    - 当不带参数直接调用INFO命令时，使用default作为默认参数。
    ```shell
    redis> INFO
    # Server
    redis_version:2.9.11
    redis_git_sha1:937384d0
    redis_git_dirty:0
    redis_build_id:8e9509442863f22
    redis_mode:standalone
    os:Linux 3.13.0-35-generic x86_64
    arch_bits:64
    multiplexing_api:epoll
    gcc_version:4.8.2
    process_id:4716
    run_id:26186aac3f2380aaee9eef21cc50aecd542d97dc
    tcp_port:6379
    uptime_in_seconds:362
    uptime_in_days:0
    hz:10
    lru_clock:1725349
    config_file:

    # Clients
    connected_clients:1
    client_longest_output_list:0
    client_biggest_input_buf:0
    blocked_clients:0

    # Memory
    used_memory:508536
    used_memory_human:496.62K
    used_memory_rss:7974912
    used_memory_peak:508536
    used_memory_peak_human:496.62K
    used_memory_lua:33792
    mem_fragmentation_ratio:15.68
    mem_allocator:jemalloc-3.2.0

    # Persistence
    loading:0
    rdb_changes_since_last_save:6
    rdb_bgsave_in_progress:0
    rdb_last_save_time:1411011131
    rdb_last_bgsave_status:ok
    rdb_last_bgsave_time_sec:-1
    rdb_current_bgsave_time_sec:-1
    aof_enabled:0
    aof_rewrite_in_progress:0
    aof_rewrite_scheduled:0
    aof_last_rewrite_time_sec:-1
    aof_current_rewrite_time_sec:-1
    aof_last_bgrewrite_status:ok
    aof_last_write_status:ok

    # Stats
    total_connections_received:2
    total_commands_processed:4
    instantaneous_ops_per_sec:0
    rejected_connections:0
    sync_full:0
    sync_partial_ok:0
    sync_partial_err:0
    expired_keys:0
    evicted_keys:0
    keyspace_hits:0
    keyspace_misses:0
    pubsub_channels:0
    pubsub_patterns:0
    latest_fork_usec:0
    migrate_cached_sockets:0

    # Replication
    role:master
    connected_slaves:0
    master_repl_offset:0
    repl_backlog_active:0
    repl_backlog_size:1048576
    repl_backlog_first_byte_offset:0
    repl_backlog_histlen:0

    # CPU
    used_cpu_sys:0.21
    used_cpu_user:0.17
    used_cpu_sys_children:0.00
    used_cpu_user_children:0.00

    # Cluster
    cluster_enabled:0

    # Keyspace
    db0:keys=2,expires=0,avg_ttl=0
    ```

-----

**SHUTDOWN**

- SHUTDOWN [SAVE|NOSAVE]
- SHUTDOWN命令执行以下操作：
    - 停止所有客户端
    - 如果有至少一个保存点在等待，执行SAVE命令
    - 如果AOF选项被打开，更新AOF文件
    - 关闭redis服务器
- 如果持久化被打开的话，SHUTDOWN命令会保证服务器正常关闭而不丢失任何数据，
- 另一个方面，假如只是单纯地执行SAVE命令，然后再执行QUIT命令，则没有这一保证——因为在执行SAVE之后，执行QUIT之前的这段时间中间，其他客户端可能正在和服务器进行通讯，这时如果执行QUIT就会造成数据丢失。
- 通过使用可选的修饰符，可以修改SHUTDOWN命令的表现。比如说：
    - 执行SHUTDOWN SAVE会强制让数据库执行保存操作，即使没有设定保存点。
    - 执行SHUTDOWN NOSAVE会阻止数据库执行保存操作，即使已经设定有一个或多个保存点。
- 执行失败时返回错误。执行成功时不返回任何信息，服务器和客户端的连接断开，客户端自动退出。

-----

**TIME**

- TIME
- 返回当前服务器时间。
- 一个包含两个字符串的列表：第一个字符串是当前时间（以UNIX时间戳格式表示10位，精确到秒），而第二个字符串是当前这一秒钟已经逝去的微秒数（6位数）。

-----

**CLIENT_GETNAME**

- CLIENT GETNAME
- 返回CLIENT SETNAME命令为连接设置的名字。
- 因为新创建的连接默认是没有名字的，对于没有名字的连接，CLIENT GETNAME返回空白回复。

-----

**CLIENT_SETNAME**

- CLIENT SETNAME connection-name
- 为当前连接分配一个名字。
- 这个名字会显示在CLIENT LIST命令的结果中，用于识别当前正在与服务器进行连接的客户端。
- 举个例子，在使用Redis构建队列时，可以根据连接负责的任务，为信息生产者和信息消费者分别设置不同的名字。
- 名字使用Redis的字符串类型来保存，最大可以占用512MB。另外，为了避免和CLIENT LIST命令的输出格式发生冲突，名字里不允许使用空格。
- 要移除一个连接的名字，可以将连接的名字设为空字符串""。
- 使用CLIENT GETNAME命令可以取出连接的名字。
- 新创建的连接默认是没有名字的。
- 设置成功时返回OK。

-----

**CLIENT_LIST**

- CLIENT LIST
- 以人类可读的格式，返回所有连接到服务器的客户端信息和统计数据。
- 命令返回多行字符串，这些字符串按以下形式被格式化：
    - 每个已连接客户端对应一行（以LF分割）
    - 每行字符串由一系列`属性=值`形式的域组成，每个域之间以空格分开
- 以下是域的含义：
    - addr：客户端的地址和端口
    - fd：套接字所使用的文件描述符
    - age：以秒计算的已连接时长
    - idle：以秒计算的空闲时长
    - flags：客户端flag（见下文）
    - db：该客户端正在使用的数据库ID
    - sub：已订阅频道的数量
    - psub：已订阅模式的数量
    - multi：在事务中被执行的命令数量
    - qbuf：查询缓冲区的长度（字节为单位，0表示没有分配查询缓冲区）
    - qbuf-free：查询缓冲区剩余空间的长度（字节为单位，0表示没有剩余空间）
    - obl：输出缓冲区的长度（字节为单位，0表示没有分配输出缓冲区）
    - oll：输出列表包含的对象数量（当输出缓冲区没有剩余空间时，命令回复会以字符串对象的形式被入队到这个队列里）
    - omem：输出缓冲区和输出列表占用的内存总量
    - events：文件描述符事件（见下文）
    - cmd：最近一次执行的命令
- 客户端flag可以由以下部分组成：
    - O：客户端是MONITOR模式下的附属节点（slave）
    - S：客户端是一般模式下（normal）的附属节点
    - M：客户端是主节点（master）
    - x：客户端正在执行事务
    - b：客户端正在等待阻塞事件
    - i：客户端正在等待VM I/O操作（已废弃）
    - d：一个受监视（watched）的键已被修改，EXEC命令将失败
    - c：在将回复完整地写出之后，关闭链接
    - u：客户端未被阻塞（unblocked）
    - A：尽可能快地关闭连接
    - N：未设置任何flag
- 文件描述符events事件可以是：
    - r：客户端套接字（在事件loop中）是可读的（readable）
    - w：客户端套接字（在事件loop中）是可写的（writeable）
```shell
redis> CLIENT LIST
addr=127.0.0.1:43143 fd=6 age=183 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
addr=127.0.0.1:43163 fd=5 age=35 idle=15 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=ping
addr=127.0.0.1:43167 fd=7 age=24 idle=6 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
```

-----

**CLIENT_KILL**

- CLIENT KILL ip:port
- 关闭地址为ip:port的客户端。
- ip:port应该和CLIENT LIST命令输出的其中一行匹配。
- 因为Redis使用单线程设计，所以当Redis正在执行命令的时候，不会有客户端被断开连接。
- 如果要被断开连接的客户端正在执行命令，那么当这个命令执行之后，在发送下一个命令的时候，它就会收到一个网络错误，告知它自身的连接已被关闭。
- 当指定的客户端存在，且被成功关闭时，返回OK。

## 配置选项

-----

**CONFIG_SET**

- CONFIG SET parameter value
- CONFIG SET命令可以动态地调整Redis服务器的配置而无须重启。
- 你可以使用它修改配置参数，或者改变Redis的持久化方式。
- CONFIG SET可以修改的配置参数可以使用命令CONFIG GET *来列出，所有被CONFIG SET修改的配置参数都会立即生效。
- 当设置成功时返回OK，否则返回一个错误。

-----

**CONFIG_GET**

- CONFIG GET parameter
- CONFIG GET命令用于取得运行中的Redis服务器的配置参数。
- CONFIG GET接受单个参数parameter作为搜索关键字，查找所有匹配的配置参数，其中参数和值以"键-值对"的方式排列。
- 比如执行CONFIG GET s*命令，服务器就会返回所有以s开头的配置参数及参数的值：
    ```shell
    redis> CONFIG GET s*
    1) "save"                       # 参数名：save
    2) "900 1 300 10 60 10000"      # save 参数的值
    3) "slave-serve-stale-data"     # 参数名： slave-serve-stale-data
    4) "yes"                        # slave-serve-stale-data 参数的值
    5) "set-max-intset-entries"     # ...
    6) "512"
    7) "slowlog-log-slower-than"
    8) "1000"
    9) "slowlog-max-len"
    10) "1000"
    ```
- 如果你只是寻找特定的某个参数的话，你当然也可以直接指定参数的名字。
- 使用命令CONFIG GET *，可以列出CONFIG GET命令支持的所有参数。
- 所有被CONFIG SET所支持的配置参数都可以在配置文件redis.conf中找到，不过CONFIG GET和CONFIG SET使用的格式和redis.conf文件所使用的格式由以下两点不同：
    - 10kb、2gb这些在配置文件中所使用的储存单位缩写，不可以用在CONFIG命令中，CONFIG SET的值只能通过数字值显式地设定。
    - save选项在redis.conf中是用多行文字储存的，但在CONFIG GET命令中，它只打印一行文字。
        ```shell
        #以下是 save 选项在 redis.conf 文件中的表示：
        save 900 1
        save 300 10
        save 60 10000

        #但是 CONFIG GET 命令的输出只有一行：
        redis> CONFIG GET save
        1) "save"
        2) "900 1 300 10 60 10000"
        ```
        - 上面save参数的三个值表示：在900秒内最少有1个key被改动，或者300秒内最少有10个key被改动，又或者60秒内最少有1000个key被改动，以上三个条件随便满足一个，就触发一次保存操作。
- 返回给定配置参数的值。

-----

**CONFIG_RESETSTAT**

- CONFIG RESETSTAT
- 重置INFO命令中的某些统计数据，包括：
    - Keyspace hits (键空间命中次数)
    - Keyspace misses (键空间不命中次数)
    - Number of commands processed (执行命令的次数)
    - Number of connections received (连接服务器的次数)
    - Number of expired keys (过期key的数量)
    - Number of rejected connections (被拒绝的连接数量)
    - Latest fork(2) time(最后执行 fork(2) 的时间)
    - The aof_delayed_fsync counter(aof_delayed_fsync 计数器的值)
- 总是返回OK。

-----

**CONFIG_REWRITE**

- CONFIG REWRITE
- CONFIG REWRITE命令对启动Redis服务器时所指定的redis.conf文件进行改写：因为CONFIG_SET命令可以对服务器的当前配置进行修改，而修改后的配置可能和redis.conf文件中所描述的配置不一样，CONFIG REWRITE的作用就是通过尽可能少的修改，将服务器当前所使用的配置记录到redis.conf文件中。
- 重写会以非常保守的方式进行：
    - 原有redis.conf文件的整体结构和注释会被尽可能地保留。
    - 如果一个选项已经存在于原有redis.conf文件中，那么对该选项的重写会在选项原本所在的位置（行号）上进行。
    - 如果一个选项不存在于原有redis.conf文件中，并且该选项被设置为默认值，那么重写程序不会将这个选项添加到重写后的redis.conf文件中。
    - 如果一个选项不存在于原有redis.conf文件中，并且该选项被设置为非默认值，那么这个选项将被添加到重写后的redis.conf文件的末尾。
    - 未使用的行会被留白。比如说，如果你在原有redis.conf文件上设置了数个关于save选项的参数，但现在你将这些save参数的一个或全部都关闭了，那么这些不再使用的参数原本所在的行就会变成空白的。
- 即使启动服务器时所指定的redis.conf文件已经不再存在，CONFIG REWRITE命令也可以重新构建并生成出一个新的redis.conf文件。
- 另一方面，如果启动服务器时没有载入redis.conf文件，那么执行CONFIG REWRITE命令将引发一个错误。
- 对redis.conf文件的重写是原子性的，并且是一致的：如果重写出错或重写期间服务器崩溃，那么重写失败，原有redis.conf文件不会被修改。如果重写成功，那么redis.conf文件为重写后的新文件。
- 返回一个状态值：如果配置重写成功则返回OK，失败则返回一个错误。

## 调试

-----

**PING**

- PING
- 如果连接正常就返回一个PONG，否则返回一个连接错误。
- 通常用于测试与服务器的连接是否仍然生效，或者用于测量延迟值。

-----

**ECHO**

- ECHO message
- 打印一个特定的信息message，测试时使用。
- 返回message自身。

-----

**OBJECT**

- OBJECT subcommand [arguments [arguments]]
- OBJECT命令允许从内部察看给定key的Redis对象，它通常用在除错(debugging)或者了解为了节省空间而对key使用特殊编码的情况。当将Redis用作缓存程序时，你也可以通过OBJECT命令中的信息，决定key的驱逐策略(eviction policies)。
- OBJECT命令有多个子命令：
    - OBJECT REFCOUNT <key> 返回给定key引用所储存的值的次数。此命令主要用于除错。
    - OBJECT ENCODING <key> 返回给定key锁储存的值所使用的内部表示(representation)。
    - OBJECT IDLETIME <key> 返回给定key自储存以来的空闲时间(idle， 没有被读取也没有被写入)，以秒为单位。
- 对象可以以多种方式编码：
    - 字符串可以被编码为raw(一般字符串)或int(为了节约内存，Redis会将字符串表示的64位有符号整数编码为整数来进行储存）。
    - 列表可以被编码为ziplist或linkedlist。ziplist是为节约大小较小的列表空间而作的特殊表示。
    - 集合可以被编码为intset或者hashtable。intset是只储存数字的小集合的特殊表示。
    - 哈希表可以编码为zipmap或者hashtable。zipmap是小哈希表的特殊表示。
    - 有序集合可以被编码为ziplist或者skiplist格式。ziplist用于表示小的有序集合，而skiplist则用于表示任何大小的有序集合。
- 假如你做了什么让Redis没办法再使用节省空间的编码时（比如将一个只有1个元素的集合扩展为一个有100万个元素的集合），特殊编码类型会自动转换成通用类型。
- REFCOUNT和IDLETIME返回数字。ENCODING返回相应的编码类型。

-----

**SLOWLOG**

- SLOWLOG subcommand [argument]
- Slow log是Redis用来记录查询执行时间的日志系统。
- 查询执行时间指的是不包括像客户端响应，发送回复等IO操作，而单单是执行一个查询命令所耗费的时间。
- 另外，slow log保存在内存里面，读写速度非常快，因此你可以放心地使用它，不必担心因为开启slow log而损害Redis的速度。
- slow log的行为由两个配置参数指定，可以通过改写redis.conf文件或者用CONFIG GET和CONFIG SET命令对它们动态地进行修改。
- 第一个选项是slowlog-log-slower-than，它决定要对执行时间大于多少微秒的查询进行记录。
- 另一个选项是slowlog-max-len，它决定slow log最多能保存多少条日志，slow log本身是一个FIFO队列，当队列大小超过slowlog-max-len时，最旧的一条日志将被删除，而最新的一条日志加入到slow log，以此类推。
- 要查看slow log，可以使用SLOWLOG GET或者SLOWLOG GET number命令，前者打印所有slow log，后者只打印指定数量的日志，最新的日志会最先被打印。
```shell
redis> SLOWLOG GET
1) 1) (integer) 12                      # 唯一性(unique)的日志标识符
   2) (integer) 1324097834              # 被记录命令的执行时间点，以 UNIX 时间戳格式表示
   3) (integer) 16                      # 查询执行时间，以微秒为单位
   4) 1) "CONFIG"                       # 执行的命令，以数组的形式排列
      2) "GET"                          # 这里完整的命令是 CONFIG GET slowlog-log-slower-than
      3) "slowlog-log-slower-than"

2) 1) (integer) 11
   2) (integer) 1324097825
   3) (integer) 42
   4) 1) "CONFIG"
      2) "GET"
      3) "*"

3) 1) (integer) 10
   2) (integer) 1324097820
   3) (integer) 11
   4) 1) "CONFIG"
      2) "GET"
      3) "slowlog-log-slower-than"

# ...
```
- 日志的唯一 id 只有在 Redis 服务器重启的时候才会重置，这样可以避免对日志的重复处理(比如你可能会想在每次发现新的慢查询时发邮件通知你)。
- 使用命令SLOWLOG LEN可以查看当前日志的数量。
- 使用命令SLOWLOG RESET可以清空slow log。

-----

**MONITOR**

- MONITOR
- 实时打印出Redis服务器接收到的命令，调试用。
- 总是返回OK。

-----

**DEBUG_OBJECT**

- DEBUG OBJECT key
- DEBUG OBJECT是一个调试命令，它不应被客户端所使用，详见OBJECT命令。
- 当key存在时，返回有关信息。当key不存在时，返回一个错误。

-----

**DEBUG SEGFAULT**

- DEBUG SEGFAULT
- 执行一个不合法的内存访问从而让Redis崩溃，仅在开发时用于BUG模拟。
- 无返回值。

## 内部命令

-----

**MIGRATE**

- 将key原子性地从当前实例传送到目标实例的指定数据库上，一旦传送成功，key保证会出现在目标实例上，而当前实例上的key会被删除。
- 这个命令是一个原子操作，它在执行的时候会阻塞进行迁移的两个实例，直到以下任意结果发生：迁移成功，迁移失败，等待超时。
- 命令的内部实现是这样的：它在当前实例对给定key执行DUMP命令 ，将它序列化，然后传送到目标实例，目标实例再使用RESTORE对数据进行反序列化，并将反序列化所得的数据添加到数据库中；当前实例就像目标实例的客户端那样，只要看到RESTORE命令返回OK，它就会调用DEL删除自己数据库上的key。
- timeout参数以毫秒为格式，指定当前实例和目标实例进行沟通的最大间隔时间。这说明操作并不一定要在timeout毫秒内完成，只是说数据传送的时间不能超过这个timeout数。
- MIGRATE命令需要在给定的时间规定内完成IO操作。如果在传送数据时发生IO错误，或者达到了超时时间，那么命令会停止执行，并返回一个特殊的错误：IOERR。
- 当IOERR出现时，有以下两种可能：
    - key可能存在于两个实例
    - key可能只存在于当前实例
- 唯一不可能发生的情况就是丢失key，因此，如果一个客户端执行MIGRATE命令，并且不幸遇上IOERR错误，那么这个客户端唯一要做的就是检查自己数据库上的key是否已经被正确地删除。
- 如果有其他错误发生，那么MIGRATE保证key只会出现在当前实例中。（当然，目标实例的给定数据库上可能有和key同名的键，不过这和MIGRATE命令没有关系）。
- 可选项：
    - COPY：不移除源实例上的key。
    - REPLACE：替换目标实例上已存在的key。
- 迁移成功时返回OK，否则返回相应的错误。

-----

**DUMP**

- DUMP key
- 序列化给定key，并返回被序列化的值，使用RESTORE命令可以将这个值反序列化为Redis键。
- 序列化生成的值有以下几个特点：
    - 它带有64位的校验和，用于检测错误，RESTORE在进行反序列化之前会先检查校验和。
    - 值的编码格式和RDB文件保持一致。
    - RDB版本会被编码在序列化值当中，如果因为Redis的版本不同造成RDB格式不兼容，那么Redis会拒绝对这个值进行反序列化操作。
- 序列化的值不包括任何生存时间信息。
- 如果key不存在，那么返回nil。否则，返回序列化之后的值。

-----

**RESTORE**

- RESTORE key ttl serialized-value [REPLACE]
- 反序列化给定的序列化值，并将它和给定的key关联。
- 参数ttl以毫秒为单位为key设置生存时间；如果ttl为0，那么不设置生存时间。
- RESTORE在执行反序列化之前会先对序列化值的RDB版本和数据校验和进行检查，如果RDB版本不相同或者数据不完整的话，那么RESTORE会拒绝进行反序列化，并返回一个错误。
- 如果键key已经存在，并且给定了REPLACE选项，那么使用反序列化得出的值来代替键key原有的值；相反地，如果键key已经存在，但是没有给定REPLACE选项，那么命令返回一个错误。
- 如果反序列化成功那么返回OK，否则返回一个错误。

-----

**SYNC**

- SYNC
- 用于复制功能的内部命令。
- 返回序列化数据。

-----

**PSYNC**

- PSYNC master_run_id offset
- 用于复制功能的内部命令。
- 返回序列化数据。