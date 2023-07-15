# Postgres Database Common Commands

### Environment variable setting<hr>
Machine will not recognize `psql` `pg_dump` `pg_restore` commands.<br>
For windows, <br>
This PC > Properties > <br>
Advance system settings > <br>
Environment Variables > Select Path and Click Edit<br>
Set: C:\Program Files\PostgreSQL\12\bin

### Binary Path setting in PG ADMIN<hr>
File > Preferences > <br>
Paths > Binary Path<br>
PostgreSQL Binary Path (not `EDB Advanced Server Binary Path`)> PostgreSQL 12<br>
Set: C:\Program Files\PostgreSQL\12\bin


### To login<hr>
`psql -U postgres -h localhost -W`<br>
`psql -U postgres -h localhost`<br>
`psql -U postgres`<br>


### To see the list of DBs<hr>
`\l`<br>


### To exit the shell<hr>
`\q`<br>


### To backup a DB:<hr>
1. let's go to the `directory/folder` (e.g. desktop) where I want to backup the file.<br>
2. Then open the command prompt/shell inside the desktop and run the following command <br><br>
```
pg_dump -U postgres --format=c DB_TO_BACKUP > BACK_UP_DB_NAME.dump
```
NOTE: <br>
If we are looking to use `pg_dump`, the command is meant for normal command prompt not for `psql` command prompt.<br>
we don't have to login to `psql`; we will directly use `pg_dump` commands. <br>


### To restore a DB:<hr>
1. let's go to the `directory/folder` (e.g. desktop) where I want to restore the file from<br>
2. Then open the `command prompt`/`shell` inside the desktop and run the following command <br><br>
```
pg_restore -U postgres -d EXITING_DB DB_TO_RESTORE.dump
```
NOTE: <br>
If we are looking to use `pg_restore`, the command is meant for normal command prompt not for `psql` command prompt.<br>
we don't have to login to `psql`; we will directly use `pg_restore` commands. <br>


### To restore a DB for remote server (e.g. AWS Lightsail):<hr>
NOTE: we don't have to login to `psql`; we will directly use `pg_restore` commands<br><br>
1. let's go to the directory/folder where I want to restore the file from (e.g. desktop)<br>
2. Then open the `command prompt`/`shell` inside the directory and run the following command <br><br>
```
pg_restore --verbose --clean --no-acl --no-owner -h ls-xxxxxxxxx.codu04qtcvtr.ap-xxxxx-1.rds.xxxxx.com -p 5432 -U wims -d db_in_my_pc -v db_in_remote
```
3. It will ask password for remote server (in my case, aws lightsail RDS password for the user 'wims')
4. If password ok, then it will print out restoring steps automatically in the console.


### To restore a DB for remote server (e.g. AWS RDS):<hr>
NOTE: we don't have to login to `psql`; we will directly use `pg_restore` commands<br><br>
1. let's go to the directory/folder where I want to restore the file from (e.g. desktop)<br>
2. Then open the `command prompt`/`shell` inside the directory and run the following command <br><br>
```
pg_restore --verbose --clean --no-acl --no-owner -h notbook-beta-db-instance.xxxxxx.ap-northeast-1.rds.amazonaws.com -p 5432 -U postgres -d notebookbetaDB C:\Users\monir\OneDrive\Desktop\notebookbetaDB.dump
```
3. It will ask password for remote server (in my case, aws RDS password for the user 'postgres')
4. If password ok, then it will print out restoring steps automatically in the console.

### Common Errors:<hr>

#### Error 01: pg_restore: error: could not open input file "notebookbetaDB": No such file or directory<br>
#### Reason: 
It needs a path
#### Solution: 
Specifying the path of the dump file like this ... ... -U postgres -d notebookbetaDB C:\Users\monir\OneDrive\Desktop\notebookbetaDB.dump


#### Error 02: pg_restore: error: input file is too short (read 0, expected 5)<br>
#### Reason: 
If the dump file is corrupted or incomplete. <br>
#### Solution: 
Check the size of the dump file and make sure it's not zero bytes. You can do this by right-clicking on the file in Windows Explorer and selecting "Properties."<br>

#### Error 03: pg_restore: error: input file appears to be a text format dump. Please use psql.<br>
#### Solution: 
1. let's go to the `directory/folder` (e.g. desktop) where I want to restore the file from<br>
2. Then open the `command prompt`/`shell` inside the desktop and run the following command <br><br>
```
psql -d existing_db_name -U postgres -f db_to_restore.sql
```
For AWS RDS
```
psql -h analytics-beta-db-instance.xxxxxxxxxx.ap-northeast-1.rds.amazonaws.com -U postgres -d analyticsbetaDB < dump0622.sql
```

