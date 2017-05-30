##SQL实现并集，交集，差集
###数据准备
	CREATE TABLE IF NOT EXIST tab1(
		x string,
		y string
	);
	
	CREATE TABLE IF NOT EXIST tab2(
		x string,
		y string
	);
	
	SELECT * FROM tab1;
	x y
	1 2
	2 3
	
	SELECT * FROM tab2;
	x y
	3 1
	2 3
	
###并集（tab1 U tab2）
tab1 FULL OUTTER JOIN tab2 ON condition：tab1，tab2连接，如果tab1和tab2的行满足condition条件，组合在一起。不满足条件，另外表为null。

如果tab1或tab2很多行与join的另一个表满足条件的行很多，整个操作会很慢。

	SELECT tab1.x,tab1.y,tab2.x,tab2.y
	FROM tab1
	FULL OUTTER JOIN tab2
	ON (tab1.x=tab2.x and tab1.y=tab2.y)
	
	result:
	tab1.x tab1.y tab2.x tab2.y
	1      2      null   null
	2      3      2      3
	null   null   3      1
	2      3      2      3

###差集（tab1 - tab2）
tab1 LEFT OUTTER JOIN tab2 on condition：将tab1的行左连接tab2（遍历tab1的行寻找tab2满足condition条件的行，将其附加在tab1的行后面，如果满足条件的tab2行是多个，tab1的该行将被复制多份一一附加tab2满足条件的行）

如果tab1待遍历的行多同时tab2满足要求的行也多，整个操作会非常慢。

	SELECT tab1.x,tab1.y,tab2.x,tab2.y
	FROM tab1
	LEFT OUTTER JOIN tab2
	ON (tab1.x=tab2.x and tab1.y=tab2.y)
	WHERE tab2.x is null;
	
	result:
	tab1.x tab1.y tab2.x tab2.y
	1      2      null   null
	
	
	
###交集（tab1 ^ tab2）
tab1 INNER JOIN tab2 ON condition：tab1，tab2连接，如果tab1和tab2的行满足condition条件，组合在一起。不满足条件不出现在返回集中。

	SELECT tab1.x,tab1.y,tab2.x,tab2.y
	FROM tab1
	INNER JOIN tab2
	ON (tab1.x=tab2.x and tab1.y=tab2.y)
	
	result:
	tab1.x tab1.y tab2.x tab2.y
	2      3      2      3
	
###参考：
<http://www.cnblogs.com/sunjie9606/p/4167190.html>
