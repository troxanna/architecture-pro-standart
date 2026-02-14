```puml
@startuml
skinparam defaultFontSize 10
skinparam shadowing false

!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

title Интернет-банк — Диаграмма контейнеров (C4) — Outbox-сервис только для депозитов

Person(customer, "Клиент интернет-банка")

' External systems
System_Ext(absApi, "Интеграционный API АБС", "Контрактный API АБС (депозиты)")
System_Ext(loanBpmApi, "Кредитный конвейер (API)", "Master кредитных заявок/статусов/предодобренных предложений")
System_Ext(depositCache, "Кэш ставок и продуктов", "Источник депозитных продуктов и ставок для каналов")
System_Ext(creditCache, "Кэш кредитных продуктов", "Read-model условий кредитов для каналов")
System_Ext(otpSvc, "OTP/2FA сервис", "Генерация и проверка OTP-кодов")

System_Boundary(ibBoundary, "Интернет-банк") {

  Container(webApp, "Веб-приложение интернет-банка", "ASP.NET MVC 4.5 (.NET Framework)", "UI для клиента: кредиты/депозиты, предодобренные предложения, подача заявок")

  Container(api, "IB Backend (монолит)", ".NET Framework 4.5", "Доменная логика интернет-банка: депозиты, кредиты, заявки, валидации, интеграции")

  ContainerDb(ibDb, "БД интернет-банка", "MS SQL","Пользователи, локальные данные, черновики заявок, аудит")

  Container(depOutboxSvc, "Deposit Outbox Service", ".NET Service + MS SQL", "Надёжная доставка депозитных заявок в АБС. Используется только депозитным доменом")

  Container(obs, "Observability service", "Logs/Metrics/Tracing", "Корреляционные ID, централизованные логи и метрики (R5, S2)")
}

' Relationships
Rel(customer, webApp, "Работа в интернет-банке", "HTTPS/TLS")

Rel(webApp, api, "HTTP вызовы", "HTTPS/TLS")
Rel(api, ibDb, "Чтение/запись", "SQL")

' Read models for UI
Rel(api, depositCache, "Чтение депозитных продуктов и ставок", "HTTPS/TLS")
Rel(api, creditCache, "Чтение кредитных продуктов и условий", "HTTPS/TLS")

' OTP for confirmation (кредиты/депозиты)
Rel(api, otpSvc, "Запрос OTP / проверка OTP", "HTTPS/TLS")

' Deposits: async reliable delivery via dedicated outbox service (ONLY deposits)
Rel(api, depOutboxSvc, "Передаёт депозитную команду «Подать заявку»", "Internal HTTP")
Rel(depOutboxSvc, absApi, "Отправка депозитной заявки", "HTTPS/TLS")

' Credits: direct integration (no outbox microservice)
Rel(api, loanBpmApi, "Передаёт кредитную заявку / запрашивает статусы и предодобренные", "HTTPS/TLS")

' Observability hookups
Rel(api, obs, "Метрики/логи/трейсы", "Internal")
Rel(depOutboxSvc, obs, "Метрики/логи/трейсы", "Internal")

@enduml
```