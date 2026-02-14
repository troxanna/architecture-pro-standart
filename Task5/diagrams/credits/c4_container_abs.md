```puml
@startuml
skinparam defaultFontSize 10
skinparam shadowing false

!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

title АБС — Диаграмма контейнеров (C4) — функциональность кредитов (объединённый модуль)

' People
Person(boDeposits, "Бэк-офис депозитов")
Person(boCredits, "Бэк-офис кредитов")
Person(branchAgent, "Сотрудник отделения", "Оформляет кредит/продолжает заявку в АБС")

' External systems
System_Ext(ib, "Интернет-банк", "Канал для действующих клиентов")
System_Ext(crm, "CRM / Кол-центр", "Канал для заявок с сайта (депозиты)")
System_Ext(cacheSvc, "Кэш ставок и продуктов", "Чтение ставок/продуктов каналами")
System_Ext(smsGw, "СМС-шлюз", "OTP и бизнес-уведомления")
System_Ext(loanBpmApi, "Кредитный конвейер (API)", "Camunda + Oracle")
System_Ext(kafka, "Kafka (платформенная шина событий)", "Брокер сообщений банка")

System_Boundary(absBoundary, "АБС") {

  Container(absClient, "Десктоп-клиент АБС", "Delphi Desktop", "UI для сотрудников (депозиты и кредиты)")

  ContainerDb(absOracle, "База данных АБС", "Oracle","Данные + бизнес-логика (PL/SQL)")

  Container(depositModule, "Модуль депозитов", "PL/SQL (Oracle)","Заявки на депозит, расчёт ставок, статусы")

  Container(creditModule, "Модуль кредитов", "PL/SQL (Oracle)","Оформление кредита в отделении (создание/поиск по ID), регистрация кредита/договора, риск-параметры")

  Container(intApi, "Интеграционный API АБС", "REST/HTTP", "Точка входа интеграций (депозиты/кредиты)")
}

' Internal relationships
Rel(absClient, absOracle, "Работа пользователей", "DB connection")

Rel(depositModule, absOracle, "Хранение данных/процедуры", "PL/SQL")
Rel(creditModule, absOracle, "Хранение данных/процедуры", "PL/SQL")

Rel(absClient, depositModule, "Операции по депозитам", "")
Rel(absClient, creditModule, "Операции по кредитам (отделение/кредиты)", "")

Rel(intApi, depositModule, "Команды/запросы по депозитам", "Internal calls")
Rel(intApi, creditModule, "Команды/запросы по кредитам", "Internal calls")

' Deposits integrations
Rel(ib, intApi, "Заявки на депозит / статусы", "HTTPS/TLS")
Rel(crm, intApi, "Регистрация депозитной заявки", "HTTPS/TLS")

Rel(depositModule, kafka, "События ставок/продуктов/статусов", "Kafka")
Rel(kafka, cacheSvc, "Обновление кэша", "Kafka")
Rel(kafka, smsGw, "Уведомления по депозитам", "Kafka")

' Loans integrations
Rel(branchAgent, absClient, "Создание новой заявки или продолжение по ID", "")

Rel(intApi, loanBpmApi, "Запрос данных по заявке / синхронизация по ID", "Internal REST API")
Rel(loanBpmApi, intApi, "Передача параметров одобренного кредита", "Internal REST API")ф

Rel(creditModule, kafka, "События оформления кредита (по необходимости)", "Kafka")
Rel(kafka, smsGw, "Уведомления по кредитам", "Kafka")


' People usage
Rel(boDeposits, absClient, "Работа с депозитами", "")
Rel(boCredits, absClient, "Работа с кредитами/настройками", "")
@enduml
```