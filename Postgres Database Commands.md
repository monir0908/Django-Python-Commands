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
1. let's go to the directory/folder where I want to backup the file.<br>
2. Then open the command prompt/shell inside the directory and run the following command <br><br>
`pg_dump -U postgres --format=c DB_TO_BACKUP > BACK_UP_DB_NAME.dump`<br><br>
NOTE: <br>
If we are looking to use `pg_dump`, the command is meant for normal command prompt not for `psql` command prompt.<br>
we don't have to login to `psql`; we will directly use `pg_dump` commands. <br>


### To restore a DB:<hr>
1. let's go to the `directory/folder` where I want to restore the file from<br>
2. Then open the `command prompt`/`shell` inside the directory and run the following command <br><br>
`pg_restore -U postgres -d EXITING_DB DB_TO_RESTORE.dump`<br><br>
NOTE: <br>
If we are looking to use `pg_restore`, the command is meant for normal command prompt not for `psql` command prompt.<br>
we don't have to login to `psql`; we will directly use `pg_restore` commands. <br>


### To restore a DB for remote server (e.g. AWS Lightsail):<hr>
NOTE: we don't have to login to `psql`; we will directly use `pg_restore` commands<br>
1. let's go to the directory/folder where I want to restore the file from<br>
2. Then open the `command prompt`/`shell` inside the directory and run the following command <br><br>
`pg_restore --verbose --clean --no-acl --no-owner -h ls-xxxxxxxxx.codu04qtcvtr.ap-xxxxx-1.rds.xxxxx.com -p 5432 -U wims -d db_in_my_pc -v db_in_remote`<br><br>
3. It will ask password for remote server (in my case, aws lightsail RDS password for the user 'wims')
4. If password ok, then it will print out restoring steps automatically in the console as we used `-v` flag

