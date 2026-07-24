FROM mysql:9.5-oraclelinux9

COPY ./init_sql/*.sql /docker-entrypoint-initdb.d/

COPY ./config/mysql.cnf /etc/mysql/conf.d/mysql.cnf

ENV LANG=C.UTF-8
ENV TZ=Asia/Seoul

EXPOSE 3306