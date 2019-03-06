[TOC]

# 前言

在etl清洗流程中，前一步处理将数据清洗到hive中，hive某字段中落有json字符串。我需要将json中的嵌套json解析为单独的字段。

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0l5ac9t7dj319a0r4wgo.jpg)

解析为id_no、name、mobile、apply_id... extend_params_ip 、extend_params_ip_user_number、extend_params_contact（直接存储数组）...

重新解析到hive外部表，实际落入到hbase

但是之前落入hive的人未处理异常json的情况如上图，所以，我的处理手段是将json中的异常字符如`\"` 替换为`"`。故事就这样开始了



# 1. 在hue中测试处理json

参考 [转义字符反斜杠\\(在hive+shell以及java中注意事项)](https://www.bbsmax.com/A/MyJxeX625n/)

可以知道：

> 转义字符的特殊情况，自身的转义，比如java有时候需要两个转义字符`\\`，或者四个转义字符`\\\\`。
> 1)java的两种情况
>
> - 正则表达式匹配
>
> - string的split函数
>
>   这两种情况中字符串包含转义字符"\"时，需要先对转义字符自身转义，就是说需要两个转义字符"\\"。（java解析后，再有正则和split自身特定进行解析）
>
>   而当匹配字符正斜线`\`，则需要四个转义字符`\\\\`，因为，首先java（编译器？）自身先解析，转义成`\\`（两个`\`），再由正则或split的解析功能转义成一个`\`，才是最终要处理的字符。
>
>   这是因为解析过程需要两次，才能在字符串中出现正斜线"\"，出现后才能转义后面的字符。
>
> 2)hive中的split和正则表达式
>
> ​	hive用java写的，所以同Java一样，两种情况也需要两个"\\"。如果是正则的话或者split的话，要匹配`\` 需要4个`\`



> ### 如果hive和shell和正则再相遇，那么将是三次解析
>
>    shell语言也有转义字符，自身直接处理。 而hive语句在shell脚本中执行时，就需要先由shell转义后，再由hive处理。这个过程又造成二次转义。 所以，注意hive语句在shell脚本执行时，转义字符需要翻倍。hive处理的是shell转义后的语句，必须转以后正确，才能执行。如果是一个`\`,则需要6个\ : `\\\\\\`，



本次用到函数regexp_replace第二参数本身就是一个正则，那么hive+正则，就是两次转义，对于反斜杠`\`,就是`\\\\`,对于`"`是`\\"`

hue先测试一条数据

```sql
hive<<!

set mapreduce.job.queuename=root.dw;
set hive.support.concurrency=false;

create external table if not exists ods.ods_r1_apply_risk_params_ex(row_key string,id_no string,name string,mobile string,apply_id string,apply_code string,apply_source string,apply_type string,apply_times string,user_id string,extend_params_ip string,extend_params_ip_user_number string,extend_params_contact string)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' 
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,params:id_no,params:name,params:mobile,params:apply_id,params:apply_code,params:apply_source,params:apply_type,params:apply_times,params:user_id,params:extend_params_ip,params:extend_params_ip_user_number,params:extend_params_contact") 
TBLPROPERTIES("hbase.table.name" = "ods_r1_apply_risk_params_hb");


insert overwrite table ods.ods_r1_apply_risk_params_ex
select
      a.row_key,
      b.id_no,
      b.name,
      b.mobile,
      b.apply_id,
      b.apply_code,
      b.apply_source,
      b.apply_type,
      b.apply_times,
      b.user_id,
      extend_params_ip,
      extend_params_ip_user_number,
      extend_params_contact
from (select ods_r1_apply_risk_params.idapply_risk_params_id as row_key,ods_r1_apply_risk_params.apply_risk_params_apply_params from ods.ods_r1_apply_risk_params where ods_r1_apply_risk_params.dt=20180101 limit 1
) a lateral view default.json_tuple2(a.apply_risk_params_apply_params,'id_no','name','mobile','apply_id','apply_code','apply_source','apply_type','apply_times','user_id','extend_params') 
 b  as id_no,name,mobile,apply_id,apply_code,apply_source,apply_type,apply_times,user_id,extend_params lateral view default.json_tuple2( regexp_replace(regexp_replace(b.extend_params,'\\\\\"','"'),'\\\\\\\\"'),'ip','ip_user_number','contact') c as extend_params_ip,extend_params_ip_user_number,extend_params_contact
;
!

```

得到结果

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0l6mji7ezj31kw03qgll.jpg)



当时没有细看，就直接写入到脚本开始调度

谁曾想

# 2. 命令行调用hive

所以直接将hue的脚本命令拷入脚本执行，结果无一正确结果，返回来看，发现会经历shell首先会解析一次，所以增加一个斜杠用8个代替

```sql
select
      regexp_replace(regexp_replace(b.extend_params,'\\\\\\\\\\"','"'),'\\\\\\\\\\\\\\\\\\"','"') ee,
      b.extend_params ep,
      a.row_key,
      b.id_no,
      b.name,
      b.mobile,
      b.apply_id,
      b.apply_code,
      b.apply_source,
      b.apply_type,
      b.apply_times,
      b.user_id,
      extend_params_ip,
      extend_params_ip_user_number,
      extend_params_contact
from (select ods_r1_apply_risk_params.idapply_risk_params_id as row_key,ods_r1_apply_risk_params.apply_risk_params_apply_params from ods.ods_r1_apply_risk_params where idapply_risk_params_id=20180101 limit 1
) a lateral view default.json_tuple2(a.apply_risk_params_apply_params,'id_no','name','mobile','apply_id','apply_code','apply_source','apply_type','apply_times','user_id','extend_params') 
 b  as id_no,name,mobile,apply_id,apply_code,apply_source,apply_type,apply_times,user_id,extend_params lateral view default.json_tuple2( regexp_replace(regexp_replace(b.extend_params,'\\\\\\\\\\"','"'),'\\\\\\\\\\\\\\\\\\"','"'),'ip','ip_user_number','contact') c as extend_params_ip,extend_params_ip_user_number,extend_params_contact
;
```

shell执行的时候，就可以看到解析了一次

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0l7gbo4itj32ye0og4qp.jpg)

转义字符就少了一半，第一次就是双反斜杠变为单反斜杠，最后打完收工

show的最原始 

```
'\\\\\\\\"','"'),'\\\\\\\\\\\\"','"
```

