```puml
@startuml
skinparam defaultFontSize 10
skinparam shadowing false

!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

title Интернет-банк — Диаграмма контейнеров (C4)

Person(customer, "Клиент интернет-банка")

' External systems
System_Ext(absApi, "Интеграционный API АБС", "Контрактный API для заявок/статусов")
System_Ext(cacheSvc, "Кэш ставок и продуктов", "Источник депозитных продуктов и ставок для каналов")
System_Ext(otpSvc, "OTP/2FA сервис", "Генерация и проверка OTP-кодов")
System_Ext(smsGw, "СМС-шлюз", "Транспорт доставки СМС")

System_Boundary(ibBoundary, "Интернет-банк") {

  Container(webApp, "Веб-приложение интернет-банка", "ASP.NET MVC 4.5 (.NET Framework)", "UI для клиента: просмотр депозитов/ставок и подача заявки")

  Container(api, "IB Backend service", ".NET Framework 4.5",  "Сервисы домена интернет-банка: депозиты, кредиты, заявки, валидации, интеграции")

  ContainerDb(ibDb, "БД интернет-банка", "MS SQL", "Пользователи, локальные данные и черновики заявок/аудит")

  Container(ibOutbox, "Outbox service", ".NET Service",  "Надёжная доставка заявок/событий из IB")

  Container(obs, "Observability service", "Logs/Metrics/Tracing", "Корреляционные ID, централизованные логи и метрики (R5, S2)")
}

' Relationships
Rel(customer, webApp, "Работа в интернет-банке", "HTTPS/TLS")

Rel(webApp, api, "HTTP вызовы", "HTTPS/TLS")
Rel(api, ibDb, "Чтение/запись", "SQL")

' Read flow for rates/products (no ABS direct reads)
Rel(api, cacheSvc, "Чтение депозитных продуктов и ставок (базовые/персональные)", "HTTPS/TLS")

' OTP for confirmation (F8)
Rel(api, otpSvc, "Запрос OTP / проверка OTP", "HTTPS/TLS")
Rel(otpSvc, smsGw, "Отправка СМС с OTP", "API/SMPP (async)")

' Submit application to ABS via API (preferably async with delivery)
Rel(api, ibOutbox, "Запись команды отправки заявки", "Internal")
Rel(ibOutbox, absApi, "Отправка заявки в АБС ", "HTTPS/TLS")

' Observability hookups
Rel(api, obs, "Метрики/логи/трейсы", "Internal")
Rel(ibOutbox, obs, "Метрики/логи/трейсы", "Internal")
@enduml
```