# 测试流程管理平台

## 一、数据结构
```sql
CREATE DATABASE `test_management`;

CREATE TABLE `user`(
  `id` bigint(11) unsigned p NULL AUTO_INCREMENT COMMENT '自增id',
  `user_id` bigint(20) unsigned NOT NULL DEFAULT '0' COMMENT '用户id',
  `username` varchar(16) NOT NULL DEFAULT '' COMMENT '用户名',
  `password` varchar(256) NOT NULL DEFAULT '' COMMENT '密码: md5(password+salt)',
  `salt` varchar(32) NOT NULL DEFAULT '' COMMENT '盐',
  `name` varchar(16) NOT NULL DEFAULT '' COMMENT '姓名',
  `phone` varchar(16) NOT NULL DEFAULT '' COMMENT '手机号',
  `department` varchar(64) NOT NULL DEFAULT '' COMMENT '所属部门',
  `portrait_url` varchar(256) DEFAULT '' COMMENT '头像url地址',
  `role` smallint(4) NOT NULL DEFAULT '0' COMMENT '0:未使用 1:系统管理员 2:被测单位负责人 3:测评单位负责人 4:测评单位测试人员 5:地方路局 6:铁路总局',
  `create_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uniq_user_id` (`user_id`),
  UNIQUE KEY `uniq_username` (`username`),
  KEY `idx_phone` (`phone`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='用户表';

CREATE TABLE `test_project`(
  `id` bigint(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增id',
  `project_id` bigint(20) unsigned NOT NULL DEFAULT '0' COMMENT '项目id',
  `name` varchar(16) NOT NULL DEFAULT '' COMMENT '项目名称',
  `test_leader_id` bigint(20) unsigned NOT NULL DEFAULT '0' COMMENT '测试单位负责人id',
  `under_test_leader_id` bigint(20) unsigned NOT NULL DEFAULT '0' COMMENT '被测单位负责人id',
  `status` smallint(4) NOT NULL DEFAULT '0' COMMENT '项目进展状态 0:未使用 1:定级材料审核中 2:定级材料审核完成 3:定级材料审核失败 4:备案中 5:备案成功 6:备案失败 7:测试待确认 8:测试中 9:驳回整改 10:测试通过 11:项目取消',
  `project_location_code` smallint(4) NOT NULL DEFAULT '0' COMMENT '保留字段，用于记录位置信息，负责项目分发给不同负责人',
  `project_location` varchar(64) NOT NULL DEFAULT '' COMMENT '项目所在位置信息',
  `rank` varchar(16) NOT NULL DEFAULT '' COMMENT '项目定级 如 S1,A2,G1',
  `type` smallint(4) NOT NULL DEFAULT 0 COMMENT '项目测试类型 0:未使用 1:等保测试 2:风险评估测试 待补充...',
  `create_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uniq_project_id` (`project_id`),
  KEY `idx_test_leader_id` (`test_leader_id`),
  KEY `idx_under_test_leader_id` (`under_test_leader_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='提测项目表';

CREATE TABLE `project_tester_relation`(
  `id` bigint(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增id',
  `relation_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '项目和测试者关系id',
  `project_id` bigint(20) unsigned NOT NULL DEFAULT '0' COMMENT '项目id',
  `tester_id` bigint(20) unsigned NOT NULL DEFAULT '0' COMMENT '测试者id',
  `create_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE
  CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_relation_id` (`relation_id`),
  KEY `idx_project_id` (`project_id`),
  KEY `idx_tester_id` (`tester_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='项目与具体测试者关系表';

CREATE TABLE `project_material`(
  `id` bigint(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增id',
  `material_id` bigint(20) unsigned NOT NULL DEFAULT '0' COMMENT '提测项目子任务id',
  `project_id` bigint(20) unsigned NOT NULL DEFAULT '0' COMMENT '项目id',
  `committer_id` bigint(20) unsigned NOT NULL DEFAULT '0' COMMENT '提交者id',
  `type` smallint(4) NOT NULL DEFAULT '0' COMMENT '子任务类型 0:未使用 1:备案资料 2:公安部整改意见 3:定级审核材料 4:测评报告 5:整改方案',
  `status` smallint(4) NOT NULL DEFAULT '0' COMMENT '参照test_project表中的status字段，表示在哪个阶段提交的材料',
  `version` smallint(4) NOT NULL DEFAULT '0' COMMENT '某类材料的提交版本号',
  `remark` varchar(256) NOT NULL DEFAULT '' COMMENT '提交材料时的一些备注文字信息',
  `file_url` varchar(512) NOT NULL DEFAULT '' COMMENT '附件url地址',
  `content` mediumtext NOT NULL COMMENT '存储可能的json字符串(含有整改意见)',
  `create_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uniq_material_id` (`material_id`),
  KEY `idx_committer_id` (`committer_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='提测项目子任务材料表';

CREATE TABLE `standard_library`(
  `id` bigint(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增id',
  `standard_id` bigint(20) unsigned NOT NULL DEFAULT '0' COMMENT '测试评定标准id',
  `standard_rank` smallint(4) NOT NULL DEFAULT '0' COMMENT '代表此条记录是第几级的 0:未使用 1:一级 2:二级 3:三级即具体标准',
  `headline` varchar(16) NOT NULL DEFAULT '' COMMENT '一级标题',
  `secondary_headline` varchar(16) NOT NULL DEFAULT '' COMMENT '二级标题',
  `name` varchar(16) NOT NULL DEFAULT '' COMMENT '标准库检测项名称',
  `headline_rank` smallint(4) NOT NULL DEFAULT '0' COMMENT '一级标题排序序号',
  `secondary_headline_rank` smallint(4) NOT NULL DEFAULT '0' COMMENT '二级标题排序序号',
  `name_rank` smallint(4) NOT NULL DEFAULT '0' COMMENT '检测项排序序号',
  `rank` varchar(8) NOT NULL DEFAULT '' COMMENT '标准等级 S1-3、A1-3、G1-3',
  `content` text NOT NULL COMMENT '检测内容',
  `create_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uniq_standard_id` (`standard_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='标准库表';
```

## 二、交互流程
1. 内置一个系统管理员，管理员录入标准库记录，需要录入检测项名称、检测项一级标题、二级标题、一级标题序号、二级标题序号以及该条记录对应的等级(SAG与123混合)
2. 管理员可以分配不同用户权限的用户初始账号
3. 被测单位负责人进行项目创建，选择测试类型(如等保、安全测试),定级(SAG选择),如果有定级审核材料，应新建一条project_material记录，并修改test_project中的status(支持多文件?)，提交后修改test_project中的状态至定级材料审核中
4. 提交后，通过短信提醒测试单位负责人，通知其进行定级材料审核，如果未通过审核，修改test_project表中的status，并短信通知被测单位负责人，让其重新提交定级材料；如果审核通过，修改数据库状态，同样通知被测单位负责人进度，测试单位负责人负责分配该项目的具体测试人员，因为可能有多个测试人员，所以每有一个建立一条project_tester_relation记录
5. 测试人员可以看到一个他负责的所有项目的列表
6. 测试人员点击某个项目，进入项目的详情页面，根据项目的定级，从数据库中整理出一个模板。测试用户在页面上，进行打分，并对某项给出整改意见，然后统一返回给后端json，存储在project_material中的content字段
7. 测试人员可随时下载模板或者修改之后的报告
8. 测试人员在测试完成之后，可以选择通过和不通过。如果审核通过，则可以直接打印，盖章；如果审核不通过，则短信通知，被测试单位负责人，让其整改并上传整改方案。上传整改方案之后跳转到第四步，循环。

## 三、模板定义
{"安全通用要求":
    {"安全物理环境":
       {"物理访问控制":
         {"content":"机房出入口应安排专人值守或配置电子门禁系统，控制、鉴                    别和记录进入的人员。",
          "mark":"5",
          "opinion":"希望可以整改"},
       "防盗窃和防破坏":{"content":"机房出入口应安排专人值守或配置电子门禁系统，控制、鉴别和记录进入的人员。",
          "mark":"5",
          "opinion":"希望可以整改"}},
     "安全通信网络":
       {"通信传输": {"content":"机房出入口应安排专人值守或配置电子门禁系统，控制、鉴别和记录进入的人员。",
          "mark":"5",
          "opinion":"希望可以整改"},
        "可信验证": {"content":"机房出入口应安排专人值守或配置电子门禁系统，控制、鉴别和记录进入的人员。",
          "mark":"5",
          "opinion":"希望可以整改"}}
},
"云计算安全扩展要求":
    {"安全物理环境":
       {"基础设施位置":{"content":"机房出入口应安排专人值守或配置电子门禁系统，控制、鉴别和记录进入的人员。",
          "mark":"5",
          "opinion":"希望可以整改"}},
     "安全通信网络":
       {"网络架构": {"content":"机房出入口应安排专人值守或配置电子门禁系统，控制、鉴别和记录进入的人员。",
          "mark":"5",
          "opinion":"希望可以整改"}},
     "安全区域边界":
       {"访问控制": {"content":"机房出入口应安排专人值守或配置电子门禁系统，控制、鉴别和记录进入的人员。",
          "mark":"5",
          "opinion":"希望可以整改"}}
}
}

## 四、接口定义

## 接口通用字段定义

### 通用请求参数
以下接口中，请求参数中只会给出data中的对象定义。

| 字段名称 | 字段类型 | 取值示例 | 备注 |
| -- | -- | -- | -- |
| data | object | {} | 具体返回值的json对象 |
| user_profile| object | {} | 用户的个人信息资料 |
| request_time| int | 1561552025099| 请求时间戳 |
| token | String |  | 防止重复登录的token凭证 |

| userProfile字段名称 | 字段类型 | 取值示例 | 备注 |
| -- | -- | -- | -- |
| user_id | Long ||| 用户id |
| username | String ||| 用户名 |
| phone| String ||| 该用户的手机号 |

### 通用响应参数
以下接口中，响应参数中只会给出data中的对象定义。

| 字段名称 | 字段类型 | 取值示例 | 备注 |
| -- | -- | -- | -- |
| data | object | {} | 具体返回值的json对象 |
| err_no | int | 10100000000| 长度固定为11位 |
| err_msg| String |  | 错误描述信息 |
| response_time | int | | 响应时间戳|

## 接口列表

### 1.登录
url: v1/user/login

request

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| username| String(16) | Y |  | 用户名 长度小于等于16 |
| password| String | Y | | 对应公钥加密过的密码 |

请求json示例：

```
{
	"user_profile": {
	},
	"data": {
		"username": "gaofeng",
		"password": "HoLAT150YPtUj1UEnRofXLTpx8cmy5hMbjstt0HeMFCL011rIOP/ihcMTNVCD335VNIgpKAsMZPxhVKPsfIUOvTH9S2QdanhWZu1BAy7m2O+AFmWa6rKpCuCi7+6lNhRGsihdRFzAx024UJNBHTmc2TYN8rd43usONFmSZQQwik="
	},
	"request_time": 1563032147000,
	"token": ""
}
```

response

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| token | String | Y |  | 该用户对应的登录凭证，每次登录会改变 |

响应json示例：

```
{
    "err_no": 0,
    "err_msg": "success",
    "response_time": 1563032428574,
    "data": {
        "token": "fAttNJoPOV4ogNV6zTdwIpwN94GFvjBqdHBHDj29LJuqfevWwWs1BYWG52ExGTHW1MA6WqOAScKKkgNu8wxFHA%3D%3D"
    }
}
```


错误号定义

| 错误号 | 错误信息 |
|-------|---------|
| 101121021 | 参数校验错误 |
| 101121022 | 服务器未捕获异常 |
| 101131841 | 用户名不存在 |
| 101131849 | 密码错误 |

### 2.分配用户账号角色
url: v1/user/assignUser

request

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| username| String(16) | Y |  | 用户名 长度小于等于16 |
| password| String | Y | | 对应公钥加密过的密码 |
| name | String | Y | | 该用户的真实姓名 |
| phone| String | Y | | 该用户经过合法性校验的手机号 |
| department | String | Y | | 该用户所属部门 |
| role | int | Y | | 该用户对应的角色 1:系统管理员 2:被测单位负责人 3:测评单位负责人 4:测评单位测试人员 5:地方路局 6:铁路总局|

请求json示例：

```
{
	"user_profile": {
		"user_id": 1562861191461431,
		"username": "administrator",
		"phone": "17801020888"
	},
	"data": {
		"username": "qijiazhen",
		"password": "dad4h0UTOtOUKsCeLyo0vTWuWA1QBqcTX1lWGNX+mzrZJABCzZa3XZO+guQX9DgUZlTEf82zuKSewOM7KxSXUQ4hIlPnmwQsfzA+8XWPwAXv75SywSe8WXPoo+qXHfkQYTrA1PeG/Myrx5Eae8KeAu1xuGm3bupgrj8eZwFDxJQ=",
		"name": "祁家祯",
		"phone": "17801020777",
		"department": "开发部门",
		"role": 6
	},
	"request_time": 1563004345000,
	"token": "fAttNJoPOV4ogNV6zTdwIpwN94GFvjBqdHBHDj29LJuqfevWwWs1BYWG52ExGTHW1MA6WqOAScKKkgNu8wxFHA%3D%3D"
}
```

response

null

响应json示例：

```
{
    "err_no": 0,
    "err_msg": "success",
    "response_time": 1563005026331,
    "data": null
}
```


错误号定义：

| 错误号 | 错误信息 |
|-------|---------|
| 101121021 | 参数校验错误 |
| 101121022 | 服务器未捕获异常 |
| 101090910 | 用户身份错误，非管理员不能分配用户角色 |
| 101090918| 用户名已经被注册过 |
| 101090920| 手机号已经被注册过 |
| 101110127 | 数据库操作错误 |

### 3.获取所有测评单位测试人员

url: v1/user/getUsersByRole

request

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| role | int | Y || 0:未使用 1:系统管理员 2:被测单位负责人 3:测评单位负责人 4:测评单位测试人员 5:地方路局 6:铁路总局 |

请求json示例：

```
{
	"user_profile": {
	},
	"data": {
		"role": 6
	},
	"request_time": 1563032147000,
	"token": ""
}
```

response

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| users | list<User> | Y | | 用户列表 |

| User字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| user_id| Long | Y || 用户id|
| name | String | Y || 姓名 |
| phone| String | Y || 手机号 |
| department| String| Y || 所属部门 |
| portrait_url | String | N || 头像url地址|

响应json示例：

```
{
    "err_no": 0,
    "err_msg": "success",
    "response_time": 1563035635246,
    "data": [
        {
            "user_id": 1562861299770577,
            "name": "高峰",
            "phone": "17801020789",
            "department": "开发部门",
            "portrait_url": ""
        },
        {
            "user_id": 1563004369420350,
            "name": "祁家祯",
            "phone": "17801020777",
            "department": "开发部门",
            "portrait_url": ""
        },
        {
            "user_id": 1563005026326963,
            "name": "龙灏天",
            "phone": "17801020666",
            "department": "开发部门",
            "portrait_url": ""
        },
        {
            "user_id": 1563005296255749,
            "name": "张三",
            "phone": "17801020555",
            "department": "开发部门",
            "portrait_url": ""
        }
    ]
}
```

错误号定义:

| 错误号 | 错误信息 |
|-------|---------|
| 101121021 | 参数校验错误 |
| 101121022 | 服务器未捕获异常 |

### 4.录入标准库记录

url: v1/standard/add

request

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| standard_rank | int | Y | | 代表此条记录是第几级的 0:未使用 1:一级 2:二级 3:三级即具体标准|
| headline | String| Y | | 标准的一级标题 例如:6.2 云计算安全扩展要求 则传'云计算安全扩展要求' |
| headline_rank| int | Y | 1 | 一级标题对应的序号  例如: 6.2.1 安全物理环境 则传6|
| secondary_headline | String | N | | 标准的二级标题 例如: 6.2.1 安全物理环境 则传'安全物理环境'|
| secondary_headline_rank | int | N | | 二级标题对应的序号: 例如: 6.2.1 安全物理环境 则传2|
| name | String | N | | 标准库检测项名称 例如: 6.2.1.1 基础设施位置 则传'基础设施位置'|
| name_rank | int | N | | 检测项的序号 例如: 6.2.3.1 访问控制 则传1|
| rank | String | N || 标准等级 S1-3、A1-3、G1-3 |
| content | String | N || 具体标准的内容 |

请求json示例：

```
{
	"user_profile": {
		"user_id": 1562861299770577,
		"username": "gaofeng",
		"phone": "17801020789"
	},
	"data": {
		"standard_rank": 3,
		"headline": "安全通用要求",
		"headline_rank": 1,
		"secondary_headline": "安全物理环境",
		"secondary_headline_rank": 1,
		"name": "物理访问控制",
		"name_rank": "1",
		"rank": "S1",
		"content": "机房出入口应安排专人值守或配置电子门禁系统，控制、鉴别和记录进入的人员。"
	},
	"request_time": 1563004345000,
	"token": "fAttNJoPOV4ogNV6zTdwIpwN94GFvjBqdHBHDj29LJuqfevWwWs1BYWG52ExGTHW1MA6WqOAScKKkgNu8wxFHA%3D%3D"
}
```

response

null

响应json示例：

```
{
    "err_no": 0,
    "err_msg": "success",
    "response_time": 1563273060256,
    "data": null
}
```

错误号定义:

| 错误号 | 错误信息 |
|-------|---------|
| 101121021 | 参数校验错误 |
| 101121022 | 服务器未捕获异常 |
| 101161801| 数据库指定位置已经存在一条记录 |
| 101161814 | 数据库插入错误 |

### 5.按要求获取标准库记录

url: v1/standard/list

request

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| standard_rank | int | Y || 表示获取几级标题 1:一级 2:二级 3:三级即具体标准 |
| headline_rank| int | N | 1 | 一级标题对应的序号  例如: 6.2.1 安全物理环境 则传6|
| secondary_headline_rank | int | N | | 二级标题对应的序号: 例如: 6.2.1 安全物理环境 则传2|

请求json示例：

```
{
	"user_profile": {
		"user_id": 1562861299770577,
		"username": "gaofeng",
		"phone": "17801020789"
	},
	"data": {
		"standard_rank": 3,
		"headline_rank": 1,
		"secondary_headline_rank": 1,
	},
	"request_time": 1563004345000,
	"token": "fAttNJoPOV4ogNV6zTdwIpwN94GFvjBqdHBHDj29LJuqfevWwWs1BYWG52ExGTHW1MA6WqOAScKKkgNu8wxFHA%3D%3D"
}
```

response

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| headline | String| N | | 标准的一级标题 例如:6.2 云计算安全扩展要求 则传'云计算安全扩展要求' |
| headline_rank| int | N | 1 | 一级标题对应的序号  例如: 6.2.1 安全物理环境 则传6|
| secondary_headline | String | N | | 标准的二级标题 例如: 6.2.1 安全物理环境 则传'安全物理环境'|
| secondary_headline_rank | int | N | | 二级标题对应的序号: 例如: 6.2.1 安全物理环境 则传2|
| name | String | N | | 标准库检测项名称 例如: 6.2.1.1 基础设施位置 则传'基础设施位置'|
| name_rank | int | N | | 检测项的序号 例如: 6.2.3.1 访问控制 则传1|
| rank | String | N || 标准等级 S1-3、A1-3、G1-3 |
| content | String | N || 具体标准的内容 |

响应json示例：

```
{
    "err_no": 0,
    "err_msg": "success",
    "response_time": 1563383424543,
    "data": [
        {
            "headline": "安全通用要求",
            "headline_rank": 1,
            "secondary_headline": "安全物理环境",
            "secondary_headline_rank": 1,
            "name": "物理访问控制",
            "name_rank": 1,
            "rank": "S1",
            "content": "机房出入口应安排专人值守或配置电子门禁系统，控制、鉴别和记录进入的人员。"
        }
    ]
}
```

错误号定义:

| 错误号 | 错误信息 |
|-------|---------|
| 101121021 | 参数校验错误 |
| 101121022 | 服务器未捕获异常 |
| 101180042 | 记录等级应为1-3 |
| 101180043 | 一级标题，headlineRank、SecondaryHeadlineRank都应为0 |
| 101180044 | 二级标题，headlineRank不应为0，SecondaryHeadlineRank应为0 |
| 101180044 | 一级标题，headlineRank、SecondaryHeadlineRank都不应为0 |

### 6.测试项目创建

url: v1/project/create

request

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| name | String | Y |  | 项目名称 |
| project_location | String | Y || 项目所在位置信息|
| rank | String | Y || 项目定级 如 S1,A2,G1|
| type | int | Y || 项目测试类型 1:等保测试 2:风险评估测试 待补充...|

请求json示例：

```
{
	"user_profile": {
		"user_id": 1562861299770577,
		"username": "gaofeng",
		"phone": "17801020789"
	},
	"data": {
		"name": "测试项目1",
		"project_location": "北京交通大学",
		"rank": "S1,A2,G1",
		"type": 1
	},
	"request_time": 1563004345000,
	"token": "fAttNJoPOV4ogNV6zTdwIpwN94GFvjBqdHBHDj29LJuqfevWwWs1BYWG52ExGTHW1MA6WqOAScKKkgNu8wxFHA%3D%3D"
}
```

response

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| project_id | int | Y | | 项目id|

响应json示例：

```
{
    "err_no": 0,
    "err_msg": "success",
    "response_time": 1563520153540,
    "data": {
        "project_id": 1563520153538800
    }
}
```

错误号定义:

| 错误号 | 错误信息 |
|-------|---------|
| 101121021 | 参数校验错误 |
| 101121022 | 服务器未捕获异常 |
| 101182058 | 项目名称已存在 |
| 101190152 | 暂无可调度的测试单位负责人 |

### 7.修改项目状态

url: v1/project/updateStatus

request

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| project_id | int | Y | | 项目id|
| status| int | Y || 待更新的状态 |

请求json示例：

```
{
	"user_profile": {
		"user_id": 1562861299770577,
		"username": "gaofeng",
		"phone": "17801020789"
	},
	"data": {
		"project_id": 1563520153538800,
		"status": 2
	},
	"request_time": 1563004345000,
	"token": "fAttNJoPOV4ogNV6zTdwIpwN94GFvjBqdHBHDj29LJuqfevWwWs1BYWG52ExGTHW1MA6WqOAScKKkgNu8wxFHA%3D%3D"
}
```

response

null

响应json示例:

```
{
    "err_no": 0,
    "err_msg": "success",
    "response_time": 1563527347993,
    "data": null
}
```

错误号定义:

| 错误号 | 错误信息 |
|-------|---------|
| 101121021 | 参数校验错误 |
| 101121022 | 服务器未捕获异常 |
| 101191548 | 未知状态 |
| 101191552 | 不存在该项目 |
| 101191637 | 状态不可达或者数据库错误 |

### 8.分配项目的测试人员

url: v1/project/assignTester

request

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| project_id| int| Y || 项目id |
| testers| list<int>| Y || 被分配的测试人的用户ID(可以单个也可以多个,按list传输)|

请求json示例：

```
{
	"user_profile": {
		"user_id": 1562861299770577,
		"username": "gaofeng",
		"phone": "17801020789"
	},
	"data": {
		"project_id": 1563520153538800,
		"testers": [1563005296255749, 1563471692050997]
	},
	"request_time": 1563004345000,
	"token": "fAttNJoPOV4ogNV6zTdwIpwN94GFvjBqdHBHDj29LJuqfevWwWs1BYWG52ExGTHW1MA6WqOAScKKkgNu8wxFHA%3D%3D"
}
```

response

null

响应json示例：

```
{
    "err_no": 0,
    "err_msg": "success",
    "response_time": 1563543180143,
    "data": null
}
```


错误号定义:

| 错误号 | 错误信息 |
|-------|---------|
| 101121021 | 参数校验错误 |
| 101121022 | 服务器未捕获异常 |
| 101192103 | 不存在该项目 |
| 101192110 | 传来的用户ID并不都是测试人员 |
| 101192124 | 数据库记录创建失败|

### 9.拉取某个用户所负责的所有项目列表

url: v1/project/list

request

null

请求json示例：

response

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| projects| list<Project> | Y || 所负责项目列表 |

| Project字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| project_id | int | Y || 项目id |
| test_leader_name| String | Y || 测试单位负责人姓名 |
| under_test_leader_name| String | Y || 被测单位负责人姓名 |
| status | int | Y || 项目进展状态 |
| project_location | String | Y || 项目所在位置信息 |
| rank | String | Y || 项目定级 如 S1,A2,G1 |
| type | int | Y || 项目测试类型 1:等保测试 2:风险评估测试 待补充... |

响应json示例：

错误号定义:

### 10.拉取某个项目的详情

url: v1/project/list

request

null

请求json示例：

response

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| projects| list<Project> | Y || 所负责项目列表 |

| Project字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| project_id | int | Y || 项目id |
| test_leader_name| String | Y || 测试单位负责人姓名 |
| under_test_leader_name| String | Y || 被测单位负责人姓名 |
| status | int | Y || 项目进展状态 |
| project_location | String | Y || 项目所在位置信息 |
| rank | String | Y || 项目定级 如 S1,A2,G1 |
| type | int | Y || 项目测试类型 1:等保测试 2:风险评估测试 待补充... |

响应json示例：

错误号定义:

### 11.获取某个项目对应的模板信息

url: v1/project/template

request

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| project_id | int| Y || 项目id |


请求json示例：

response

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| material_id | int| Y || 提测项目子任务id |
| content | String | Y || 模板的json信息 |

响应json示例：

错误号定义:

### 12.上传项目子任务材料

url: v1/material/upload

request

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| project_id | int | Y | | 项目id|
| type | int | Y | | 子任务类型 1:备案资料 2:公安部整改意见 3:定级审核材料 4:测评报告 5:整改方案 |
| remark | String | N | | 对项目以及材料的文字备注信息 |
| file_url | String | N || 如果有附件，则上传附件的文件地址 |
| content | String | N || 测评报告以及整改方案的json内容 |



请求json示例：

response

null

响应json示例：

错误号定义:

### 13.拉取审核材料列表

url: v1/material/list

request

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| project_id | int | Y | | 项目id|

请求json示例：

response

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| materials | list<Material> | Y | | 项目材料列表 |

| Material字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| material_id | int | Y | | 项目材料id |
| type | int | Y || 子任务类型 1:备案资料 2:公安部整改意见 3:定级审核材料 4:测评报告 5:整改方案 |
| status | int| Y || 子任务对应的状态 |
| remark | String | N || 提交材料时的一些备注文字信息|
| file_url| String | N || 附件url地址 |
| content | String | N || 存储可能的json字符串(含有整改意见) |

响应json示例：

错误号定义:

### 14.创建提测项目子任务材料

url: v1/material/update

request

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| project_id | int| Y || 项目id |
| material_id | int | Y || 子任务材料id |
| type | int | Y || 子任务类型 1:备案资料 2:公安部整改意见 3:定级审核材料 4:测评报告 5:整改方案|
| remark | String | N || 提交材料时的一些备注文字信息 |
| file_url | String | O | | 附件url地址 |
| content | String | O | | 存储可能的json字符串(含有整改意见) |

请求json示例：

response

null

响应json示例：

错误号定义:

### 15.文件上传

url: v1/file/upload

request

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| upload_file | MultipartFile | Y || 上传的文件 |

请求json示例：

response

| 字段名称 | 字段类型 | 是否必传 | 取值示例 | 备注 |
| -- | -- | -- | -- | -- |
| file_uri | String | Y || 文件在后台的静态资源uri |

响应json示例：

错误号定义:


## 五、问题
1. 各个状态间都是串行的吗？是否有并行关系？
2. 一次操作可以上传多个附件吗？以防万一把project_material中的file_url字段设置的长一些
3. 需要对接发送短信api，计划使用腾讯云，因为不需要公司资质，只需要注册一个公众号，短信签名通常与公众号名称相同最容易过审。所以有如下任务：1.申请公众号 2.申请短信签名 3.申请短信模板
4. 加密部署问题更换jre lib文件 https://www.hellojava.com/a/44108.html
5. 不要使用 java.util.base64 使用org.apache.commons.codec.binary.Base64

## 项目依赖工具
1. mysql 8.0.11
2. redis 4.0.10
