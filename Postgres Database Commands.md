# Postgres Database Common Commands

### To login<hr>
`psql -U postgres -h localhost -W`<br>
`psql -U postgres -h localhost`<br>
`psql -U postgres`<br>


### To see the list of DBs<hr>
`\l`<br>


### To exit the shell<hr>
`\q`<br>


### To backup a DB:<hr>
`pg_dump -U postgres --format=c DB_TO_BACKUP > BACK_UP_DB_NAME.dump`<br>

### To restore a DB:<hr>
`pg_restore -U postgres -d EXITING_DB DB_TO_RESTORE.dump`<br>

