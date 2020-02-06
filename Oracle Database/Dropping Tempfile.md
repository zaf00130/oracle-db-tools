```
alter database tempfile '/u01/oradata/temp02.dbf' offline;
```
```
alter database tempfile '/u01/oradata/temp02.dbf' drop including datafiles;
```
```
alter database tempfile '/u01/oradata/temp02.dbf' drop including datafiles
*
ERROR at line 1:
ORA-25152: TEMPFILE cannot be dropped at this time
```

```
SELECT DISTINCT b.sid, b.username, b.status, b.program, c.file_name, c.tablespace_name
FROM v$sort_usage a
LEFT JOIN v$session b ON (a.session_addr = b.saddr)
LEFT JOIN dba_temp_files c ON (a.segfile# = c.file_id + (SELECT value FROM v$parameter WHERE name = 'db_files'))
ORDER BY c.file_name, b.sid
;
```

| SID  	| USERNAME 	| STATUS   	| PROGRAM                    	| FILE_NAME               	| TABLESPACE_NAME 	|
|------	|----------	|----------	|----------------------------	|-------------------------	|-----------------	|
| 231  	| APPS     	| ACTIVE   	| frmweb@server1 (TNS V1-V3) 	| /u01/oradata/temp01.dbf 	| TEMP            	|
| 477  	| APPS     	| INACTIVE 	| JDBC Thin Client           	| /u01/oradata/temp01.dbf 	| TEMP            	|
| 479  	| APPS     	| INACTIVE 	| JDBC Thin Client           	| /u01/oradata/temp01.dbf 	| TEMP            	|
| 929  	| APPS     	| INACTIVE 	| frmweb@server1 (TNS V1-V3) 	| /u01/oradata/temp01.dbf 	| TEMP            	|
| 1164 	| APPS     	| INACTIVE 	| frmweb@server1 (TNS V1-V3) 	| /u01/oradata/temp01.dbf 	| TEMP            	|
| 54   	| APPS     	| INACTIVE 	| JDBC Thin Client           	| /u01/oradata/temp02.dbf 	| TEMP            	|
| 1616 	| APPS     	| INACTIVE 	| JDBC Thin Client           	| /u01/oradata/temp02.dbf 	| TEMP            	|



I needed to drop a tempfile while the database was up but I was getting an error  

`ORA-25152: TEMPFILE cannot be dropped at this time`  

So I wanted to see which session(s) were using a particular tempfile. 
At this point I can kill the sessions using temp02.dbf or wait for them to close out and proceed with dropping the tempfile.
