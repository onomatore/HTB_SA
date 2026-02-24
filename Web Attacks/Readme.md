## Задание

`Попробуйте повысить свои привилегии и воспользоваться различными уязвимостями, чтобы прочитать флаг на '/flag.php'.`

`Выполните аутентификацию в с помощью пользователя "htb-student" и пароля "Academy_student!"`

## Решение

Перейдя по ссылке требуется авторизация, выполняем её с помощью выданных кредов.

После авторизации попадаем на страницу `/profile.php`
![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/1.png)

Большая часть сайта заглушки, однако есть интресность а именно настройки аккаунта -  `/settings.php` , где можно изменить пароль пользователя. Попробуем изменить наш пароль и перехватим запрос.
В открытом виде лежит функция смены пароля:
![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/2.png)

Мне стало интересно и я решил посмотреть перехваченный запрос профиля и там я увидел ещё кое-что интересное
![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/3.png)

Видимо куки передаются в открытом виде без шифра и прочего, значит есть возможность для перечисления пользовтелей.

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/4.png)

Действительно, видим что наш uid передается в виде cookie. Я изменил uid на 75, получил статус 200 и вывод страницы. Попробуем перечислить 300 пользователей, может найдем что-нибудь интересное. Перечисление ничего не дало везде одинаковые данные. Тогда вернемся к смене пароля и наконец попробуем его изменить. 


Решил перейти в браузер ZAPа, потому что в обычном фаерфоксе были какие-то траблы с обновлением пароля, либо определена только функция, а кнопка для виду, либо косяк с браузером. Так или иначе перезагрузил сервер задания, снова залогинился.

Словил такой запрос когда логинился.

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/5.png)

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/6.png)

И тут наконец прогрузилось имя профиля, а также за бесконечными ожиданиями ответа от всех сайтов со стилями, перехватился такой интересный запрос.

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/7.png)

Собственно решив изменить uid и / на 75, получил такой ответ. То есть успешно получилось сменить пользователя. Значит можем вернуться к гипотезе о перечислении пользователей.

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/8.png)

После этого, да пользователь сменился.

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/9.png)

Теперь пробуем перечислить остальных.
Пользователей всего 100, но ничего потенциально нужно нет.
Пробую сбросить пароль, на этот раз книпка жмется и перехватываются следующие запросы:

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/1.png)

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/11.png)


![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/12.png)

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/13.png)


Пробуем подменить токен, чтобы изменить пароль для другого пользователя.
Просто изменить на `/50`  не получилось, ошибка 'access denied'  и токен для 74 пользователя, попробуем с подменой куки.
`{"token":"e51a85aa-17ac-11ec-8e4f-17183faa5d02"}`
С подменной куки и токен тоже изменился,  но пользователь был попрежнему 74, поэтому новая ошибка `invalid token`, значит попробуем его декэшировать - не получилось.

В любом другом случае ловим ошибку `access denied`, но если меняем через метод `GET`, то все успешно и мы сменили пароль для 50 пользователя. Попробуем залогинится, так как мы знаем его логин из перечисления, а пароль только что изменили.

Просматривая перечисленных пользователей, наткнулся на админа - `uid=52`

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/14.png)

`username:"a.corrales"`

Если просто подменить запрос `/api/token/52`, то хоть имя пользователя и измениться, но доступа к нему в профиль у нас не будет. Поэтому меняем пароль для профиля админа по уже отработанной стратегии, и логинимся.

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/15.png)

Видим, что у нас добавилась новая функция, добавить событие, также проверим нет ли чего нового в настройках - там все та же смена пароля. Пробуем понять как через добавление события, мы сможем получить доступ к системе.  Нажав нас переадресует на`/event.php` и мы видим следующее

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/16.png)

Создадим тестовое событие и перехватим запрос.

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/17.png)

Нас переадресует на `/addEvent.php` и мы видим возможность для `XXE`.

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/18.png)

Поле `name` выводит нам текст, поэтому пробуем через него определить тип нашей уязвимости.

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/19.png)

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/20.png)


Теперь нам нужно попробовать прочитать любой файл .php , это будет `index.php` -  успешно получается, значит мы можем пробовать читать наш флаг.
Используем простой пейлоад:

`<!DOCTYPE email [`
  `<!ENTITY company SYSTEM  "php://filter/convert.base64-encode/resource=/flag.php">`
`]>`
`...`
`<name>&company;</name>`

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/21.png)

и получим b64 строку: `PD9waHAgJGZsYWcgPSAiSFRCe200NTczcl93M2JfNDc3NGNrM3J9IjsgPz4K`, декодим её и получаем флаг: <?php $flag = "HTB{m4573r_w3b_4774ck3r}"; ?>

![image](https://github.com/onomatore/HTB_SA/blob/main/Web%20Attacks/img/22.png)
