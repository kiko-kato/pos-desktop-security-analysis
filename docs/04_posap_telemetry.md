# Модуль 4. Протокол POS API и живая телеметрия

Кассовое приложение работает с сервером не через публичный Remap API 1.2 (`api.moysklad.ru`), а через закрытый протокол POS API v1.0:
`https://online.moysklad.ru/api/posap/1.0/`

Авторизация идет кастомными заголовками кассы:
- `Lognex-Pos-Auth-Token: 00891...[хвост]`
- `Lognex-Pos-Auth-Cashier-Uid: cashier1@example.com`
- `X-Requested-With: XMLHttpRequest`

## Фактический ответ сервера (GET /settings/retailstore)

При отправке одиночного фонового запроса с токеном, извлеченным из LevelDB, облачный сервер МоегоСклада вернул статус `HTTP 200 OK` со следующими данными тестовой точки:

```json
{
  "meta": {
    "href": "https://online.moysklad.ru/api/posap/1.0/entity/retailstore/83063f7d-c5d5-11f0-0a80-175b000399dd",
    "type": "retailstore",
    "id": "83063f7d-c5d5-11f0-0a80-175b000399dd"
  },
  "name": "Точка 1",
  "cash": 5640000.0,
  "authTokenAttached": true,
  "organization": {
    "name": "ИП Иванов Иван Иванович",
    "inn": "1234",
    "ogrnip": "312345678901234",
    "kktToken": "3539dc26-abda-4afc-980d-8ccaac5f...[хвост]"
  },
  "lastOperationNames": [
    {
      "entity": "retaildemand",
      "name": "00045",
      "moment": "2026-10-17 16:57:00"
    },
    {
      "entity": "retailshift",
      "name": "00012",
      "moment": "2026-10-17 07:50:00"
    }
  ],
  "environment": {
    "chequePrinter": {
      "vendor": "POScenter",
      "name": "РИТЕЙЛ-02Ф",
      "serial": "012345",
      "driver": {
        "version": "5.21.1.1290"
      }
    }
  }
}
```

## Что видно из ответа

- `cash: 5640000.0` - сумма наличности в денежном ящике на момент запроса составляет ровно 56 400.00 рублей (сумма отдается в копейках).
- `lastOperationNames` - номер последнего чека продажи (№00045) и активной смены (№00012).
- `environment` - точная модель фискальника (РИТЕЙЛ-02Ф, драйвер POScenter).
- `authTokenAttached: true` - токен кассы жестко завязан сервером на данную точку продаж.
