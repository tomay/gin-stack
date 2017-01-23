## docker notes.md

following this: 
https://www.andreagrandi.it/2015/02/21/how-to-create-a-docker-image-for-postgresql-and-persist-data/

### in sum
1. create the `Dockerfile`
2. To build `docker build --rm=true -t tomay/postgresql:9.3 .`
3. To run, once built: `docker run -i -t -p 5432:5432 tomay/postgresql:9.3`

### persisting data
1. install fig `pip install fig`
2. create this fig file in the same folder as the Dockerfile
```yml
dbdata:
  image: tomay/postgresql:9.3
  volumes:
    - /var/lib/postgresql
  command: "true"
 
db:
  image: tomay/postgresql:9.3
  volumes_from:
    - dbdata
  ports:
    - "5432:5432"
```
3. Run with `fig up`

To test the running container we can use any client, even the commandline one:

`psql -h localhost -p 5432 -U pguser -W pgdb`

When you are prompted for password, type: `pguser`

### create a database, table, data
```sql
CREATE DATABASE testdb;
\c testdb;
CREATE TABLE customers(id int PRIMARY KEY NOT NULL);
insert into customers (id) values (1);
insert into customers (id) values (2);
insert into customers (id) values (3);
```

### stop and restart, then check the data
1. Wherever fig is running `ctl+c`
2. Start the container again `fig up`
3. connect to the db and list
```sql
psql -h localhost -p 5432 -U pguser -W pgdb
\c testdb; 
select * from customers;
```
