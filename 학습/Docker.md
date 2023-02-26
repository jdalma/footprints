
```yaml
version: "3.9"
services:
  db:
    image: mysql:8.0.30
    platform: linux/x86_64
    restart: always
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: data-access
      TZ: Asia/Seoul
    volumes:
#      - ./db/mysql/data:/var/lib/mysql
#      - ./db/mysql/config:/etc/mysql/conf.d
      - ./db/mysql/init:/docker-entrypoint-initdb.d
```

`./db/mysql/init:/docker-entrypoint-initdb.d`이 명령으로 `init`폴더를 도커 내부에 복사하여 `init.sql`을 실행시킨다.  

1. `docker-compose -p {name} up -d` : 해당 정보로 컨테이너를 실행키산다.
   - `-d` 키워드를 추가하면 로그가 보이지 않고, 제거해야 로그가 보인다.
   - `insert`쿼리가 제대로 실행되지않아 `-d` 키워드를 제거하고 로그를 확인하였다.
2. `docker-compose -p {name} down` : 컨테이너를 제거한다.