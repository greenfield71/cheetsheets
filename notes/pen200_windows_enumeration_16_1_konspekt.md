# PEN-200: 16.1.1 & 16.1.2 — Windows Enumeration va Privileges Konspekti

Ushbu konspekt **PEN-200 (OffSec Certified Professional - OSCP)** kursining **16.1.1** va **16.1.2** bo'limlari asosida tayyorlandi. Maqsad — Windows tizimiga kirgandan so'ng xavfsizlik mexanizmlarini to'g'ri tushunish, terminal buyruqlarini yodda saqlash va tizim haqida to'liq ma'lumot (Situational Awareness) yig'ish.

---

## 1-QISM: 16.1.1. Windows Privileges va Ruxsatlarni Boshqarish Mexanizmlari

Windows operatsion tizimida foydalanuvchilarning fayllarni o'zgartirish, servislar qo'shish yoki tizim sozlamalarini boshqarish huquqlari 4 ta asosiy mexanizm orqali nazorat qilinadi:

### 1. SID (Security Identifier) — Xavfsizlik Identifikatori
> **Asosiy qoida:** Windows foydalanuvchi yoki guruhlarni ularning nomlari (username) bilan emas, balki faqat **SID** orqali taniydi va ruxsatlarni SID asosida boshqaradi.

* **SID tuzilishi:** `S - R - X - Y`
  * `S` — Qatorning SID ekanligini bildiradi.
  * `R` *(Revision)* — Versiya (hozircha doim `1`).
  * `X` *(Identifier Authority)* — SID-ni bergan organ (ko'pincha `5` = `NT Authority`).
  * `Y` *(Sub-authorities)* — Domen/mashina identifikatori va oxirgi qism — **RID (Relative Identifier)**.

* **RID qoidalari:**
  * Oddiy lokal foydalanuvchi va guruhlar yaratilganda ularning RID qiymati **`1000`** dan boshlanadi (masalan, birinchi yaratilgan qo'shimcha foydalanuvchi — `1000`, ikkinchisi — `1001`).
  * **Well-known SIDs (Mashhur / Standart SIDlar):** RID qiymati `1000` dan kichik bo'ladi:
    * `S-1-0-0` — **Nobody** (Hech kim)
    * `S-1-1-0` — **Everyone** (Barcha foydalanuvchilar)
    * `S-1-5-11` — **Authenticated Users** (Autentifikatsiyadan o'tgan foydalanuvchilar)
    * `S-1-5-18` — **Local System (SYSTEM)** (Eng yuqori tizim darajasi)
    * `S-1-5-...-500` — **Administrator** (RID `500` har doim o'rnatilgan built-in Administrator hisoblanadi)

---

### 2. Access Tokens (Kirish Tokenlari)
Foydalanuvchi tizimga kirganida (logon), Windows unga **Access Token** yaratib beradi. Bu token foydalanuvchining **Security Context** (xavfsizlik konteksti)ni saqlaydi (foydalanuvchi SID-si, a'zo bo'lgan guruhlar SID-lari, berilgan imtiyozlar).

* **Primary Token (Asosiy token):** Foydalanuvchi yangi jarayon (process) yoki oqim (thread) ochganda, unga ushbu foydalanuvchining asosiy tokeni nusxalanib beriladi.
* **Impersonation Token (Soxtalashtirish / O'zini boshqa nom bilan ko'rsatish tokeni):** Jarayon ichidagi alohida oqim (thread) o'z egasining tokeni emas, vaqtincha boshqa foydalanuvchi (masalan, server mijozi) huquqlari bilan biror amal bajarishi kerak bo'lganda ishlatiladi.

---

### 3. Mandatory Integrity Control (MIC) — Butunlik Darajalari
Windows dasturlar va obyektlarga ishonch darajasini belgilash uchun 5 ta butunlik darajasidan foydalanadi:
1. **System:** SYSTEM darajasi (OS yadrosi, drayverlar, asosiy xizmatlar).
2. **High:** Elevated (Administrator sifatida ishga tushirilgan dasturlar).
3. **Medium:** Standard Users (Oddiy foydalanuvchilar va ularning dasturlari).
4. **Low:** Sandboxed jarayonlar yoki vaqtinchalik papkalar (juda cheklangan).
5. **Untrusted:** Eng past daraja (eng yuqori xavfga ega obyektlar).

> **Oltin Qoida:** Quyi integrity levelga ega bo'lgan jarayon (masalan, Medium) yuqoriroq darajadagi obyektga (masalan, High) yoza olmaydi (write qila olmaydi), hatto NTFS ruxsati bunga ruxsat bergan taqdirda ham.

---

### 4. User Account Control (UAC)
Agar foydalanuvchi Administrator guruhida bo'lsa ham, tizimga kirganda unga ikkita token beriladi:
1. **Filtered Admin Token (standart foydalanuvchi tokeni):** Kundalik dasturlar va xavfsiz amallar uchun (Medium integrity).
2. **Administrator Token (to'liq admin tokeni):** Ma'muriy amallarni bajarish uchun (High integrity).
Ushbu to'liq tokenni faollashtirish uchun ekranda **UAC Consent Prompt** (ruxsat so'rash oynasi) chiqadi va tasdiqlanishi kerak bo'ladi.

---

## 2-QISM: 16.1.2. Situational Awareness (Vaziyatni To'liq O'rganish)

Tizimga dastlabki kirishga (masalan, bind shell yoki reverse shell) ega bo'lganingizda, darhol exploit qidirishga shoshilmaslik kerak. Avval **7 ta asosiy ma'lumotni** to'plash shart:

```text
1. Username va Hostname (Men kimman va qaysi mashinadaman?)
2. Current user guruhlari (Qaysi guruhlarga a'zoman?)
3. Boshqa foydalanuvchilar va guruhlar (Tizimda yana kimlar bor?)
4. OS nomi, versiyasi va arxitekturasi (Windows qaysi versiya va 32/64-bit?)
5. Tarmoq ma'lumotlari (IP, Gateway, Routing, ochiq portlar)
6. O'rnatilgan dasturlar (Third-party dasturlar, masalan KeePass, XAMPP, FileZilla)
7. Hozirda ishlayotgan jarayonlar (Running processes)
```

---

## 3-QISM: Terminallar va Buyruqlar Konspekti

Windows-da ikkita asosiy terminal muhiti mavjud:
* **CMD (Command Prompt):** Standart qobiq, oddiy shell ulanganda odatda shu ochiladi.
* **PowerShell:** Murakkabroq, ob'ektlar bilan ishlaydigan kuchli skriptlash qobig'i (`CMD` ichidan `powershell` buyrug'i orqali o'tiladi).

Quyida har bir ma'lumot turi uchun kerakli buyruqlar, ularning vazifasi va nimalarga e'tibor berish kerakligi keltirilgan:

### 1. Kim bo'lib turibmiz va kompyuter nomi nima?
* **Buyruq (CMD / PowerShell):**
  ```cmd
  whoami
  ```
* **Tushuntirish:** Natija `hostname\username` formatida chiqadi (masalan, `clientwk220\dave`).
* **Nimaga e'tibor berish kerak:** 
  * `hostname` orqali mashinaning vazifasini tushunish mumkin (`CLIENTWK` = oddiy ish kompyuteri, `WEB01` = veb-server, `MSSQL01` = ma'lumotlar bazasi).

---

### 2. Joriy foydalanuvchi qaysi guruhlarda bor?
* **Buyruq (CMD / PowerShell):**
  ```cmd
  whoami /groups
  ```
* **Muhim guruhlar:**
  * `BUILTIN\Remote Desktop Users` — Tizimga RDP orqali grafik interfeysda kirish imkonini beradi.
  * `helpdesk`, `adminteam` — Qo'shimcha huquqlar beruvchi nostandart guruhlar.
  * Natija oxirida foydalanuvchining **Integrity Level**-i ham ko'rinadi (masalan, `Medium Mandatory Level`).

---

### 3. Tizimdagi boshqa foydalanuvchilarni ko'rish
* **PowerShell buyrug'i:**
  ```powershell
  Get-LocalUser
  ```
* *(CMD muqobili: `net user`)*
* **Nimaga e'tibor berish kerak:**
  * `Administrator` hisobi yoqilganmi yoki o'chirilganmi (`Enabled: True/False`).
  * Nomida `admin`, `backup`, `service` so'zlari bor foydalanuvchilar (masalan: `BackupAdmin`, `daveadmin`). Bular biz uchun birlamchi nishon hisoblanadi.

---

### 4. Tizimdagi lokal guruhlarni ko'rish
* **PowerShell buyrug'i:**
  ```powershell
  Get-LocalGroup
  ```
* *(CMD muqobili: `net localgroup`)*
* **E'tibor beriladigan guruhlar:**
  * `Administrators` — Tizim ma'murlari.
  * `Backup Operators` — Hatto o'zlariga ruxsat berilmagan fayllarni ham nusxalash (backup) va o'qish huquqiga ega guruh.
  * `Remote Desktop Users` — RDP ruxsati.
  * `Remote Management Users` — WinRM (masofaviy PowerShell) ruxsati.
  * Maxsus guruhlar: `adminteam`, `BackupUsers` kabi qo'lda yaratilgan guruhlar.

---

### 5. Guruh a'zolarini tekshirish
* **PowerShell buyrug'i:**
  ```powershell
  Get-LocalGroupMember <Guruh_Nomi>
  ```
* **Amaliy misollar:**
  ```powershell
  Get-LocalGroupMember adminteam
  Get-LocalGroupMember Administrators
  ```
* *(CMD muqobili: `net localgroup Administrators`)*
* **Maqsad:** Qaysi akkauntlar administrator ekanligini aniq bilib olish.

---

### 6. Operatsion tizim, versiya va arxitektura
* **Buyruq (CMD / PowerShell):**
  ```cmd
  systeminfo
  ```
* **Nimani qidiramiz:**
  * `OS Name:` Windows 11 Pro, Windows Server 2019 va hk.
  * `OS Version:` Build raqami (masalan, `Build 22621` = Windows 11 22H2).
  * `System Type:` **`x64-based PC`** yoki **`x86`**.
    > ⚠️ **Muhim:** Agar tizim 32-bit bo'lsa, unga 64-bitlik eksployt yoki payload tashlasangiz ishlamaydi! Shuning uchun arxitekturani tekshirish shart.

---

### 7. Tarmoq interfeyslari va sozlamalari
* **Buyruq (CMD / PowerShell):**
  ```cmd
  ipconfig /all
  ```
* **Maqsad:** IP manzil, tarmoq niqobi (subnet mask), Default Gateway, DNS serverlar va IP avtomatik (DHCP) olinganmi yoki statik berilganini ko'rish.

---

### 8. Marshrutlash jadvali (Routing Table)
* **Buyruq (CMD / PowerShell):**
  ```cmd
  route print
  ```
* **Maqsad:** Mashinada biz bilmagan boshqa ichki tarmoqlar (subnets/dual-homed interfaces) bormi-yo'qligini aniqlash (keyinchalik pivoting/lateral movement qilish uchun).

---

### 9. Ochiq portlar va faol ulanishlar
* **Buyruq (CMD / PowerShell):**
  ```cmd
  netstat -ano
  ```
  * `-a` — Barcha ulanishlar va tinglanayotgan (listening) portlarni ko'rsatish.
  * `-n` — IP va portlarni nomga o'girmasdan (raqamli) ko'rsatish (juda tez ishlaydi).
  * `-o` — Har bir portga tegishli **PID (Process ID)** raqamini ko'rsatish.
* **Xavfsizlik nuqtai nazaridan tekshirish:**
  * `80`, `443` — Veb-server (Apache, IIS).
  * `3306` — MySQL ma'lumotlar bazasi.
  * `3389` — RDP (agar `ESTABLISHED` holatida bo'lsa, demak ayni paytda kimdir RDP orqali tizimda o'tiribdi — bu foydalanuvchining parolini Mimikatz orqali xotiradan olish mumkin!).

---

### 10. O'rnatilgan dasturlar (Registry orqali tekshirish)
Ko'p dasturlar oddiy ro'yxatlarda ko'rinmasligi mumkin, shuning uchun Windows Registry kalitlarini o'qish eng samarali usul:
* **PowerShell buyruqlari:**
  * **32-bit dasturlar uchun:**
    ```powershell
    Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
    ```
  * **64-bit dasturlar uchun:**
    ```powershell
    Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
    ```
* **Nimaga e'tibor berish kerak:**
  * Third-party dasturlar: `FileZilla`, `KeePass`, `7-Zip`, `XAMPP`.
  * Ushbu dasturlar orqali PrivEsc qilish mumkin (masalan, KeePass master paroli, zaif servislar, saqlangan parollar).
* **Qo'shimcha tekshiruv:** `C:\Program Files`, `C:\Program Files (x86)` va foydalanuvchining `Downloads` papkasini ko'rib chiqish tavsiya etiladi.

---

### 11. Ishlayotgan jarayonlarni (Processes) ko'rish
* **PowerShell buyrug'i:**
  ```powershell
  Get-Process
  ```
* *(CMD muqobili: `tasklist`)*
* **Maqsad:** `netstat -ano` dagi PID raqamlarini solishtirish. Masalan, `PID 3508` qaysi jarayonga tegishli? (`mysqld`), `PID 4316` nima? (`httpd` - Apache).

---

## Tezkor Yodlash Jadvali (Cheat-Sheet)

| Qidirilayotgan Ma'lumot | Terminal / Muhit | Buyruq | Izoh / Natija |
| :--- | :--- | :--- | :--- |
| **User & Hostname** | CMD / PS | `whoami` | `mashina\foydalanuvchi` |
| **Foydalanuvchi guruhlari** | CMD / PS | `whoami /groups` | RDP, helpdesk huquqlari va Integrity level |
| **Barcha foydalanuvchilar** | PowerShell | `Get-LocalUser` | Adminlar va boshqa akkauntlar ro'yxati |
| **Barcha guruhlar** | PowerShell | `Get-LocalGroup` | Standart va maxsus guruhlar |
| **Guruh a'zolari** | PowerShell | `Get-LocalGroupMember <Guruh>` | Guruh ichidagi aniq foydalanuvchilar |
| **Tizim va arxitektura** | CMD / PS | `systeminfo` | OS versiyasi, Build va x64/x86 arxitektura |
| **Tarmoq konfiguratsiyasi** | CMD / PS | `ipconfig /all` | IP, Gateway, DNS |
| **Marshrutlash jadvali** | CMD / PS | `route print` | Tarmoq yo'nalishlari |
| **Ochiq portlar va PID** | CMD / PS | `netstat -ano` | Tinglanayotgan portlar va jarayon ID-lari |
| **32-bit dasturlar** | PowerShell | `Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" \| select displayname` | O'rnatilgan uchinchi tomon dasturlari |
| **64-bit dasturlar** | PowerShell | `Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" \| select displayname` | O'rnatilgan uchinchi tomon dasturlari |
| **Faol jarayonlar** | PowerShell | `Get-Process` | Hozirda xotirada ishlab turgan dasturlar |
