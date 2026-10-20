[connection]
	rpcclient -U "" -N 172.16.5.5
	
[password policy]
	querydominfo
	
[user enumeration]
	enumdomusers
	
[user enumeration by rid]
	queryuser 0x457
	
		
