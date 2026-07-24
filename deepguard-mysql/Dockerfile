FROM mysql:9.5-oraclelinux9

COPY ./init_sql/*.sql /docker-entrypoint-initdb.d/

ENV LANG=C.UTF-8

ENV TZ=Asia/Seoul

EXPOSE 3306