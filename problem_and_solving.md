# mysql 系统更新导致服务无法启动：
原因: [ERROR] [MY-000077] [Server] /usr/sbin/mysqld: Error while setting value 'STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION' to 'sql_mode'.
解决方法：删除 /etc/mysql/mysql.conf.d/mysqld.cnf 日志中
sql_mode


