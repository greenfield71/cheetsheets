# Active Directory — Condition → Tool Index (PEN-200 Modul 22 & 23)

> **Maqsad:** OSCP imtihoni va AD lablarida yuzaga keladigan aniq xolat (condition) bo'yicha qaysi vositani (tool) qayerda (Attack Host 🐧 yoki Target Host 🪟) ishlatishni darhol topish uchun tezkor ko'rsatkich (index).  
> *(Buyruqlarning to'liq sintaksisi va parametrlarini ko'rish uchun `active_directory_cheatsheet.md` fayliga qarang).*

---

## 1. Initial Access & Setup (Dastlabki Ulanish va Muhit)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| Connect to Windows target via GUI (RDP) | `xfreerdp` | 🐧 Attack Host (Kali) |
| Connect to RDP bypassing SSL certificate errors | `xfreerdp (/cert-ignore)` | 🐧 Attack Host (Kali) |
| Bypass PowerShell script execution restrictions | `powershell.exe (-ep bypass)` | 🪟 Target Host (Windows) |

---

## 2. Domain & DC Discovery (Domen va Domain Controller Aniqlash)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| Find current Domain name | `PowerView (Get-NetDomain)` | 🪟 Target Host (Windows) |
| Find current Domain name (Built-in) | `net.exe (net user)` yoki `[adsi]''` | 🪟 Target Host (Windows) |
| Find Primary Domain Controller (PDC) | `PowerShell .NET (ActiveDirectory.Domain.GetCurrentDomain())` | 🪟 Target Host (Windows) |
| Find Primary Domain Controller (PDC) via tool | `PowerView (Get-NetDomain)` | 🪟 Target Host (Windows) |
| Find Domain Distinguished Name (DN) | `PowerShell ([adsi]''.distinguishedName)` | 🪟 Target Host (Windows) |
| Construct LDAP base connection string | `PowerShell ($LDAP = "LDAP://$PDC/$DN")` | 🪟 Target Host (Windows) |
| Resolve Domain Controller or SPN hostname to IP | `nslookup.exe` | 🪟 Target Host (Windows) |
| Resolve DC/Target hostname to IP from Kali | `nslookup` / `host` | 🐧 Attack Host (Kali) |

---

## 3. Domain User Enumeration (Foydalanuvchilarni Aniqlash)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| List all domain users (Quick & built-in) | `net.exe (net user /domain)` | 🪟 Target Host (Windows) |
| Inspect specific user details (Groups, Lockout, Password dates) | `net.exe (net user <user> /domain)` | 🪟 Target Host (Windows) |
| Enumerate all domain users with detailed attributes | `PowerView (Get-NetUser)` | 🪟 Target Host (Windows) |
| Filter users by password last set & last logon timestamps | `PowerView (Get-NetUser | select pwdlastset,lastlogon)` | 🪟 Target Host (Windows) |
| Enumerate all domain users via direct LDAP query | `PowerShell (.NET DirectorySearcher / samAccountType=805306368)` | 🪟 Target Host (Windows) |
| Check user's group memberships via LDAP | `PowerShell (.NET DirectorySearcher / memberof)` | 🪟 Target Host (Windows) |
| Modular LDAP query function execution | `PowerShell (LDAPSearch function)` | 🪟 Target Host (Windows) |
| Check current logon identity & permissions | `whoami (/user /groups /priv)` | 🪟 Target Host (Windows) |

---

## 4. Domain Group Enumeration (Guruhlarni Aniqlash)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| List all domain groups (Quick & built-in) | `net.exe (net group /domain)` | 🪟 Target Host (Windows) |
| View members of a specific domain group | `net.exe (net group "<group>" /domain)` | 🪟 Target Host (Windows) |
| Enumerate all domain groups | `PowerView (Get-NetGroup)` | 🪟 Target Host (Windows) |
| View members of a specific group | `PowerView (Get-NetGroup "<group>" | select member)` | 🪟 Target Host (Windows) |
| Enumerate all groups via LDAP | `PowerShell (LDAPSearch / objectCategory=group)` | 🪟 Target Host (Windows) |
| Query members of a specific group via LDAP | `PowerShell (LDAPSearch / cn=<group> / properties.member)` | 🪟 Target Host (Windows) |

---

## 5. Computer & Operating System Discovery (Kompyuterlar va Tizimlar)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| Enumerate all domain-joined computers | `PowerView (Get-NetComputer)` | 🪟 Target Host (Windows) |
| Identify Operating Systems and build versions of domain hosts | `PowerView (Get-NetComputer | select dnshostname,operatingsystem)` | 🪟 Target Host (Windows) |
| Identify Domain Controllers in domain | `net.exe (net group "Domain Controllers" /domain)` | 🪟 Target Host (Windows) |
| Find Domain Controllers via PowerView | `PowerView (Get-NetDomainController)` / `Get-NetComputer` | 🪟 Target Host (Windows) |

---

## 6. Privilege & Local Admin Discovery (Imtiyozlarni Aniqlash)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| Find which domain machines current user has Local Admin access to | `PowerView (Find-LocalAdminAccess)` | 🪟 Target Host (Windows) |
| Check current user's Domain SID & Group memberships | `whoami (/user /groups)` | 🪟 Target Host (Windows) |

---

## 7. Session & Logged-On User Enumeration (Faol Sessiyalar)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| Enumerate active sessions on a remote host (Find admin sessions) | `PowerView (Get-NetSession)` | 🪟 Target Host (Windows) |
| Find logged on users on a remote computer via Registry/NetBIOS | `PSTools (PsLoggedon.exe)` | 🪟 Target Host (Windows) |

---

## 8. Object ACL & Permissions Enumeration (Huquqlar va Abusing)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| Enumerate Access Control List (ACL) of a specific user or group | `PowerView (Get-ObjectAcl)` | 🪟 Target Host (Windows) |
| Find accounts with full control (`GenericAll`) over a group/user | `PowerView (Get-ObjectAcl | ? {$_.ActiveDirectoryRights -eq "GenericAll"})` | 🪟 Target Host (Windows) |
| Resolve raw binary SID to human-readable account/group name | `PowerView (Convert-SidToName)` | 🪟 Target Host (Windows) |
| Add user to sensitive group when having `GenericAll` / `WriteMembers` | `net.exe (net group "<group>" <user> /add /domain)` | 🪟 Target Host (Windows) |
| Add user to group via PowerView | `PowerView (Add-DomainGroupMember)` | 🪟 Target Host (Windows) |
| Remove user from group after exploitation (Cleanup) | `net.exe (net group "<group>" <user> /del /domain)` | 🪟 Target Host (Windows) |

---

## 9. Domain Shares & GPP Passwords (Tarmoq Papkalari va Parollar)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| Find all accessible SMB file shares across the domain | `PowerView (Find-DomainShare)` | 🪟 Target Host (Windows) |
| Browse Domain Controller SYSVOL directory for Group Policy files | `PowerShell (ls \\<DC>\sysvol\<domain>\Policies)` | 🪟 Target Host (Windows) |
| Search SYSVOL XML/backup files for encrypted `cpassword` string | `PowerShell (cat / Select-String)` | 🪟 Target Host (Windows) |
| Decrypt encrypted GPP `cpassword` string to plaintext | `gpp-decrypt` | 🐧 Attack Host (Kali) |
| Inspect sensitive files in discovered domain shares | `PowerShell (ls / cat)` / `CMD (dir / type)` | 🪟 Target Host (Windows) |

---

## 10. Automated AD Graph Collection & Analysis (Avtomatlashtirilgan Tahlil)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| Automated collection of all domain users, groups, ACLs & sessions | `SharpHound (SharpHound.ps1 / Invoke-BloodHound)` | 🪟 Target Host (Windows) |
| Automated data collection using standalone binary (Bypassing PS restrictions) | `SharpHound.exe (-c All)` | 🪟 Target Host (Windows) |
| Start graph database service for BloodHound | `neo4j (sudo neo4j start)` | 🐧 Attack Host (Kali) |
| Visual attack path analysis (Shortest paths to Domain Admin) | `BloodHound` | 🐧 Attack Host (Kali) |

---

## 11. Memory Credential Harvesting (Xotiradan Parollarni Olish)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| Launch in-memory credential dumper | `mimikatz.exe` | 🪟 Target Host (Windows) |
| Elevate privileges to inspect LSASS memory | `mimikatz (privilege::debug)` | 🪟 Target Host (Windows) |
| Dump NTLM hashes and plaintext passwords from LSASS | `mimikatz (sekurlsa::logonpasswords)` | 🪟 Target Host (Windows) |
| Dump active Kerberos tickets (TGT/TGS) from LSASS memory | `mimikatz (sekurlsa::tickets)` | 🪟 Target Host (Windows) |
| View cached Kerberos tickets in current user session | `klist` | 🪟 Target Host (Windows) |

---

## 12. Password Policy & Password Spraying (Parol Hujumlari)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| Check domain password lockout threshold (Avoid locking accounts!) | `net.exe (net accounts)` | 🪟 Target Host (Windows) |
| Validate single username/password against LDAP without full logon | `PowerShell .NET (DirectoryEntry)` | 🪟 Target Host (Windows) |
| Spray single password across all domain users from Windows | `Spray-Passwords.ps1` | 🪟 Target Host (Windows) |
| Spray single password against domain accounts via SMB from Kali | `CrackMapExec (crackmapexec smb)` | 🐧 Attack Host (Kali) |
| Fast & stealthy password spray via Kerberos Pre-Auth (Windows) | `kerbrute (kerbrute_windows_amd64.exe)` | 🪟 Target Host (Windows) |
| Fast & stealthy password spray via Kerberos Pre-Auth (Kali) | `kerbrute (passwordspray)` | 🐧 Attack Host (Kali) |

---

## 13. AS-REP Roasting (Pre-Authentication Talab Qilinmaydigan Hisoblar)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| Query & extract AS-REP hashes with domain user creds from Kali | `impacket-GetNPUsers` | 🐧 Attack Host (Kali) |
| Extract AS-REP hashes from Kali without credentials (User list only) | `impacket-GetNPUsers (-no-pass -usersfile)` | 🐧 Attack Host (Kali) |
| Query & extract AS-REP hashes from Windows target | `Rubeus (Rubeus.exe asreproast)` | 🪟 Target Host (Windows) |
| Crack captured Kerberos AS-REP hashes offline | `hashcat (-m 18200)` | 🐧 Attack Host (Kali) |

---

## 14. Kerberoasting (Servis Hisoblari Hashlarini O'g'irlash)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| Find user accounts registered with Service Principal Names (SPN) | `PowerView (Get-NetUser -SPN)` | 🪟 Target Host (Windows) |
| Query SPNs and request TGS ticket hashes from Kali | `impacket-GetUserSPNs (-request)` | 🐧 Attack Host (Kali) |
| Query SPNs, request TGS tickets and export hashes from Windows | `Rubeus (Rubeus.exe kerberoast)` | 🪟 Target Host (Windows) |
| Crack captured Kerberos TGS-REP tickets offline | `hashcat (-m 13100)` | 🐧 Attack Host (Kali) |

---

## 15. Silver Ticket Attack (TGS Chiptasini Soxtalashtirish)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| Find Domain SID required for ticket forgery | `whoami (/user)` yoki `PowerView` | 🪟 Target Host (Windows) |
| Retrieve target service account NTLM hash | `mimikatz (sekurlsa::logonpasswords)` / `hashcat` | 🪟 Target Host / 🐧 Attack Host |
| Forge Silver Ticket & inject into memory (Pass-the-Ticket) | `mimikatz (kerberos::golden /ptt)` | 🪟 Target Host (Windows) |
| Verify forged ticket is active in current session | `klist` | 🪟 Target Host (Windows) |
| Access targeted web service (HTTP) using injected ticket | `PowerShell (Invoke-WebRequest / iwr -UseDefaultCredentials)` | 🪟 Target Host (Windows) |
| Access targeted file share (CIFS/SMB) using injected ticket | `CMD (dir \\<target>\c$)` | 🪟 Target Host (Windows) |

---

## 16. DCSync Attack (Domain Controller Replikatsiyasi orqali Hashlar)

| Condition / Maqsad | Tool / Buyruq nomi | Host |
|---|---|---|
| Replicate NT hash of specific user (Administrator/krbtgt) from Windows | `mimikatz (lsadump::dcsync)` | 🪟 Target Host (Windows) |
| Replicate NT hash of specific user from Kali | `impacket-secretsdump (-just-dc-user)` | 🐧 Attack Host (Kali) |
| Dump entire domain NT hashes (All accounts) from Kali | `impacket-secretsdump (-just-dc)` | 🐧 Attack Host (Kali) |
| Execute DCSync using Pass-The-Hash from Kali | `impacket-secretsdump (-hashes)` | 🐧 Attack Host (Kali) |
| Crack dumped NTLM password hashes offline | `hashcat (-m 1000)` | 🐧 Attack Host (Kali) |
