##### 1.用户授权

revoke all privileges on 数据库.* from '账号'@'%' ；
grant create,select ,insert,update,delete on allocacoc.* to 'allocacoc'@'%' identified by '密码';
flush privileges;