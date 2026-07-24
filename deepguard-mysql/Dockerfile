FROM mysql:8.0-debian

COPY ./init_sql/*.sql /docker-entrypoint-initdb.d/

ENV MYSQL_DATABASE=mydb
ENV MYSQL_USER=myuser
ENV MYSQL_PASSWORD=mypassword

EXPOSE 3306