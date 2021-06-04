参考网址

https://www.jianshu.com/p/8c2ffeefa206

```dockerfile
FROM
ADD   xxx.jar  /xxx.jar

RUN chmod +x /docker-entrypoint.sh

ENV SPRING_PROFILES_ACTIVE="test"

EXPOSE 8080
ENTRYPOINT ["bash","docker-entrypoint.sh"]
```



docker-entrypoint.sh

```shell
#!/bin/bash

echo "starting...."
exec java -jar $JAVA_OPTS -Dspring.profiles.active=$SPRING_PROFILES_ACTIVE "xxxx.jar"
```

