## Задание

`Попробуйте повысить свои привилегии и воспользоваться различными уязвимостями, чтобы прочитать флаг на '/flag.php'.`

`Выполните аутентификацию в с помощью пользователя "htb-student" и пароля "Academy_student!"`

## Решение

Перейдя по ссылке требуется авторизация, выполняем её с помощью выданных кредов.

После авторизации попадаем на страницу `/profile.php`
<img width="1336" height="835" alt="image" src="https://github.com/user-attachments/assets/255979ea-bdef-44f2-8124-e8f0caeb2192" />

Большая часть сайта заглушки, однако есть интресность а именно настройки аккаунта -  `/settings.php` , где можно изменить пароль пользователя. Попробуем изменить наш пароль и перехватим запрос.
В открытом виде лежит функция смены пароля:
<img width="990" height="475" alt="image" src="https://github.com/user-attachments/assets/1773ccdd-a795-478e-89bd-9197622771f1" />

Мне стало интересно и я решил посмотреть перехваченный запрос профиля и там я увидел ещё кое-что интересное
<img width="466" height="218" alt="image" src="https://github.com/user-attachments/assets/2b227624-0fcc-4dab-ac81-b3e3848e6562" />

Видимо куки передаются в открытом виде без шифра и прочего, значит есть возможность для перечисления пользовтелей.
<img width="757" height="279" alt="image" src="https://github.com/user-attachments/assets/c00f3347-ff36-4317-9171-67fc9f89f394" />

Действительно, видим что наш uid передается в виде cookie. Я изменил uid на 75, получил статус 200 и вывод страницы. Попробуем перечислить 300 пользователей, может найдем что-нибудь интересное. Перечисление ничего не дало везде одинаковые данные. Тогда вернемся к смене пароля и наконец попробуем его изменить. 


Решил перейти в браузер ZAPа, потому что в обычном фаерфоксе были какие-то траблы с обновлением пароля, либо определена только функция, а кнопка для виду, либо косяк с браузером. Так или иначе перезагрузил сервер задания, снова залогинился.
		Словил такой запрос когда логинился.
<img width="638" height="311" alt="image" src="https://github.com/user-attachments/assets/1646133a-b65b-4668-a98c-0f5a49da6b45" />

<img width="314" height="431" alt="image" src="https://github.com/user-attachments/assets/4447be1d-d152-4922-afe9-7848b86c4891" />

И тут наконец прогрузилось имя профиля, а также за бесконечными ожиданиями ответа от всех сайтов со стилями, перехватился такой интересный запрос.

<img width="816" height="275" alt="image" src="https://github.com/user-attachments/assets/dc028260-ed06-4aed-a28d-fff85b500419" />

Собственно решив изменить uid и / на 75, получил такой ответ. То есть успешно получилось сменить пользователя. Значит можем вернуться к гипотезе о перечислении пользователей.

<img width="945" height="334" alt="image" src="https://github.com/user-attachments/assets/831a0a79-e903-4d8c-944d-ed0fb7bad311" />

После этого, да пользователь сменился.

<img width="307" height="422" alt="image" src="https://github.com/user-attachments/assets/21bc270a-f3e3-402b-a84d-b08b0425c743" />

Теперь пробуем перечислить остальных.
Пользователей всего 100, но ничего потенциально нужно нет.
Пробую сбросить пароль, на этот раз книпка жмется и перехватываются следующие запросы:

<img width="762" height="276" alt="image" src="https://github.com/user-attachments/assets/6b89a15c-92ec-4f00-9e76-32a27eba749c" />

<img width="454" height="215" alt="image" src="https://github.com/user-attachments/assets/28316b2b-d57f-4088-97a7-4153cc9b55bb" />


<img width="763" height="336" alt="image" src="https://github.com/user-attachments/assets/0691e72b-a65d-48c7-b5e5-fc1f88fd3ced" />

<img width="469" height="213" alt="image" src="https://github.com/user-attachments/assets/e1f5b973-d715-4fcd-b65f-80fcfd2c15b4" />


Пробуем подменить токен, чтобы изменить пароль для другого пользователя.
Просто изменить на `/50`  не получилось, ошибка 'access denied'  и токен для 74 пользователя, попробуем с подменой куки.
`{"token":"e51a85aa-17ac-11ec-8e4f-17183faa5d02"}`
С подменной куки и токен тоже изменился,  но пользователь был попрежнему 74, поэтому новая ошибка `invalid token`, значит попробуем его декэшировать - не получилось.

В любом другом случае ловим ошибку `access denied`, но если меняем через метод `GET`, то все успешно и мы сменили пароль для 50 пользователя. Попробуем залогинится, так как мы знаем его логин из перечисления, а пароль только что изменили.

Просматривая перечисленных пользователей, наткнулся на админа - `uid=52`

<img width="943" height="278" alt="image" src="https://github.com/user-attachments/assets/eb8ba4e6-67d7-4d4d-8fb7-9ab372193091" />

`username:"a.corrales"`

Если просто подменить запрос `/api/token/52`, то хоть имя пользователя и измениться, но доступа к нему в профиль у нас не будет. Поэтому меняем пароль для профиля админа по уже отработанной стратегии, и логинимся.

<img width="1036" height="692" alt="image" src="https://github.com/user-attachments/assets/f28760f1-beb2-4bcc-82cc-7904b3dbd117" />

Видим, что у нас добавилась новая функция, добавить событие, также проверим нет ли чего нового в настройках - там все та же смена пароля. Пробуем понять как через добавление события, мы сможем получить доступ к системе.  Нажав нас переадресует на`/event.php` и мы видим следующее

<img width="343" height="354" alt="image" src="https://github.com/user-attachments/assets/db68b24b-bc09-4b73-b122-0e23becc61a0" />

Создадим тестовое событие и перехватим запрос.

<img width="612" height="440" alt="image" src="https://github.com/user-attachments/assets/f159d13e-b39d-480e-b51d-470d4ba54f72" />

Нас переадресует на `/addEvent.php` и мы видим возможность для `XXE`.

<img width="478" height="312" alt="image" src="https://github.com/user-attachments/assets/4b8ef9cb-b969-47e3-97e9-85870dddbca4" />

Поле `name` выводит нам текст, поэтому пробуем через него определить тип нашей уязвимости.

<img width="1642" height="429" alt="image" src="https://github.com/user-attachments/assets/d10cda4c-59f8-406a-a1dc-4dc623a22fab" />

<img width="1634" height="427" alt="image" src="https://github.com/user-attachments/assets/52f77406-cc2b-4da4-b94b-f1334eae732b" />


Теперь нам нужно попробовать прочитать любой файл .php , это будет `index.php` -  успешно получается, значит мы можем пробовать читать наш флаг.
Используем простой пейлоад:
<!DOCTYPE email [
  <!ENTITY company SYSTEM  "php://filter/convert.base64-encode/resource=/flag.php">
]>
...
<name>&company;</name>

<img width="607" height="255" alt="image" src="https://github.com/user-attachments/assets/eefb4d65-f7b0-4864-a80f-9875cfb497fc" />

и получим b64 строку: PD9waHAgJGZsYWcgPSAiSFRCe200NTczcl93M2JfNDc3NGNrM3J9IjsgPz4K

<img width="801" height="581" alt="image" src="https://github.com/user-attachments/assets/e905d7db-3225-40a8-9bd9-a1f0f79be7e5" />

декодим её и получаем флаг: <?php $flag = "HTB{m4573r_w3b_4774ck3r}"; ?>
<img width="801" height="581" alt="image" src="https://github.com/user-attachments/assets/9a77052a-12d3-4acc-9320-61fa152f3095" />
