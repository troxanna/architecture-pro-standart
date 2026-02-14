```puml
@startuml
skinparam defaultFontSize 10
skinparam shadowing false

!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

title АБС — Диаграмма контейнеров (C4)

' People
Person(boDeposits, "Бэк-офис депозитов")
Person(boCredits, "Бэк-офис кредитов")

' External systems
System_Ext(ib, "Интернет-банк", "Канал для действующих клиентов")
System_Ext(crm, "CRM / Кол-центр", "Канал для заявок с сайта")
System_Ext(cacheSvc, "Кэш ставок и продуктов", "Чтение ставок/продуктов каналами")
System_Ext(smsGw, "СМС-шлюз", "OTP и бизнес-уведомления")
System_Ext(kafka, "Kafka", "Брокер сообщений", "Асинхронная доставка событий")

System_Boundary(absBoundary, "АБС") {

  Container(absClient, "Десктоп-клиент АБС", "Delphi Desktop","UI для сотрудников (депозиты/кредиты)")

  ContainerDb(absOracle, "База данных АБС", "Oracle","Данные + бизнес-логика (PL/SQL)")

  Container(creditModule, "Модуль кредитов", "PL/SQL (Oracle)","Риск-параметры банка/клиента, используемые для расчёта депозитных ставок")

  Container(depositModule, "Модуль депозитов", "PL/SQL (Oracle)","Заявки на депозит, расчёт базовых/персональных ставок, статусы, подтверждение условий")

  Container(intApi, "Интеграционный API АБС", "REST/HTTP","Точка входа для каналов: приём заявок, выдача статусов/деталей")
}

' Internal relationships
Rel(absClient, absOracle, "Работа пользователей", "DB connection")
Rel(creditModule, absOracle, "Хранение данных/процедуры", "PL/SQL in DB")
Rel(depositModule, absOracle, "Хранение данных/процедуры", "PL/SQL in DB")

Rel(absClient, creditModule, "Операции кредитного бэк-офиса", "Calls PL/SQL")
Rel(absClient, depositModule, "Операции депозитного бэк-офиса", "Calls PL/SQL")

Rel(intApi, depositModule, "Команды/запросы по депозитным заявкам", "Internal calls")
Rel(creditModule, depositModule, "Риск-параметры для расчёта ставок", "Internal calls")

' Publish events to Kafka (external platform)
Rel(depositModule, kafka, "Публикация событий", "Kafka protocol")

' External consumers
Rel(kafka, cacheSvc, "События об изменении ставок и продуктов", "Kafka protocol")
Rel(kafka, smsGw, "События об изменении статуса заявки", "Kafka protocol")

' External interactions
Rel(ib, intApi, "Отправка заявки на депозит", "HTTPS/TLS")
Rel(crm, intApi, "Регистрация заявки на депозит", "HTTPS/TLS")

Rel(boDeposits, absClient, "Работа с заявками/ставками", "")
Rel(boCredits, absClient, "Ввод риск-параметров", "")
@enduml

```