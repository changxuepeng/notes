##### 1.mysql性能监控-profile

```mysql
set  profiling =1;

select * from  student;
show profiles;

show profile;

```

![20210525145049](mysql调优.assets/20210525145049.jpg)

##### 2.mysql性能监控-performance schema

默认情况下是开启的