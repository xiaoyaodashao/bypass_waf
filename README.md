# xwaf
### waf自动暴破
	xwaf是一个python写的waf自动绕过工具
		https://github.com/3xp10it/bypass_waf/blob/master/xwaf.py
	上一个版本bypass_waf.py文件可参考下面链接
		https://github.com/3xp10it/bypass_waf/blob/master/bypass_waf.py
	xwaf相比上个版本更智能,可无人干预,自动暴破waf
代码流程图:
[127.0.0.1/1.php?id=1为例]

1>start
2>检测系统/usr/share/sqlmap/127.0.0.1/log文件是否存在
3>获取log文件:
	如果不存在log文件则调用get_log_file_need_tamper函数,执行完这个函数后获得log文件,也即成功检测出目标url有sqli
	注入漏洞,如果执行完get_log_file_need_tamper函数没有获得log文件则认为该url没有sqli漏洞
4>获取db_type[数据库类型]
	调用get_db_type_need_tamper函数,用于后面的tamper排列组合时,只将目标url对应的数据库类型的tamper用于该目标在sql注入时tamper的选择后的组合
5>获取sqli_type[注入方法]
	调用get_good_sqli_type_need_tamper函数,sql注入方法中一共有U|S|E+B|Q|T 6种注入方法,后3种查询效率低,首先在log文件中查找是否有U|S|E这3种高效方法中的任意一种,如果
	有略过这一步,否则执行get_good_sqli_type_need_tamper函数,执行该函数将尝试获得一种以上的高效注入方法
6>获取current-db[当前数据库名]
	如果上面获得了高效注入方法,则先用高效注入方法获得current-db,如果没有则用B|Q|T方法尝试获得current-db,尝试获
	得current-db的函数是get_db_name_need_tamper
7>获取table[当前数据库的表名]
	如果上面获得了高效注入方法,则先用高效注入方法获得table,如果没有则用B|Q|T方法尝试获得table,尝试获得table的函数是
	get_table_name_need_tamper
8>获取column[当前数据库的第一个表的所有列名]
	如果上面获得了高效注入方法,则先用高效注入方法获得column,如果没有则用B|Q|T方法获得column,尝试获得column
	的函数是get_column_name_need_tamper
9>获取entries[column对应的真实数据]
	调用get_entries_need_tamper函数,执行完get_entries_need_tamper函数后,waf成功绕过,从上面的步骤一直到这个步骤,
	逐步获得最佳绕过waf的脚本组合

about:
1>xwaf支持记忆,运行中断后下次继续运行时会在中断时的最后一个命令附近继续跑,不会重新经历上面的所有函数的处理
2>各个get_xxx_need_tamper函数的处理采用针对当前url的数据库类型(eg.MySQL)的所有过waf的脚本(在sqlmap的tamper目录中)的排列组合的结果与--hex或--no-cast选项进行暴力破解
如果--hex起作用了则不再使用--no-cast尝试,--no-cast起作用了也不再用--hex尝试
3>need py3.5
4>usage:
python3 xwaf.py "http://127.0.0.1/1.php?id=1"
过程中如果被中断了,接着再运行相同命令即可从断点附近接着暴
5>xwaf运行完后将在/root/.sqlmap/output/127.0.0.1目录下的ini文件中看到相关信息,bypassed_command是成功暴破waf的
sqlmap语句
6>在tamper组合中,先用到的tamper会加入到上面的ini文件中,在以后的每个tamper组合中,综合已经得到的有用的tamper再组
合,在上面的ini文件中的tamper_list即为不断完善的tamper组合
