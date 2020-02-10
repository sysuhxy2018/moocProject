# DB设计

总设计图：

<img src="./screenshots/chapter2/schema.png" style="zoom:50%;margin-left:0;" />

除了在MySQL中添加表格外，还要在package中添加对应实体的类。以用户信息为例：

### tb_person_info

``` mysql
use o2o;

create table tb_person_info (
	user_id int(10) not null auto_increment,
    name varchar(32) default null,
    profile_img varchar(1024) default null,
    email_img varchar(1024) default null,
    gender varchar(2) default null,
    enable_status int(2) not null default 0 comment "0:禁用, 1:可用",
    user_type int(2) not null default 1 comment "1:顾客, 2:店家, 3:超管",
    create_time datetime default null,
    last_edit_time datetime default null,
    primary key(user_id)
) engine = innodb auto_increment = 1 default charset = utf8mb4;
```

comment相当于备注，在创建后信息是会一直保留的，并且可以查看到，如下图。utf8mb4相比utf8(utf8mb3)扩充了更多字符，如emoji表情等。

<img src="./screenshots/chapter2/comment.png" style="margin-left:0;" />

### Personinfo.java

``` java
package com.xinyuan.o2o.entity;

import java.util.Date;

public class PersonInfo {

	private Long userId;
	
	private String name;
	
	private String profileImg;
	
	private String email;
	
	private String gender;
	
	private Integer enableStatus;
	
	private Integer userType;	// 1.顾客 2.店家 3.超管
	
	private Date createTime;
	
	private Date lastEditTime;

	// ... 这部分可以通过eclipse自动构建setter和getter方法。
}

```

这里用Integer而不是int，主要是考虑到数据库方面可能有null值，这里用包装类的话也可以表示null值，保持一致。









