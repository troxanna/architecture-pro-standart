```puml
@startuml
' C4-PlantUML (Context diagram)
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml

skinparam defaultFontSize 10
skinparam shadowing false

title TO-BE: Открытие депозита онлайн — Диаграмма контекста (C4)

' People
Person(customer, "Клиент", "Пользователь сайта и интернет-банка")
Person(ccAgent, "Сотрудник кол-центра", "Обрабатывает заявки новых клиентов из CRM и консультирует по депозитам")
Person(boDeposits, "Сотрудник бэк-офиса депозитов", "Подтверждает условия/ставку и обрабатывает заявки в АБС")
Person(boCredits, "Сотрудник бэк-офиса кредитов", "Вносит параметры кредитного риска банка для расчёта ставок")

' External systems
System_Ext(telco, "Телеком-оператор", "Внешний оператор доставки СМС")

' Bank systems
System_Boundary(bank, "Банк «Стандарт»") {

  System(website, "Сайт банка", "Просмотр депозитов и подача заявки")
  System(ib, "Интернет-банк", ".NET MVC монолит (ASP.NET MVC 4.5), MS SQL", "Просмотр депозитов и подача заявки на депозит")
  System(cache, "Система кэша ставок и продуктов", "Java Spring Boot + Redis", "Источник данных для сайта/ИБ по ставкам и продуктам")
  System(abs, "АБС", "Oracle + PL/SQL, клиент Delphi", "Master: заявки и ставки по депозитам, статусы, бэк-офис обработка")
  System(crm, "CRM / Система кол-центра", "React + Java Spring Boot + PostgreSQL", "Приём заявок с сайта, работа менеджеров, передача заявок в АБС")
  System(sms, "СМС-шлюз банка", "Интеграция с оператором", "OTP/уведомления клиентам")
  System(kafka, "Kafka (Event Bus)", "Банковская платформа событий", "Асинхронная доставка событий между системами")
}

' Relationships
Rel(customer, website, "Просматривает депозиты и ставки, подаёт заявку")
Rel(website, cache, "Получает список депозитов и актуальные ставки")
Rel(website, crm, "Передаёт заявку")

Rel(ccAgent, crm, "Просматривает заявки, фиксирует результат звонка")
Rel(crm, abs, "Регистрирует/передаёт заявку на депозит")

Rel(customer, ib, "Просматривает депозиты/ставки, подаёт заявку на депозит")
Rel(ib, cache, "Получает список депозитов и ставки")

Rel(ib, sms, "Инициирует OTP для подтверждения действия")
Rel(sms, telco, "Отправка СМС (OTP/уведомления)")
Rel(customer, telco, "Получает СМС-код/уведомления")

Rel(ib, abs, "Отправляет заявку на депозит")
Rel(boDeposits, abs, "Обрабатывает заявку, подтверждает условия/ставку, меняет статус")
Rel(boCredits, abs, "Вносит уровень кредитного риска банка")

' Event-driven integration
Rel(abs, kafka, "Публикует события (ставки/продукты/статусы)", "Kafka protocol")
Rel(kafka, cache, "События обновления read-model", "Kafka protocol")
Rel(kafka, sms, "События для отправки уведомлений", "Kafka protocol")

@enduml

```