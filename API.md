# <p align="center"> **API PROSTOR ver.1.07-1** </p>

---
1. Авторизация. Получение токена доступа
2. Получение баланса бонусов и контактных данных участника бонусной программы «Prostor Club» по номеру мобильного телефона
3. Добавление участников программы лояльности «Prostor Club»
4. Редактирование данных участников программы лояльности «Prostor Club»
5. Получение покупок клиента за выбранный период
6. Получение последних покупок клиента 
7. Детализация чека
8. Получение списка работающих магазинов
9.  Получение цены товара
10. Получение заказов клиента
11. Процессинг покупок и возвратов
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
## 1. Авторизация. Получение токена доступа
Для работы с сервисом авторизации в GET запросе используется Basic авторизация.
(Получить код авторизации необходимо у ответственного сотрудника компании «Prostor»)

`GET`
```json
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
## 2. Сервис получения баланса бонусов и контактных данных участника бонусной программы «Prostor Club» по номеру мобильного телефона  
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
Тело: 
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
```json
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
## 2.1 Сервис получения расширенных данных баланса бонусов и контактных данных участника бонусной программы «Prostor Club» по номеру телефона
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

## 3. Сервис добавления участников программы лояльности «Prostor Club»
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
```json
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

## 4. Сервис редактирования данных участников программы лояльности «Prostor Club»
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
```json
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
