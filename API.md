# <center> **API PROSTOR ver.1.07-1** </center>
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
Authorization | `String` | Authorization header <br> Допустимые значения: “Basic value”. <br> Например: Basic YWhhbWlsdG9uQGFwaWdlZS5jb206bXlwYXNzdzByZAo 
papiauth | `string(36)` | **Для клиента мобильного приложения**. <br> Ключ, который использовался при ОТП авторизации конкретного пользователя в заголовке papiauth. Предназначен для идентификации клиента.


### Response (Success 200)
Name| Type| Discription
------------ | ------------ | ------------
Success | `boolean` | Результат запроса. <br> “true” - успешный запрос <br> “false” - ошибка
Response/KEY | `String(36)` | Ключ доступа к сервису получения баланса (ограничен по времени)
Error | `String` | Описание ошибки

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
## 2.1 Верификация номера телефона клиента
В ответе вы получите `“token_verification”`, который необходимо передавать в теле запроса сервисов требующих верификацию. 
Клиенту должен поступить звонок или смс. Клиент должен сообщить последние 4 цифры номера, с которого поступил звонок или полученный код в смс.

`GET`
```json
https://api.prostor.ua/api/hs/AuthService/v1/ValidateContact/{phone_number}/{verification method}
```
### Request headers
Name| Type| Discription
------------ | ------------ | ------------
Authorization | `String` | Authorization header <br> Допустимые значения: “Basic value” 
papiauth | `string(36)` | Ключ (KEY) полученный в ответе сервиса авторизации
Cookie | `string` | Server ID полученный в ответе авторизации. <br> Пример: SERVERID=cl-api-01\|X62L6\|X62L4

### Request parameters
Name| Type| Discription
------------ | ------------ | ------------
phone_number | `String` | Номер телефона клиента в формате 38ХХХХХХХХХХ 
verification_method | `string` | Метод верификации. Допустимые значения: <br> *callback* - обратный звонок <br> *sms* -  OTP sms

### Response (Success 200)
Name| Type| Discription
------------ | ------------ | ------------
Success | `boolean` | Результат запроса. <br> “true” - успешный запрос <br> “false” - ошибка
Response | `String()` | Описание ответа. Если проверка пройдена “Passed” или “data not verified”, если не пройдена
Error | `String()` | Описание ошибки: <br> **“Parse Error”** – параметр передан в неверном формате <br> **“Required Parameter Not Specified”** – не передан обязательный параметр <br> **“Token_invalid”** – в заголовке передан неправильный ключ или действие ключа истекло, код ответа в данном случае 401

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
