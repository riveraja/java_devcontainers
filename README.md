## Java Devcontainers

Sample Java app in devcontainers, with ProxySQL and MySQL in the backend.

### Create mysql users with the right privileges and authentication method
```bash
docker exec -it mysql1 bash
mysql -uroot -p$MYSQL_ROOT_PASSWORD -e "CREATE USER 'root'@'172.%' IDENTIFIED WITH mysql_native_password BY 't00r'"
mysql -uroot -p$MYSQL_ROOT_PASSWORD -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'172.%'"
mysql -uroot -p$MYSQL_ROOT_PASSWORD -e "CREATE USER 'monit'@'172.%' IDENTIFIED WITH mysql_native_password BY 'monit0r'"
mysql -uroot -p$MYSQL_ROOT_PASSWORD -e "GRANT ALL PRIVILEGES ON *.* TO 'monit'@'172.%'"
```

### Test if connections work from ProxySQL container
```bash
docker exec -it proxysql1 bash
mysql -uroot -p$MYSQL_ROOT_PASSWORD -e "show databases"
```

### Create database/table
```sql
CREATE DATABASE test;
CREATE TABLE x (
    id int auto_increment not null primary key,
    c varchar(100)
);
```

### Initialize the database
```bash
#!/bin/bash

COUNT=$1

echo "Creating ${COUNT} items..."
i=0

while [ $i -le $COUNT ];
do
	MOD=$((i % 1000))
	if [ $i -gt 0 ] && [ $MOD -eq 0 ];
		then echo "Inserted ${i} items."
	fi
	RANDSTRING=$(tr -dc A-Za-z0-9 </dev/urandom | head -c 13)
	mysql -uroot -p$MYSQL_ROOT_PASSWORD -h127.0.0.1 -P6033 -e "INSERT INTO test.x (c) VALUES ('$RANDSTRING');"
	((i++))
done
```