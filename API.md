# <p align="center"> **API PROSTOR ver.1.07-1** </p>

---
1. [Авторизация. Получение токена доступа](#title1)
2. [Получение баланса бонусов и контактных данных участника бонусной программы «Prostor Club» по номеру мобильного телефона](#title2)
3. [Добавление участников программы лояльности «Prostor Club»](#title3)
4. [Редактирование данных участников программы лояльности «Prostor Club»](#title4)
5. [Получение покупок клиента за выбранный период](#title5)
6. [Получение последних покупок клиента](#title6)
7. [Детализация чека](#title7)
8. [Получение списка работающих магазинов](#title8)
9.  [Получение цены товара](#title9)
10. [Получение заказов клиента](#title10)
11. [Процессинг покупок и возвратов](#title11)
12. Получение населенных пунктов и отделений служб доставки
13. Получение истории Online заказов (на сайте)
14. Получение истории Offline заказов (в магазине)  
15. Подарочные сертификаты
16. Сервис опросов
17. Сервис отправки Push
18. Начисление бонусов
---
### Versions
Date| Service| Change| Version
------------ | ------------- | ------------- | -------------
24.04.2024 | Авторизация | Добавлены разделы 2.3, 2.4 | V1_07-0
18.06.2024 | Подарочные сертификаты | Новый раздел 16 | V1_07-1
20.09.2024 | Опросы | Новый раздел 17 | V1_07-1
22.09.2024 | Отправка Push | Новый раздел 18 | V1_07-1
---
## <a id="title1">1. Авторизация. Получение токена доступа</a>
Для работы с сервисом авторизации в GET запросе используется Basic авторизация.
(Получить код авторизации необходимо у ответственного сотрудника компании «Prostor»)

`GET`
```http
https://api.prostor.ua/api/hs/AuthService/v1/login
```
### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value”. <br> Например: Basic YWhhbWlsdG9uQGFwaWdlZS5jb206bXlwYXNzdzByZAo 
papiauth | `string(36)` | **Для клиента мобильного приложения**. <br> Ключ, который использовался при ОТП авторизации конкретного пользователя в заголовке papiauth. Предназначен для идентификации клиента.


### Response (Success 200)
Name| Type| Discription
------------ | ------------ | ------------
Success | `boolean` | Результат запроса. <br> “true” - успешный запрос <br> “false” - ошибка
Response/KEY | `string(36)` | Ключ доступа к сервису получения баланса (ограничен по времени)
Error | `string` | Описание ошибки

**Пример запроса/ответа сервиса:**

*Запрос:*
```json
https://api.prostor.ua/api/hs/AuthService
    "Host": "api.prostor.ua",
    "Cache-Control": "no-cache",
    "Connection": "close",
    "Accept-Encoding": "gzip, deflate, br",
    "Accept": "*/*",
    "Authorization": "Basic YWhhbWlsdG9uQGFwaWdlZS5jb206bXlwYXNzdzByZAo ",
    "Content-Length": "0"
```
*Ответ:*
```json
    "Content-type": "application/json; charset=utf-8"
{
    "success": "true",
    "response": {
        "KEY": "f6595522-dfa6-407b-b15f-7cb4ae18ebbb"
    },
    "error": ""
}
```
*Пример запроса с использованием CURL:*
```json
curl --location --request GET 'https://api.prostor.ua/api/hs/AuthService/v1/login' \
--header 'Authorization: Basic cG9saWNhcmQ6S3JlY2hldG92aWNo' \  --data-raw
```
---
## 1.1 Верификация номера телефона клиента
В ответе вы получите `“token_verification”`, который необходимо передавать в теле запроса сервисов требующих верификацию. 
Клиенту должен поступить звонок или смс. Клиент должен сообщить последние 4 цифры номера, с которого поступил звонок или полученный код в смс.

`GET`
```json
https://api.prostor.ua/api/hs/AuthService/v1/ValidateContact/{phone_number}/{verification method}
```
### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации
Cookie | `string` | Server ID полученный в ответе авторизации. <br> Пример: SERVERID=cl-api-01\|X62L6\|X62L4

### Request parameters
Name| Type| Discription
------------ | ------------ | ------------
phone_number | `string` | Номер телефона клиента в формате 38ХХХХХХХХХХ 
verification_method | `string` | Метод верификации. Допустимые значения: <br> *callback* - обратный звонок <br> *sms* -  OTP sms

### Response (Success 200)
Name| Type| Discription
------------ | ------------ | ------------
Success | `boolean` | Результат запроса. <br> “true” - успешный запрос <br> “false” - ошибка
Response | `string()` | Описание ответа. Если проверка пройдена “Passed” или “data not verified”, если не пройдена
Error | `string()` | Описание ошибки: <br> **“Parse Error”** – параметр передан в неверном формате <br> **“Required Parameter Not Specified”** – не передан обязательный параметр <br> **“Token_invalid”** – в заголовке передан неправильный ключ или действие ключа истекло, код ответа в данном случае 401

**Пример ответа сервиса:**
```json
{
    "success": "true",
    "response": {
        "token_verification": "53758513-05ca-4f2e-bd1c-2acac37734bf"
    },
    "error": ""
}
```
---
## 1.2 Сервис проверки кода верификации
Сервис предназначен для проверки кода верификации  
`POST`
```json
https://api.prostor.ua/api/hs/AuthService/v1/check/otp
```
### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации
Cookie | `string` | Server ID полученный в ответе авторизации. <br> Пример: SERVERID=cl-api-01\|X62L6\|X62L4

### Request parameters
Name| Type| Discription
------------ | ------------ | ------------
phone_number | `string` | Номер телефона клиента в формате 38ХХХХХХХХХХ 
code_verification | `string` | Код полученный от клиента
token_verification | `string` | Токен верификации полученный в ответе на запрос верификации номера телефона

**Пример запроса:**
```json
{
    "MobileNumber": "380977777777",
    "code_verification": "610422",
    "token_verification": "416a1705-c180-40e6-bf91-b891e93d"
}
```
**Ответ:**
```json
{
    "success": "true",
    "response": "Passed",
    "error": ""
}
```
---
## 1.3 Сервис записи данных для PUSH уведомлений
Сервис предназначен для записи данных устройства пользователя  
`POST`
```json
https://api.prostor.ua/api/hs/AuthService/v1/push_credentials
```
### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации

### Request parameters
Name| Type| Discription
------------ | ------------ | ------------
MobileNumber | `string` | Номер телефона клиента в формате 38ХХХХХХХХХХ 
DeviceId | `string` | Идентификатор устройства
FireBaseToken | `string` | Токен FireBase

**Пример запроса:**
```json
{
	"MobileNumber": "38073XXXXXXX",
	"DeviceId": "DeviceId",
	"FireBaseToken": "FireBaseToken"
}
```
**Ответ:**
```json
{
    "success": "true",
    "response": "data recorded",
    "error": ""
}
```
---
## 1.4 Сервис удаления данных для PUSH уведомлений
Сервис предназначен для записи данных устройства пользователя  
`DELETE`  
```json
https://api.prostor.ua/api/hs/AuthService/v1/push_credentials
```
### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации

### Request parameters
Name| Type| Discription
------------ | ------------ | ------------
MobileNumber | `string` | Номер телефона клиента в формате 38ХХХХХХХХХХ 
DeviceId | `string` | Идентификатор устройства
FireBaseToken | `string` | Токен FireBase

**Пример запроса:**
```json
{
"MobileNumber": "38073XXXXXXX",
	"DeviceId": "DeviceId",
	"FireBaseToken": "FireBaseToken"
}

```
**Ответ:**
```json
{
    "success": "true",
    "response": "data deleted",
    "error": ""
}
```
---
## <a id="title2"> 2. Сервис получения баланса бонусов и контактных данных участника бонусной программы «Prostor Club» по номеру мобильного телефона </a>
`POST`  
```json
https://api.prostor.ua/api/hs/BalanceInfo/v1/uplid
```
### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации
Cookie | `string` | Server ID полученный в ответе авторизации. <br> Пример: SERVERID=cl-api-01\|X62L6\|X62L4

### Request body
Name| Type| Discription
------------ | ------------ | ------------
PhoneNumber | `string(13)` | Номер телефона клиента в формате 38ХХХХХХХХХХ 

### Response (Success 200)
Name| Type| Discription
------------ | ------------ | ------------
Success | `boolean` | Результат запроса: <br> “true” - успешный запрос <br> “false” - ошибка
**Response:** | `string()` | Ответ: 
&nbsp;&nbsp;&nbsp;PhoneNumber | `string(13)` | Номер телефона запрашиваемого клиента
&nbsp;&nbsp;&nbsp;BonusAmount | `float` | Ответ: Количество активных бонусов на счету клиента в грн.
&nbsp;&nbsp;&nbsp;CardNumber | `string(13)` | Номер карты лояльности клиента
&nbsp;&nbsp;&nbsp;Name | `string()` | Имя клиента
&nbsp;&nbsp;&nbsp;Birthdate | `string()` | Дата рождения клиента
&nbsp;&nbsp;&nbsp;Email | `string()` | Почта клиента
Error | `string()` | Описание ошибки: <br> *“Parse Error”* – параметр передан в неверном формате <br> *“Required Parameter Not Specified”* – не передан обязательный параметр <br> *“token_invalid”* – в заголовке передан неправильный ключ или действие ключа истекло, код ответа в данном случае 401.

**Пример запроса:**
```json
https://api.prostor.ua/api/hs/BalanceInfo
{
    "Accept": "*/*",
    "papiauth": "4f88299d-7383-41bb-9868-b9a59758da9e",
    "Authorization": "Basic YWhhbWlsdG9uQGFwaWdlZS5jb206bXlwYXNzdzByZAo ",
    "Content-Type": "application/json",
    "Host": "api.prostor.ua",
    "Cache-Control": "no-cache",
    "Accept-Encoding": "gzip, deflate, br",
    "Connection": "keep-alive",
    "Content-Length": "36"
}
Тело: 
{
    "PhoneNumber": "380939055895"
}
```
**Ответ:**
```json
{
    "Content-type": "application/json;  charset=utf-8"
}
```
**Body:**
```json 
{
    "success": "true",
    "response": {
        "PhoneNumber": "380999999999",
        "CardNumber": "2509999999993",
        "BonusAmount": 92.9,
        "Name": "Иванов Артем Юрьевич",
        "Birthdate": "1985-07-21",
        "Email": "te3715@mail.com"
    },
    "error": ""
}
```
**Пример запроса с использованием CURL:**
```
curl --location --request POST 'https://api.prostor.ua/api/hs/BalanceInfo/v1/uplid/' \
--header 'papiauth: 3fbc75c3-9064-43e7-be21-63740fbe2cdb' \
--header 'Authorization: Basic cG9saWNhcmQ6S3JlY2hldG92aWNo' \
--header 'Content-Type: application/json' \
--header 'Cookie: SERVERID=cl-api-01|X62L6|X62L4' \
--data-raw '{
	"PhoneNumber": "380999999999"
}'
```

## 2.1 Сервис получения расширенных данных баланса бонусов и контактных данных участника бонусной программы «Prostor Club» по номеру телефона

`POST`
```json
https://api.prostor.ua/api/hs/BalanceInfo/v2/uplid
```
### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации
Cookie | `string` | Server ID полученный в ответе авторизации. <br> Пример: SERVERID=cl-api-01\|X62L6\|X62L4

### Request body
Name| Type| Discription
------------ | ------------ | ------------
PhoneNumber | `string(13)` | Номер телефона клиента в формате 38ХХХХХХХХХХ

### Коллекция CardInfo:
***Параметры с данными об остатках бонусов BonusesInfo:***
Name| Type| Discription
------------ | ------------ | ------------
BonusTypeCode | `string` | Код типа бонусов (напр. денежные, статусные)
BonusStatusCode | `string` | Код состояния бонусов (напр. активные, неактивные, сгоревшие)
BonusAmount | `float` | Сумма начисленных бонусов (в копейках)
**CancellationDateInfo:** |   | Коллекция с данными о сроках действия бонусов:  
&nbsp;&nbsp;&nbsp;BonusAmount | `float` | Сумма бонусов (в копейках)
&nbsp;&nbsp;&nbsp;CancellationDate | `date` | Дата сгорания бонусов
**BlockingDateInfo:** |   | Массив с данными о блокировках бонусов  
&nbsp;&nbsp;&nbsp;BonusAmount | `float` | Сумма бонусов (в копейках)
&nbsp;&nbsp;&nbsp;BlockingDate | `date` | Дата блокировки бонусов
&nbsp;&nbsp;&nbsp;PreprocessingId | `string` | Уникальный идентификатор препроцессинга покупки, которая заблокировала бонусы
&nbsp;&nbsp;&nbsp;TISGuid | `string` | Уникальный номер покупки (GUID в учетной системе), которая заблокировала бонусы

***Состояние карты***
Name| Type| Discription
------------ | ------------ | ------------
CardStatus | `string` | Состояние карты (напр. Заблокирована, Зарегистрирована, Не активирована, Активирована…)
ActivatedOn | `date` | Дата активации карты
CardNumber | `string(13)` | Номер карты лояльности клиента
TotalActiveBonusAmount | `string` | Общая сумма активных бонусов (в копейках)

***Параметры с данными о купонах клиента CouponsInfo:***
Name| Type| Discription
------------ | ------------ | ------------
Number | `string` | Номер выданного купона
ExpiredOn | `date` | Дата до которой действует купон
ActivatedOn | `date` | Дата выдачи купона
StatusName | `string` | Статус купона

### Коллекция ContactInfo:
***Блок параметров с основной информации о контакте:***
Name| Type| Discription
------------ | ------------ | ------------
Name | `string` | Контакт. ФИО <br> В данном параметре передается информация ФИО контакта.
Email | `string` | Средства связи. <br> Email В данном параметре передается информация об адресе электронной почты контакта.
MobileNumber | `string` | Средства связи. Мобильный телефон <br> В данном параметре передается информация с указанием номера мобильного телефона контакта
Gender | `int (0, 1 or 2)` | Контакт. Пол <br> В данном параметре передается информация о поле контакта: <br> Значение параметра 0 – Контакт. Пол не заполнено. <br> Значение параметра 1 –Контакт. Пол= Мужской. <br> Значение параметра 2 –Контакт. Пол= Женский.
Sex | `string` | Контакт. Пол <br> В данном параметре передается информация о поле контакта. Значения: <br>  - Мужской <br>  - Женский
Birthdate | `date` | Контакт. Дата рождения <br> В данном параметре передается информация о дате рождения контакта.
ContactId | `string` | Контакт. Id <br> В данном параметре передается уникальный идентификатор записи контакта в БД. Параметр не отображается пользователю на сайте.
SourceCode | `int` | Идентификатор источника добавления контакта. <br> 1 - Личный кабинет <br> 2 - Мобильное приложение <br> 3 - СРМ <br> 4 - Касса
SourceName | `string` | Расшифровка источника добавления контакта

***Блок параметров с информацией о предпочитаемых каналах коммуникаций с контактом:***
Name| Type| Discription
------------ | ------------ | ------------
UseSMS | `int (0 or 1)` | Контакт.Каналы коммуникации. Не использовать SMS. <br> Если значение параметра 0 – Каналы коммуникации. Не использовать SMS = Ложь. <br> Если значение параметра 1 Каналы коммуникации. Не использовать SMS = Истина
UseEmail | `int (0 or 1)` | Контакт.Каналы коммуникации. Не использовать Email <br> Если значение параметра 0 – Каналы коммуникации. Не использовать Email = Ложь. <br> Если значение параметра 1 – Каналы коммуникации. Не использовать Email = Истина.

***Блок параметров «ContactAdress»:***  
В виде дополнительной коллекции передаётся информация об адресе контакта:
Name| Type| Discription
------------ | ------------ | ------------
City | `string` | Адрес контакта. Город <br> В данном параметре передается информация о городе контакта.
Address | `string` | Адрес контакта. Адрес <br> В данном параметре передается информация об адресе контакта
Index | `string` | Адрес контакта. Индекс <br> В данном параметре передается информация об индексе контакта.

**Пример запроса:**
```json
{
	"PhoneNumber": "380999999999"
}
```
**Ответ:**
```json
{
    "success": "true",
    "response": {
        "CardInfo": {
            "CouponsInfo": [],
            "BonusesInfo": [
                {
                    "BonusTypeCode": "Money",
                    "BonusStatusCode": "Burnt",
                    "BonusAmount": 75241
                },
                {
                    "BonusTypeCode": "Сорри-бонусы",
                    "BonusStatusCode": "Burnt",
                    "BonusAmount": 6600
                },
                {
                    "BonusTypeCode": "Бонусы 50%",
                    "BonusStatusCode": "Burnt",
                    "BonusAmount": 10120
                },
                {
                    "BonusTypeCode": "Бонусы 15%",
                    "BonusStatusCode": "Burnt",
                    "BonusAmount": 10040
                },
                {
                    "BonusTypeCode": "Бонусы 15%",
                    "BonusStatusCode": "Active",
                    "BonusAmount": 36390,
                    "CancellationDateInfo": [
                        {
                            "BonusAmount": 27700,
                            "CancellationDate": "2023-07-20T00:00:00.0000000"
                        },
                        {
                            "BonusAmount": 8690,
                            "CancellationDate": "2023-08-05T00:00:00.0000000"
                        }
                    ]
                },
                {
                    "BonusTypeCode": "Бонусы 15%",
                    "BonusStatusCode": "Blocked",
                    "BonusAmount": 215,
                    "BlockingDateInfo": [
                        {
                            "BonusAmount": 215,
                            "BlockingDate": "2023-07-17T10:14:23.0000000",
                            "PreprocessingId": "3a0c7584-e89e-783b-1835-58c125361703",
                            "TISGuid": "ac0de5c2-af3b-4cfa-aadb-82571d148847"
                        }
                    ]
                },
                {
                    "BonusTypeCode": "Бонусы 15%",
                    "BonusStatusCode": "Blocked",
                    "BonusAmount": 42,
                    "BlockingDateInfo": [
                        {
                            "BonusAmount": 42,
                            "BlockingDate": "2023-07-17T10:14:23.0000000",
                            "PreprocessingId": "3a0c7584-e89e-783b-1835-58c125361703",
                            "TISGuid": "ac0de5c2-af3b-4cfa-aadb-82571d148847"
                        }
                    ]
                },
                {
                    "BonusTypeCode": "Бонусы 15%",
                    "BonusStatusCode": "Blocked",
                    "BonusAmount": 43,
                    "BlockingDateInfo": [
                        {
                            "BonusAmount": 43,
                            "BlockingDate": "2023-07-17T10:14:23.0000000",
                            "PreprocessingId": "3a0c7584-e89e-783b-1835-58c125361703",
                            "TISGuid": "ac0de5c2-af3b-4cfa-aadb-82571d148847"
                        }
                    ]
                }
            ],
            "ActivatedOn": "2020-07-15T15:28:38.0000000+03:00",
            "CardStatus": "Активирована",
            "TotalActiveBonusAmount": "36390",
            "CardNumber": "2509793637153",
            "Info": "OK"
        },
        "ContactInfo": {
            "MobileNumber": "380999999999",
            "ContactId": "6fd927f3-8fea-4cfc-9784-b115c706955c",
            "Name": "Иванов Артем Юрьевич",
            "Email": "arti4ik@gmail.com",
            "IsRefer": false,
            "Sex": "Мужской",
            "Birthdate": null,
            "CommunicationChannels": {
                "UseSMS": false,
                "UseEmail": false,
                "AdsByEmail": false,
                "AdsBySMS": false,
                "GetCatalogs": false
            },
            "ContactAdress": [
                {
                    "Index": "49000",
                    "Region": null,
                    "City": "Днепр",
                    "Type": "Домашний",
                    "Address": null
                }
            ],
            "FavoriteCategoryList": [],
            "Info": "OK"
        }
    },
    "error": ""
}
```

---
## 2.2 Сервис получения купонов клиента по номеру телефона
`POST`
```json
https://api.prostor.ua/api/hs/BalanceInfo/v1/ContactCoupons/
```

### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации
Cookie | `string` | Server ID полученный в ответе авторизации. <br> Пример: SERVERID=cl-api-01\|X62L6\|X62L4

### Request body
Name| Type| Discription
------------ | ------------ | ------------
MobileNumber | `string(13)` | Номер телефона клиента в формате 38ХХХХХХХХХХ

### Response (Success 200)
Name| Type| Discription
------------ | ------------ | ------------
Number | `string` | Номер выданного купона
ExpiredOn | `date` | Дата до которой действует купон
ActivatedOn | `date` | Дата выдачи купона
StatusName | `string` | Статус купона
CategoryName | `string` | Категория купона
Id | `string` | Идентификатор купона
**ActiveCampaings:** | `array` | Данные акции купона
&nbsp;&nbsp;&nbsp;Name | `string` | Наименование акции
&nbsp;&nbsp;&nbsp;Description | `string` | Описание акции
&nbsp;&nbsp;&nbsp;Code | `string` | Номер акции
&nbsp;&nbsp;&nbsp;CampaingId | `string` | Идентификатор акции

**Пример запроса:**
```json
{
	"MobileNumber": "380977777777"
}
```
**Ответ:**
```json
{
    "error": "",
    "response": {
        "ContactCoupons": [
            {
                "StatusName": "Выдана",
                "Name": "2100000003907",
                "Id": "9fd38ef0-6504-4a4f-a0ac-a19fb63e074b",
                "ExpiredDate": "08.04.2024 0:00:00",
                "CategoryName": "Купон (Процессинговый)",
                "ActiveCampaings": [
                    {
                        "Name": "ЗНИЖКА 10% на наступну покупку_использование",
                        "Description": "ЗНИЖКА 10% на наступну покупку_использование",
                        "Code": "848",
                        "CampaingId": "1a29b6f3-1f3b-46df-8958-48fa3d1fc88e"
                    }
                ],
                "ActivatedOn": "25.03.2024 13:13:31"
            }
        ]
    },
    "success": "true"
}
```
---

## 3. <a id="title3"> Сервис добавления участников программы лояльности «Prostor Club» </a>
`POST`
```json
https://api.prostor.ua/api/hs/AddContact/v1/addupl/
```
Пользователи с неавторизованными клиентами должны использовать:
```json
https://api.prostor.ua/api/hs/AddContact/v1/addupl_withauth/
```

### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации
Cookie | `string` | Server ID полученный в ответе авторизации. <br> Пример: SERVERID=cl-api-01\|X62L6\|X62L4

### Request body
Обязательные параметры отмечены <mark>цветом</mark>. Пустые параметры передавать не нужно.
Name| Type| Discription
------------ | ------------ | ------------
<mark>MobileNumber</mark> | `string(50)` | Номер телефона клиента в формате 38ХХХХХХХХХХ 
<mark>token_verification</mark> | `string(36)` | Токен верификации номера телефона полученный в ответе сервиса верификации номера телефона клиента. **Для пользователей с неавторизованными клиентами**
<mark>code_verification</mark> | `string(4)` | 4 цифры верификации, которые сообщил клиент (см. описание сервиса верификации). **Для пользователей с неавторизованными клиентами**
Name | `string(100)` | Контакт. ФИО <br> В данном параметре необходимо передавать  Фамилию Имя Отчество
Email | `string(100)` | Средства связи. Email. <br> В данном параметре необходимо передавать адрес электронной почты контакта.
Gender | `int (0,1 or 2)` | Контакт. Пол. <br> Если значение параметра 0 – не заполнять данное поле. <br> Если значение параметра 1 – устанавливать значение по полю Контакт. Пол= Мужской. <br> Если значение параметра 2 – устанавливать значение по полю Контакт. Пол= Женский.
Birthdate | `date` | Контакт. Дата рождения. <br> Дату передавать в формате ISO 8601: YYYY-MM-DD
UseSMS | `int (0 or 1)` | Контакт. Каналы коммуникации. Не использовать SMS. <br> Если значение параметра 0 – устанавливать значение по полю Каналы коммуникации. Не использовать SMS = Ложь <br> Если значение параметра 1 – устанавливать значение по полю Каналы коммуникации. Не использовать SMS = Истина.
UseEmail | `int (0 or 1)` | Контакт. ФИО <br> Контакт. Каналы коммуникации. Не использовать Email. <br> Если значение параметра 0 – устанавливать значение по полю Каналы коммуникации. Не использовать Email = Ложь. <br> Если значение параметра 1 – устанавливать значение по полю Каналы коммуникации. Не использовать Email = Истина.
SourceCode | `int` | Контакт. ФИО <br> Идентификатор источника добавления контакта. <br> 1 - Личный кабинет <br> 2 - Мобильное приложение <br> 3 - СРМ <br> 4 - Касса

### Коллекция  «ContactAdress»:
***В виде дополнительной коллекции передаётся информация о адресах контакта:***
Name| Type| Discription
------------ | ------------ | ------------
City | `string(50)` | Адрес контакта. Город 
Region | `string(100)` | Адрес контакта. Область
Address | `string(100)` | Адрес контакта. Адрес <br> В данном параметре необходимо передавать значение полей, через запятую: <br> Название улицы, номер дома, номер корпуса, номер квартиры.
Index | `string(50)` | Адрес контакта. Индекс

### Response (Success 200)
Name| Type| Discription
------------ | ------------ | ------------
Success | `boolean` | Результат запроса. <br> “true” - успешный запрос <br> “false” - ошибка
**Response:** | `string` | Ответ
&nbsp;&nbsp;&nbsp;Description | `string` | Описание результата работы сервиса
&nbsp;&nbsp;&nbsp;**Result:** | `string` |  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PhoneNumber | `string(13)` | Номер телефона клиента
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NewCardNumber | `string(13)` | Номер карты лояльности клиента
Error | `string()` | Описание ошибки. <br> **“Parse Error”** – параметр передан в неверном формате <br> **“Required Parameter Not Specified”** – не передан обязательный параметр <br> **“token_invalid”** – в заголовке передан неправильный ключ или действие ключа истекло, код ответа в данном случае 401. <br> **“ContactPhoneAlreadyExists”** - Контакт с параметром **MobileNumber** уже существует в системе. <br> **“EmailAlreadyExists”** - Контакт с параметром Email уже существует в системе.

**Пример запроса:**
```json
https://api.prostor.ua/api/hs/AddContact/v1/addupl/
```
**Header:**
```json
{
    "Accept": "*/*",
    "papiauth": "9cec856a-6c8b-4f27-86c7-f83feeebba76",
    "Authorization": "Basic YWhhbWlsdG9uQGFwaWdlZS5jb206bXlwYXNzdzByZAo",
    "Content-Type": "application/json",
    "Host": " "api.prostor.ua",
    "Cache-Control": "no-cache",
    "Accept-Encoding": "gzip, deflate, br",
    "Connection": "keep-alive",
    "Content-Length": "295"
}
```
**Body:**
```json
{
    "Name": "Новый Edit contact",
    "Gender": 1,
    "Birthdate": "03.03.1986",
    "MobileNumber": "3805237730930",
    "Email": "edit_38052353651cpq@mail.ua",
    "ContactAdress": {
        "City": "Киев",
        "Address": "Улица, 17",
        "Index": "49005"
    }
}
```
**Ответ:**
```json
{
    "success": "true",
    "response": {
        "Description": "OK Данные успешно добавлены.",
        "Result": {
            "NewCardNumber": "2505237730931",
            "MobileNumber": "3805237730930"
        }
    },
    "error": ""
}
```
**Пример запроса с использованием CURL:**
```
curl --location --request POST 'https://api.prostor.ua/api/hs/AddContact/v1/addupl/' \
--header 'papiauth: 9cec856a-6c8b-4f27-86c7-f83feeebba76' \
--header 'Authorization: Basic YWhhbWlsdG9uQGFwaWdlZS5jb206bXlwYXNzdzByZAo' \
--header 'Content-Type: application/json' \
--data-raw '{
    "Name": "Новый Edit contact",
    "Gender": 1,
    "Birthdate": "03.03.1986",
    "MobileNumber": "3805237730930",
    "Email": "edit_38052353651cpq@mail.ua",
    	"ContactAdress": {
    	"City": "Киев",
    	"Address": "Улица, 17",
    	"Index": "49005"
    }
 } '
```
---

## <a id="title4"> 4. Сервис редактирования данных участников программы лояльности «Prostor Club» </a>
`POST`
```json
https://api.prostor.ua/api/hs/EditContactInfo/v1/edituplinfo/
```
Пользователи с неавторизованными клиентами должны использовать:
```json
https://api.prostor.ua/api/hs/EditContactInfo/v1/edituplinfo_withauth/
```

### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации
Cookie | `string` | Server ID полученный в ответе авторизации. <br> Пример: SERVERID=cl-api-01\|X62L6\|X62L4

### Request body
Обязательные параметры отмечены <mark>цветом</mark>. Пустые параметры передавать не нужно.
Name| Type| Discription
------------ | ------------ | ------------
<mark>MobileNumber</mark> | `string(50)` | Номер телефона клиента в формате 38ХХХХХХХХХХ 
<mark>token_verification</mark> | `string(36)` | Токен верификации номера телефона полученный в ответе сервиса верификации номера телефона клиента. **Для пользователей с неавторизованными клиентами**
<mark>code_verification</mark> | `string(4)` | 4 цифры верификации, которые сообщил клиент (см. описание сервиса верификации). **Для пользователей с неавторизованными клиентами**
Name | `string(100)` | Контакт. ФИО <br> В данном параметре необходимо передавать  Фамилию Имя Отчество
Email | `string(100)` | Средства связи. Email. <br> В данном параметре необходимо передавать адрес электронной почты контакта.
Gender | `int (0,1 or 2)` | Контакт. Пол. <br> Если значение параметра 0 – не заполнять данное поле. <br> Если значение параметра 1 – устанавливать значение по полю Контакт. Пол= Мужской. <br> Если значение параметра 2 – устанавливать значение по полю Контакт. Пол= Женский.
Birthdate | `date` | Контакт. Дата рождения. <br> Дату передавать в формате ISO 8601: YYYY-MM-DD
UseSMS | `int (0 or 1)` | Контакт. Каналы коммуникации. Не использовать SMS. <br> Если значение параметра 0 – устанавливать значение по полю Каналы коммуникации. Не использовать SMS = Ложь <br> Если значение параметра 1 – устанавливать значение по полю Каналы коммуникации. Не использовать SMS = Истина.
UseEmail | `int (0 or 1)` | Контакт. ФИО <br> Контакт. Каналы коммуникации. Не использовать Email. <br> Если значение параметра 0 – устанавливать значение по полю Каналы коммуникации. Не использовать Email = Ложь. <br> Если значение параметра 1 – устанавливать значение по полю Каналы коммуникации. Не использовать Email = Истина.
SourceCode | `int` | Контакт. ФИО <br> Идентификатор источника добавления контакта. <br> 1 - Личный кабинет <br> 2 - Мобильное приложение <br> 3 - СРМ <br> 4 - Касса

### Коллекция  «ContactAdress»:
***В виде дополнительной коллекции передаётся информация о адресах контакта:***
Name| Type| Discription
------------ | ------------ | ------------
City | `string(50)` | Адрес контакта. Город 
Region | `string(100)` | Адрес контакта. Область
Address | `string(100)` | Адрес контакта. Адрес <br> В данном параметре необходимо передавать значение полей, через запятую: <br> Название улицы, номер дома, номер корпуса, номер квартиры.
Index | `string(50)` | Адрес контакта. Индекс

### Response (Success 200)
Name| Type| Discription
------------ | ------------ | ------------
Success | `boolean` | Результат запроса. <br> “true” - успешный запрос <br> “false” - ошибка
Response: | `string()` | Ответ
Error | `string()` | Описание ошибки. <br> **“Parse Error”** – параметр передан в неверном формате <br> **“Required Parameter Not Specified”** – не передан обязательный параметр <br> **“token_invalid”** – в заголовке передан неправильный ключ или действие ключа истекло, код ответа в данном случае 401. <br> **“ContactPhoneAlreadyExists”** - Контакт с параметром **MobileNumber** уже существует в системе. <br> **“EmailAlreadyExists”** - Контакт с параметром Email уже существует в системе. <br> **“ContactCannotFind”** - Не найден контакт по передаваемым параметрам.

**Пример запроса:**
```json
https://api.prostor.ua/api/hs/EditContactInfo/v1/edituplinfo/
```
**Header:**
```json
{
    "Accept": "*/*",
    "papiauth": "9d81131f-d05e-4d37-ab95-7094243cc48e",
    "Authorization": "Basic QWRtpc3RyYXRvcjp6YXEx=",
    "Content-Type": "application/json",
    "Host":  "api.prostor.ua",
    "Cache-Control": "no-cache",
    "Accept-Encoding": "gzip, deflate, br",
    "Connection": "keep-alive",
    "Content-Length": "288"
}
```
**Body:**
```json
{
    "Name": "Новый Edit contact",
    "Gender": 1,
    "Birthdate": "03.03.1986",
    "MobileNumber": "380523536951",
    "Email": "edit_380523536951@mail.ua",
    "ContactAdress": {
        "City": "",
        "Address": "Улица, 17",
        "Index": "49005"
    }
}
```
**Ответ:**
```json
{
    "success": "true",
    "response": "Данные успешно обновлены.",
    "error": ""
}
```
**Пример запроса с использованием CURL:**
```
curl --location --request POST https://api.prostor.ua/api/hs/AddContact/v1/addupl/' \
--header 'papiauth: 9cec856a-6c8b-4f27-86c7-f83feeebba76' \
--header 'Authorization: Basic cG9saWNhcmQ6emFxMTIz' \
--header 'Content-Type: application/json' \
--data-raw '{
    "Name": "Новый Edit contact",
    "Gender": 1,
    "Birthdate": "03.03.1986",
    "MobileNumber": "3805237730930",
    "Email": "edit_38052353651cpq@mail.ua",
    	"ContactAdress": {
    	"City": "Киев",
    	"Address": "Улица, 17",
    	"Index": "49005"
    }
 } '
```
---

## <a id="title5"> 5. Сервис получения покупок клиента за выбранный период </a>
`POST`
```json
https://api.prostor.ua/api/hs/PurchaseList/v1/list/
```
### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации
Cookie | `string` | Server ID полученный в ответе авторизации. <br> Пример: SERVERID=cl-api-01\|X62L6\|X62L4

### Request body
Обязательные параметры отмечены <mark>цветом</mark>. Пустые параметры передавать не нужно.
Name| Type| Discription
------------ | ------------ | ------------
<mark>MobileNumber</mark> | `string(50)` | Номер телефона клиента в формате 38ХХХХХХХХХХ 
<mark>StartDate</mark> | `date` | Дата начала периода фильтрации. Дата/Время в формате ISO 8601. <br> YYYY-MM-DDThh:mm:ss
<mark>DueDate</mark> | `date` | Дата завершения периода фильтрации. Дата/Время в формате ISO 8601. <br> YYYY-MM-DDThh:mm:ss

### Server response (Success 200)
Name| Type| Discription
------------ | ------------ | ------------
Success | `boolean` | Результат запроса. <br> “true” - успешный запрос <br> “false” - ошибка
**Response:** | `string()` | Ответ
&nbsp;&nbsp;&nbsp;PuchasesList | `array` | Список найденных покупок клиента
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Date | `date` | Дата покупки
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PurchaseNumber | `string()` | Номер чека
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IsReturn | `boolean` | Признак чека возврата
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PointOfSale | `string()` | Наименование торговой точки
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CodeStore | `string()` | Уникальный код торговой точки
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TotalAmount | `float()` | Сумма чека
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AccruedBonuses | `float()` | Начисленные бонусы
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PaidByBonuses | `float()` | Оплачено бонусами
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TotalDiscount | `float()` | Сумма скидки
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CardNumber | `string()` | Номер карты лояльности
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IsOffLinePurchase | `boolean` | true - покупка в оффлайне <br> false - онлайн покупка
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tax_url | `string()` | Ссылка на чек, если она есть (https://check.checkbox.ua/{fiscal_number})
Error | `string()` | Описание ошибки. <br> **“Parse Error”** – параметр передан в неверном формате <br> **“Required Parameter Not Specified”** – не передан обязательный параметр <br> **“token_invalid”** – в заголовке передан неправильный ключ или действие ключа истекло, код ответа в данном случае 401. <br> **“OrderNotFound”** - не найдены покупки по карте за указанный период.

**Пример запроса:**
```json
https://api.prostor.ua/api/hs/PurchaseList/v1/list/
```
**Header:**
```json
Заголовки: 
{
    "Accept": "*/*",
    "papiauth": "3b9837d4-1ff9-4994-87fd-50efe25bfbbd",
    "Authorization": "Basic QWRtpc3RyYXRvcjp6YXEx=",
    "Content-Type": "application/json",
    "Host":  "api.prostor.ua",
    "Cache-Control": "no-cache",
    "Accept-Encoding": "gzip, deflate, br",
    "Connection": "keep-alive",
     "Content-Length": "111"
}
```
**Body:**
```json
{
    "MobileNumber": "380999999999",
    "StartDate": "2020-02-05T08:00:42",
    "DueDate": "2020-02-07T18:31:42"
}
```
**Ответ:**
```json
{
    "success": "true",
    "response": {
        "PuchasesList": [
            {
                "Date": "2020-02-06T15:35:09+02:00",
                "PurchaseNumber": "1327",
	"IsReturn": false,
                "PointOfSale": "ПРОСТОР 606 Николаев",
                "Quantity": 1,
                "TotalAmount": 111,
                "AccruedBonuses": 0,
                "PaidByBonuses": -11,
                "CardNumber": "2500044048399"
            }
        ]
    },
    "error": ""
}
```
**Пример запроса с использованием CURL:**
```
curl --location --request POST 'https://api.prostor.ua/api/hs/PurchaseList/v1/list/' \
--header 'papiauth: 3b9837d4-1ff9-4994-87fd-50efe25bfbbd' \
--header 'Authorization: Basic QWRtaW5pc3RyYXRvcjp6YXExMjM=' \
--header 'Content-Type: application/json' \
--data-raw '{
	"MobileNumber": "380999999999",
	"StartDate": "2020-02-05T08:00:42",
	"DueDate": "2020-02-07T18:31:42"
}
```
---

## <a id="title6"> 6. Сервис получения последних покупок клиента </a>
Возвращает список последних совершенных клиентом покупок в заданном количестве (<mark>Quantity</mark>).
`POST`
```json
https://api.prostor.ua/api/hs/PurchaseList/v1/LastPurchase/
```
### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации
Cookie | `string` | Server ID полученный в ответе авторизации. <br> Пример: SERVERID=cl-api-01\|X62L6\|X62L4

### Request body
Обязательные параметры отмечены <mark>цветом</mark>. Пустые параметры передавать не нужно.
Name| Type| Discription
------------ | ------------ | ------------
<mark>MobileNumber</mark> | `string(50)` | Номер телефона клиента в формате 38ХХХХХХХХХХ 
<mark>Quantity</mark> | `string()` | Количество последних покупок
offset | `int` | Смещение выборки
accrual_only | `boolean` | Фильтр покупок по начислению или  списанию бонусов. Если передать true выводятся только покупки в которых было начисление бонусов, если false выводятся покупки только со списанием бонусов. Для вывода всех покупок параметр передавать не нужно.
purchases_with_bonuses | `boolean` | Фильтр покупок, в которых есть движение бонусов. Если передать true выводятся только покупки в которых было начисление бонусов или списание бонусов,  если false выводятся покупки без начислений и  без списаний. **Если параметр передан - параметр accrual_only не обрабатывается!**
StartDate | `date` | Дата начала периода фильтрации. <br> ДатаВремя в формате ISO 8601. YYYY-MM-DDThh:mm:ss
DueDate | `date` | Дата завершения периода фильтрации. <br> ДатаВремя в формате ISO 8601. YYYY-MM-DDThh:mm:ss

### Server response (Success 200)
Name| Type| Discription
------------ | ------------ | ------------
Success | `boolean` | Результат запроса. <br> “true” - успешный запрос <br> “false” - ошибка
Response | `string()` | Ответ
&nbsp;&nbsp;&nbsp;**PuchasesList:** | `array` | Список найденных покупок клиента
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Date | `date` | Дата покупки
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PurchaseNumber | `string()` | Номер чека
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IsReturn | `boolean` | Признак чека возврата
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PointOfSale | `string()` | Наименование торговой точки
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CodeStore | `string()` | Уникальный код торговой точки
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Quantity | `string()` | Количество товара
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TotalAmount | `float()` | Сумма чека
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TotalDiscount | `float()` | Сумма скидки
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AccruedBonuses | `float()` | Начисленные бонусы
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PaidByBonuses | `float()` | Оплачено бонусами
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IsOffLinePurchase | `boolean` | true - покупка в оффлайне <br> false - онлайн покупка
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CardNumber | `string()` | Номер карты лояльности
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tax_url | `string()` | Ссылка на чек, если она есть (https://check.checkbox.ua/{fiscal_number})
&nbsp;&nbsp;&nbsp;**shop_address:** | `array` | Коллекция с адресными данными магазина
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;name | `string()` | Наименование ТТ
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;code | `string()` | Код ТТ
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;address | `string()` | Адрес ТТ
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;number | `string()` | Номер ТТ
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;startTime | `string()` | Время начала работы ТТ
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;endTime | `string()` | Время окончания работы ТТ
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;longitude | `string()` | Долгота
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;latitude | `string()` | Широта
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;shop_closed | `boolean` | Признак закрытого магазина
Error | `string()` | Описание ошибки. <br> **“Parse Error”** – параметр передан в неверном формате <br> **“Required Parameter Not Specified”** – не передан обязательный параметр <br> **“token_invalid”** – в заголовке передан неправильный ключ или действие ключа истекло, код ответа в данном случае 401. <br> **“OrderNotFound”** - не найдены покупки по карте за указанный период.

**Пример запроса:**
```json
https://api.prostor.ua/api/hs/PurchaseList/v1/LastPurchase/
```
**Header:**
```json
{
    "Accept": "*/*",
    "papiauth": "d4b8b220-659f-4e0f-a218-aafc5d33dfc3",
    "Authorization": "Basic QWRtaW5pc3RyYXRvcjp6YXExMjM=",
    "Content-Type": "application/json",
    "Host":  "api.prostor.ua",
    "Cache-Control": "no-cache",
    "Accept-Encoding": "gzip, deflate, br",
    "Connection": "keep-alive",
    "Content-Length": "56"
}
```
**Body:**
```json
{
    "MobileNumber": "380999999999",
    "Quantity": "2"
}
```
**Ответ:**
```json
{
    "success": "true",
    "response": {
        "PuchasesList": [
            {
                "Date": "2020-11-15T16:13:03+02:00",
                "PurchaseNumber": "257953",
	"IsReturn": true,
                "PointOfSale": "ПРОСТОР 544 Харьков",
                "Quantity": 3,
                "TotalAmount": 853,
                "AccruedBonuses": 0,
                "PaidByBonuses": -833,
                "CardNumber": "2500017032311"
            },
            {
                "Date": "2020-10-18T14:30:12+03:00",
                "PurchaseNumber": "126218",
                "PointOfSale": "ПРОСТОР 592 Харків",
                "Quantity": 2,
                "TotalAmount": 818,
                "AccruedBonuses": 0,
                "PaidByBonuses": -817,
                "CardNumber": "2500017032311"
            }
        ]
    },
    "error": ""
}
```
**Пример запроса с использованием CURL:**
```
curl --location --request POST 'https://api.prostor.ua/api/hs/PurchaseList/v1/LastPurchase/' \
--header 'papiauth: d4b8b220-659f-4e0f-a218-aafc5d33dfc3' \
--header 'Authorization: Basic QWRtaW5pc3RyYXRvcjp6YXExMjM=' \
--header 'Content-Type: application/json' \
--data-raw '{
	"MobileNumber": "380662727000",
	"Quantity": "2"}'
```
---

## <a id="title7"> 7. Детализация чека </a>
`POST`
```json
https://api.prostor.ua/api/hs/PurchaseList/v1/detailPurchase/
```
### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации
Cookie | `string` | Server ID полученный в ответе авторизации. <br> Пример: SERVERID=cl-api-01\|X62L6\|X62L4

### Request body
Обязательные параметры отмечены <mark>цветом</mark>. Пустые параметры передавать не нужно.
Name| Type| Discription
------------ | ------------ | ------------
<mark>CardNumber</mark> | `string()` | Номер карты клиента, полученный в результате работы сервиса по получению покупок
<mark>PurchaseNumber</mark> | `string()` | Номер чека покупки, полученный в результате работы сервиса по получению покупок
IsOnLinePurchase | `boolean` | Признак типа покупки: <br> true - покупка совершенная онлайн <br> false - покупка совершенная в стационарном магазине.

### Server response (Success 200)
Name| Type| Discription
------------ | ------------ | ------------
Success | `boolean` | Результат запроса. <br> “true” - успешный запрос <br> “false” - ошибка
Response | `string()` | Ответ
&nbsp;&nbsp;&nbsp;Puchases |  | Найденный чек
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Products:** |  | Коллекция продуктов чека
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Position | `string()` | Номер позиции в чеке
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Code | `boolean` | Код товара
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ProductName | `string()` | Наименование товара
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Price | `float()` | Цена товара
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Quantity | `int()` | Количество товара
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;BonusesPaidAmount | `float()` | Оплачено бонусами
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;BonusesAmount | `float()` | Начислено бонусов
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CashPaidAmount | `float()` | Стоимость товара
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Barcode | `string()` | Штрихкод товара (может отсутствовать, передается null)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Parent | `string()` | Категория товара (может отсутствовать, передается null)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;images_link | `string()` | Ссылка на картинку товара (может отсутствовать, передается null)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;link | `string()` | Ссылка на товар (может отсутствовать, передается null)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Remains | `int()` | Остаток товара на центральном складе
Error | `string()` | Описание ошибки. <br> **“Parse Error”** – параметр передан в неверном формате <br> **“Required Parameter Not Specified”** – не передан обязательный параметр <br> **“token_invalid”** – в заголовке передан неправильный ключ или действие ключа истекло, код ответа в данном случае 401. <br> **“OrderNotFound”** - не найдены покупки по карте за указанный период.

**Пример запроса:**
```json
https://api.prostor.ua/api/hs/PurchaseList/v1/LastPurchase/
```
**Header:**
```json 
{
    "Accept": "*/*",
    "papiauth": "1083d6f1-e6ad-405c-af8b-23c0dd853ab3",
    "Authorization": "Basic QWRtaW5pc3RyYXRvcjp6YXExMjM=",
    "Content-Type": "application/json",
    "Host": "api.prostor.ua",
    "Cache-Control": "no-cache",
    "User-Agent": "PostmanRuntime/7.26.8",
    "Accept-Encoding": "gzip, deflate, br",
    "Connection": "keep-alive",
    "Content-Length": "66"
}
```
**Body:**
```json
{
    "CardNumber": "2500017032318",
    "PurchaseNumber": "248044"
}
```
**Ответ:**
```json
{
    "success": "true",
    "response": {
        "Purchase": {
            "Products": [
                {
                    "Position": 1,
                    "Code": "046266",
                    "ProductName": "Johnson&Johnson шамп. дит. від макушек до п'ят, 500мл",
                    "Price": 92.9,
                    "Quantity": 1,
                    "BonusesPaidAmount": 0,
                    "BonusesAmount": 92.9,
                    "CashPaidAmount": 92.9,
                    "Barcode": null,
                    "Parent": null,
                    "images_link": null,
                    "link": null
                },
                {
                    "Position": 2,
                    "Code": "144862",
                    "ProductName": "Крокси дит. колір в асорт. арт. CH-SM-19-19, 1пара",
                    "Price": 49.99,
                    "Quantity": 1,
                    "BonusesPaidAmount": 0,
                    "BonusesAmount": 60,
                    "CashPaidAmount": 49.99,
                    "Barcode": null,
                    "Parent": null,
                    "images_link": null,
                    "link": null
                },
                {
                    "Position": 3,
                    "Code": "153834",
                    "ProductName": "Бейсболка дит., колір в ассорт., р.52 арт.CH-SM-20-8, 1шт",
                    "Price": 39.99,
                    "Quantity": 1,
                    "BonusesPaidAmount": 0,
                    "BonusesAmount": 48,
                    "CashPaidAmount": 39.99,
                    "Barcode": "2000001538340",
                    "Parent": "Лето / Бейсболки, шляпы",
                    "images_link": "https://prostor.ua/content/images/16/beysbolka-detskaya-r.52-1-sht-59574599413085.jpg",
                    "link": "https://prostor.ua/product/beysbolka-detskaya-r.52-1-sht/"
                },
                {
                    "Position": 4,
                    "Code": "144846",
                    "ProductName": "Бейсболка жін. колір в ассорт. арт. CH-SM-19-3, 1шт",
                    "Price": 39.99,
                    "Quantity": 1,
                    "BonusesPaidAmount": 0,
                    "BonusesAmount": 48,
                    "CashPaidAmount": 39.99,
                    "Barcode": null,
                    "Parent": null,
                    "images_link": null,
                    "link": null
                },
                {
                    "Position": 5,
                    "Code": "153833",
                    "ProductName": "Бейсболка дит., колір в ассорт., р.54 арт.CH-SM-20-7, 1шт",
                    "Price": 39.99,
                    "Quantity": 1,
                    "BonusesPaidAmount": 0,
                    "BonusesAmount": 48,
                    "CashPaidAmount": 39.99,
                    "Barcode": "2000001538333",
                    "Parent": "Лето / Бейсболки, шляпы",
                    "images_link": "https://prostor.ua/content/images/15/beysbolka-detskaya-g.-54-1-sht-79423792552728.jpg",
                    "link": "https://prostor.ua/product/beysbolka-detskaya-g.-54-1-sht/"
                },
                {
                    "Position": 6,
                    "Code": "153834",
                    "ProductName": "Бейсболка дит., колір в ассорт., р.52 арт.CH-SM-20-8, 1шт",
                    "Price": 39.99,
                    "Quantity": 1,
                    "BonusesPaidAmount": 0,
                    "BonusesAmount": 48,
                    "CashPaidAmount": 39.99,
                    "Barcode": "2000001538340",
                    "Parent": "Лето / Бейсболки, шляпы",
                    "images_link": "https://prostor.ua/content/images/16/beysbolka-detskaya-r.52-1-sht-59574599413085.jpg",
                    "link": "https://prostor.ua/product/beysbolka-detskaya-r.52-1-sht/"
                },
                {
                    "Position": 7,
                    "Code": "144852",
                    "ProductName": "Шляпа жін. кремова арт. CH-SM-19-9, 1шт",
                    "Price": 29.99,
                    "Quantity": 1,
                    "BonusesPaidAmount": 0,
                    "BonusesAmount": 36,
                    "CashPaidAmount": 29.99,
                    "Barcode": null,
                    "Parent": null,
                    "images_link": null,
                    "link": null
                }
            ]
        }
    },
    "error": ""
}
```
**Пример запроса с использованием CURL:**
```
curl --location --request POST 'https://api.prostor.ua/api/hs/PurchaseList/v1/detailPurchase/' \
--header 'papiauth: d4b8b220-659f-4e0f-a218-aafc5d33dfc3' \
--header 'Authorization: Basic QWRtaW5pc3RyYXRvcjp6YXExMjM=' \
--header 'Content-Type: application/json' \
--data-raw '{
	"CardNumber": "2500017032318",
	"PurchaseNumber": "248044"
}'
```
---

## <a id="title8"> 8. Сервис получения списка работающих магазинов </a>
`GET`
```json
https://api.prostor.ua/api/hs/prostorbot/v1/GetStores
```
### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации
Cookie | `string` | Server ID полученный в ответе авторизации. <br> Пример: SERVERID=cl-api-01\|X62L6\|X62L4

### Server response (Success 200)
Name| Type| Discription
------------ | ------------ | ------------
Success | `boolean` | Результат запроса. <br> “true” - успешный запрос <br> “false” - ошибка
**Response:** | `string()` | Ответ
&nbsp;&nbsp;&nbsp;КодМагазина | string() | Код магазина в учетной системе
&nbsp;&nbsp;&nbsp;Наименование | string() | Наименование магазина
&nbsp;&nbsp;&nbsp;Город | string() | Город, в котором расположен магазин
&nbsp;&nbsp;&nbsp;Адрес | string() | Адрес магазина
&nbsp;&nbsp;&nbsp;Долгота | string() | Долгота, координаты магазина
&nbsp;&nbsp;&nbsp;Широта | string() | Широта, координаты магазина
Error | string() | Описание ошибки. <br> **“token_invalid”** – в заголовке передан неправильный ключ или действие ключа истекло, код ответа в данном случае 401.

## 8.1 Сервис получения списка работающих магазинов V2
`GET`
```json
https://api.prostor.ua/api/hs/prostorbot/v2/GetStores
```
### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации
Cookie | `string` | Server ID полученный в ответе авторизации. <br> Пример: SERVERID=cl-api-01\|X62L6\|X62L4

### Server response (Success 200)
Name| Type| Discription
------------ | ------------ | ------------
Success | `boolean` | Результат запроса. <br> “true” - успешный запрос <br> “false” - ошибка
**Response:** | `string()` | Ответ
&nbsp;&nbsp;&nbsp;number | string() | Номер ТТ
&nbsp;&nbsp;&nbsp;shop_closed | boolean | Признак закрытого магазина
&nbsp;&nbsp;&nbsp;name | string() | Наименование ТТ
&nbsp;&nbsp;&nbsp;code | string() | Код ТТ
&nbsp;&nbsp;&nbsp;**city:** | array | Данные населенного пункта
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;city_name | string() | Наименование НП
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;city_koatuu | string() | Код КОАТУУ
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;city_code | string() | Код справочника населенные пункты
&nbsp;&nbsp;&nbsp;address | string() | Адрес ТТ
&nbsp;&nbsp;&nbsp;address_line | string() | Улица и номер дома
&nbsp;&nbsp;&nbsp;longitude | string() | Долгота
&nbsp;&nbsp;&nbsp;latitude | string() | Широта
&nbsp;&nbsp;&nbsp;startTime | string() | Время начала работы ТТ
&nbsp;&nbsp;&nbsp;endTime | string() | Время окончания работы ТТ
&nbsp;&nbsp;&nbsp;delivery_point | boolean | Признак ТТ выдачи товаров
&nbsp;&nbsp;&nbsp;**delivery_dates:** | array | Расчетные даты доставки
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;deadline_time | string() | Час заказа
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;before_deadline | string() | Дата доставки при заказе до часа заказа
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;after_deadline | string() | Дата доставки при заказе после часа заказа
Error | string() | Описание ошибки. <br> **“token_invalid”** – в заголовке передан неправильный ключ или действие ключа истекло, код ответа в данном случае 401.

## <a id="title9"> 9. Сервис получения цены товара </a>
`POST`
```json
https://api.prostor.ua/api/hs/prostorbot/v1/GetPrice
```
### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации
Cookie | `string` | Server ID полученный в ответе авторизации. <br> Пример: SERVERID=cl-api-01\|X62L6\|X62L4

### Request body
Обязательные параметры отмечены <mark>цветом</mark>.
Name| Type| Discription
------------ | ------------ | ------------
<mark>StoreId</mark> | `string()` | Код магазина
<mark>BarCode</mark> | `string()` | Штрихкод товара
IsOnLinePurchase | `boolean` | Признак типа покупки: <br> true - покупка совершенная онлайн <br> false - покупка совершенная в стационарном магазине.

### Server response (Success 200)
Name| Type| Discription
------------ | ------------ | ------------
Success | `boolean` | Результат запроса. <br> “true” - успешный запрос <br> “false” - ошибка
**Response:** | `string()` | Ответ
&nbsp;&nbsp;&nbsp;Code | string() | Код товара в учетной системе
&nbsp;&nbsp;&nbsp;ProductName | string() | Наименование товара
&nbsp;&nbsp;&nbsp;images_link | string() | Ссылка на изображение
&nbsp;&nbsp;&nbsp;link | string() | Ссылка на товар
&nbsp;&nbsp;&nbsp;Price | float() | Цена товара в коп
&nbsp;&nbsp;&nbsp;Barcode | string() | Штрихкод
&nbsp;&nbsp;&nbsp;Rest | float() | Остаток товара в магазине
Error | string() | Описание ошибки. <br> **“token_invalid”** – в заголовке передан неправильный ключ или действие ключа истекло, код ответа в данном случае 401.

**Пример запроса:**
```json
{
	"StoreId": "00131",
    	"BarCode": "5903416007166"
}
```
**Ответ:**
```json
{
    "success": "true",
    "response": {
        "Code": "183265",
        "ProductName": "Eveline Insta Skin Care скраб-паста д/обличчя п/чорн. цяток, 75мл",
        "images_link": null,
        "link": null,
        "Price": 6269,
        "Barcode": "5903416007166"
    },
    "error": ""
}
```
**Пример запроса с использованием CURL:**
```
curl --location --request POST 'https://api.prostor.ua/api/hs/prostorbot/v1/GetPrice' \
--header 'papiauth: 56579efe-7d48-4887-bc73-be2e26b59877' \
--header 'Authorization: Basic QWRtaRvcjp6M=' \
--header 'Content-Type: application/json' \
--data-raw '{
	"StoreId": "00131",
    "BarCode": "5903416007166"
}'
```
---

## <a id="title10"> 10. Сервис получения заказов клиента </a>
`POST`
```json
https://api.prostor.ua/api/hs/prostorbot/v1/GetOrders
```
### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `string` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации
Cookie | `string` | Server ID полученный в ответе авторизации. <br> Пример: SERVERID=cl-api-01\|X62L6\|X62L4

### Request body
Обязательные параметры отмечены <mark>цветом</mark>.
Name| Type| Discription
------------ | ------------ | ------------
<mark>PhoneNumber</mark> | `string()` | Номер телефона клиента в формате 380ХХХХХХХХХ

### Server response (Success 200)
Name| Type| Discription
------------ | ------------ | ------------
Success | `boolean` | Результат запроса. <br> “true” - успешный запрос <br> “false” - ошибка
**Response:** | `string()` | Ответ
&nbsp;&nbsp;&nbsp;stat_created | string() | Дата заказа
&nbsp;&nbsp;&nbsp;order_id | float() | Номер заказа
&nbsp;&nbsp;&nbsp;stat_status | float() | Статус заказа: <br> 1 — новый <br> 2 — в обработке <br> 3 — доставлен <br> 4 — не доставлен <br> 6 — доставляется
&nbsp;&nbsp;&nbsp;total_quantity | float() | Количество товара
&nbsp;&nbsp;&nbsp;total_sum | float() | Итоговая стоимость (с учетом всех скидок, но без учета стоимости доставки)
&nbsp;&nbsp;&nbsp;np_number | string() | Номер ТТН
&nbsp;&nbsp;&nbsp;delivery_type_id | string() | Название варианта доставки
&nbsp;&nbsp;&nbsp;delivery_city | string() | Город доставки
&nbsp;&nbsp;&nbsp;delivery_address | string() | Адрес доставки
&nbsp;&nbsp;&nbsp;payment_type | string() | Тип оплаты
&nbsp;&nbsp;&nbsp;payed | float() | Статус оплаты <br> 0 - не оплачено <br> 1 - оплачено
&nbsp;&nbsp;&nbsp;**products:** | array | Коллекция продуктов участвующих в заказе
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Code | string() | Код товара в учетной системе
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ProductName | string() | Наименование товара
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Parent | string() | Категория товара (может отсутствовать, передается null)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;images_link | string() | Ссылка на изображение
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;link | string() | Ссылка на товар
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Price | float() | Цена товара в коп
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Barcode | string() | Штрихкод
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;quantity | float() | Количество товара
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;total_price | float() | Итоговая стоимость товара с учетом заказанного количества

**Пример запроса:**
```json
{
	"PhoneNumber": "380972145418"
}
```
**Ответ:**
```json
{
    "success": "true",
    "response": [
        {
            "stat_created": "2021-05-13 08:03:34",
            "order_id": 66595,
            "stat_status": 6,
            "total_quantity": 2,
            "total_sum": 513,
            "np_number": null,
            "delivery_type_id": "Новой почтой на отделение",
            "delivery_city": "Мариуполь",
            "delivery_address": "Отделение №13 (до 30 кг на одно место): просп. Строителей, 98",
            "payment_type": "Онлайн-оплата банковской картой",
            "payed": 1,
            "products": [
                {
                    "code": "157780",
                    "Barcode": "8001841663074",
                    "title": "Жидкий стиральный порошок Ariel Горный родник 2,860 л, 2.86 л",
                    "Parent": "Дом/Средства для стирки/Жидкие средства для стирки",
                    "images_link": "https://prostor.ua/content/images/35/zhidkiy-stiralnyy-poroshok-ariel-gornyy-rodnik-2860-l-34915804467783_+71b52134f0.jpg",
                    "link": "https://prostor.ua/product/zhidkiy-stiralnyy-poroshok-ariel-gornyy-rodnik-2860-l/",
                    "price": 314,
                    "quantity": 1,
                    "total_price": 314
                },
                {
                    "code": "149639",
                    "Barcode": "4015400892311",
                    "title": "Капсулы для стирки Tide Всё-в-1 Альпийская свежесть, 30 шт., 750 г",
                    "Parent": "Дом/Средства для стирки/Гелевые капсулы",
                    "images_link": "https://prostor.ua/content/images/21/kapsuly-dlya-stirki-tide-alpiyskaya-svezhest-30-sht-38502672684343_+9d18216f8a.png",
                    "link": "https://prostor.ua/product/kapsuly-dlya-stirki-tide-alpiyskaya-svezhest-30-sht/",
                    "price": 199,
                    "quantity": 1,
                    "total_price": 199
                }
            ]
        }
    ],
    "error": ""
}
```
**Пример запроса с использованием CURL:**
```
curl --location --request POST 'https://api.prostor.ua/api/hs/prostorbot/v1/GetOrders' \
--header 'papiauth: 56579efe-7d48-4887-bc73-be2e26b59877' \
--header 'Authorization: Basic Q3RyYXRvcjpM=' \
--header 'Content-Type: application/json' \
--data-raw '{
	"PhoneNumber": "380972145418"
}'

```
---

## <a id="title11"> 11. Процессинг покупок и возвратов </a>
Данный сервис используется для препроцессинга покупок и возвратов в CRM системе компании Prostor.
`POST`
```json
https://api.prostor.ua/api/hs/PurchaseList/v1/SetPurchaseInfo/
```
