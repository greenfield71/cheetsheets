to connect mssql
locate mssqlclient    -   from Impacket

Footprinting the Service
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248

MSSQL Ping in Metasploit
(scanner/mssql/mssql_ping)

Connecting with Mssqlclient.py
python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth
