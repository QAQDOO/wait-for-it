# wait-for-it

docker compose能够编排容器实例，但是容器里服务启动有快有慢，无法编排服务的启动顺序。

wait-for-it.sh脚本用于检测服务端口（例如mysql的3306端口）是否启动成功，成功后才能启动下一个服务

wait-for-it.sh ip:port 检测该ip与端口是否已经能连接，能了就执行后面的命令

参数：
-t 5     #检测该ip与端口是否已经能连接，超过指定时间5s后无论ip与端口是否已经能连接都执行后续命令 


-t 0     #不设置超时参数 或者加 --strict   ip与端口不能连接就不执行后续命令


#不加 -t 0 或者 -t 超时时间 都会在超时时间过了之后执行后续命令 不管ip与端口是否已经能连接
#默认超时时间 15秒


wait-for-it.sh脚本使用实例：

services:
  user-Service:
    image: userms:5.0
    container_name: userms
    ports:
      - "6066:6062"
    volumes:
      - /app/userMsDocker/tmp:/tmp
    networks: 
      - hsp_net
    command: ./wait-for-it.sh nacos:8848 -t 0 -- java -jar /user_ms_docker.jar
    depends_on: 
      - redis
      - mysql
      - nacos
  redis:
    image: redis
    ports:
      - "6381:6379"
    volumes:
      - /app/redis/data:/data
    networks: 
      - hsp_net
    command: --appendonly yes
  nacos:
    image: nacos/nacos-server
    environment:
      MODE: 'standalone'
    ports:
      - "8849:8848"
    networks: 
      - hsp_net
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: 'root'
    ports:
       - "3307:3306"
    volumes:
       - /app/mysql/master/log:/var/log/mysql
       - /app/mysql/master/data:/var/lib/mysql
       - /app/mysql/master/conf:/etc/mysql/conf.d
    networks:
      - hsp_net
networks: 
   hsp_net: 
