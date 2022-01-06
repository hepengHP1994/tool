# seata 安装说明
## 开始

### 1.创建文件夹存放配置文件和docker-compose文件

```
mkdir /usr/local/docker/seata
mkdir /usr/local/docker/seata/config
```

### 2.注册seata到nacos需要在config文件夹中添加registry.conf
```
vi registry.conf

文件中的内容

registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  # 表明注册到nacos
  type = "nacos"

  nacos {
    # 服务名称
    application = "seata-server"
    # nacos的IP地址加端口号 如果是本机的话添加内网IP  docker inspect nacos容器ID 就可以获取
    serverAddr = ""
    # nacos 分组
    group = "SEATA_GROUP"
    # 命名空间，默认为空
    namespace = ""
    cluster = "default"
    # nacos 用户名
    username = ""
    # nacos 密码
    password = ""
  }

  file {
    name = "file.conf"
  }
}

config {
  # file、nacos 、apollo、zk、consul、etcd3
  # 表明去那里获取配置文件填nacos是第一种，填file为第二种
  type = "file"
  nacos {
    serverAddr = ""
    namespace = ""
    group = "SEATA_GROUP"
    username = ""
    password = ""
  }
  file {
    # 配置文件在容器的这个目录下
    name = "file:/root/seata-config/file.conf"
  }
}
```


### 3.指定持久化有两种方式

##### 1.从nacos中获取数据库连接信息

新建config.txt，修改事务分组，修改数据库连接信息
```
service.vgroupMapping.my_test_tx_group=default
store.mode=db
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.jdbc.Driver
store.db.url=jdbc:mysql://127.0.0.1:3306/seata?useUnicode=true
store.db.user=username
store.db.password=password
store.db.minConn=5
store.db.maxConn=30
store.db.globalTable=global_table
store.db.branchTable=branch_table
store.db.queryLimit=100
store.db.lockTable=lock_table
store.db.maxWait=5000
```
推送脚本 https://github.com/seata/seata/tree/1.4.0/script/config-center/nacos

##### 2.在config文件夹中添加file.conf 指明持久化方式

```
## transaction log store, only used in seata-server
store {
  ## 持久化方式 mode: file、db、redis
  mode = "db"
  ## file store property
  file {
    ## store location dir
    dir = "sessionStore"
    # branch session size , if exceeded first try compress lockkey, still exceeded throws exceptions
    maxBranchSessionSize = 16384
    # globe session size , if exceeded throws exceptions
    maxGlobalSessionSize = 512
    # file buffer size , if exceeded allocate new buffer
    fileWriteBufferCacheSize = 16384
    # when recover batch read size
    sessionReloadReadSize = 100
    # async, sync
    flushDiskMode = async
  }

  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
    datasource = "druid"
    ## mysql/oracle/postgresql/h2/oceanbase etc.
    dbType = "mysql"
    # msyql8 com.mysql.cj.jdbc.Driver
    driverClassName = "com.mysql.cj.jdbc.Driver"
    # db的IP地址 如果是本机的话添加内网IP  docker inspect db容器ID 就可以获取
    url = "jdbc:mysql://172.18.0.2:3306/seata"
    user = "root"
    password = "root"
    minConn = 5
    maxConn = 100
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
    maxWait = 5000
  }

  ## redis store property
  redis {
    host = "127.0.0.1"
    port = "6379"
    password = ""
    database = "0"
    minConn = 1
    maxConn = 10
    maxTotal = 100
    queryLimit = 100
  }
}
```

### 4.持久化方式为db后，就需要添加数据库了，数据库名称 为seata，跟上面url的一致就可以了
```sql

-- -------------------------------- The script used when storeMode is 'db' --------------------------------
-- the table to store GlobalSession data
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(96),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_branch_id` (`branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;
```

4.在seata文件夹中添加docker-compose文件

```yaml
version: "3.2"
services:
    # 分布式事务服务
    seata-server:
        image: seataio/seata-server:1.3.0
        container_name: seata-server
        ports:
            - "8091:8091"
        networks:
            - seata_net
        restart: on-failure
        privileged: true
        environment:
            #宿主机ip
            - SEATA_IP=192.168.88.143
            - SEATA_CONFIG_NAME=file:/root/seata-config/registry
            - SEATA_PORT=8091
            - STORE_MODE=db
        volumes:
            # 第5步编写的registry.conf
            - "./config:/root/seata-config"
            - "./logs:/root/logs/seata"
```


### 5.使用seata的的服务数据库都要加undo_log表
```sql
-- for AT mode you must to init this sql for you business database. the seata server not need it.
CREATE TABLE IF NOT EXISTS `undo_log`
(
`branch_id`     BIGINT(20)   NOT NULL COMMENT 'branch transaction id',
`xid`           VARCHAR(100) NOT NULL COMMENT 'global transaction id',
`context`       VARCHAR(128) NOT NULL COMMENT 'undo_log context,such as serialization',
`rollback_info` LONGBLOB     NOT NULL COMMENT 'rollback info',
`log_status`    INT(11)      NOT NULL COMMENT '0:normal status,1:defense status',
`log_created`   DATETIME(6)  NOT NULL COMMENT 'create datetime',
`log_modified`  DATETIME(6)  NOT NULL COMMENT 'modify datetime',
UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
AUTO_INCREMENT = 1
DEFAULT CHARSET = utf8 COMMENT ='AT transaction mode undo table';
```
