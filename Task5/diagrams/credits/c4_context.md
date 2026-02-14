```puml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml

skinparam defaultFontSize 10
skinparam shadowing false

title TO-BE: Заявка на кредит онлайн — Диаграмма контекста (C4)

' People
Person(customer, "Клиент", "Пользователь сайта и интернет-банка")
Person(branchAgent, "Сотрудник отделения", "Оформляет кредит в отделении через АБС")
Person(boCredits, "Сотрудник бэк-офиса кредитов", "Обрабатывает заявки в кредитном конвейере")

' External systems
System_Ext(telco, "Телеком-оператор", "Доставка СМС")
System_Ext(bki, "Бюро кредитных историй", "REST API")

' Bank boundary
System_Boundary(bank, "Банк «Стандарт»") {

  System_Boundary(channels, "Channels") {
    System(website, "Сайт банка", "PHP + React.js")
    System(ib, "Интернет-банк", ".NET MVC + MS SQL")
  }

  System(cache, "Кэш кредитных продуктов", "Redis")
  System(loanBpm, "Кредитный конвейер", "Camunda + Oracle")
  System(scoring, "Система кредитного скоринга", "Python + Flask")
  System(kafka, "Kafka (шина событий)", "Внутренняя async интеграция")
  System(abs, "АБС", "Oracle + PL/SQL")
  System(sms, "СМС-шлюз банка", "Интеграция с оператором")
}

Lay_R(bank, bki)
Lay_R(bank, telco)

' Клиентские каналы
Rel(customer, website, "Просмотр кредитов, подача заявки")
Rel(customer, ib, "Просмотр кредитов, подача заявки")
Rel(customer, telco, "Получает СМС")

Rel(website, cache, "Получает условия", "HTTPS/TLS")
Rel(ib, cache, "Получает условия", "HTTPS/TLS")

Rel(website, loanBpm, "Передаёт заявку", "HTTPS/TLS")
Rel(ib, loanBpm, "Передаёт заявку", "HTTPS/TLS")

Rel(ib, sms, "OTP подтверждение", "HTTPS/TLS")
Rel(sms, telco, "Отправка СМС", "SMPP/HTTP")

' Async домен кредитов через Kafka
Rel(loanBpm, kafka, "Публикует события домена")
Rel(kafka, scoring, "События скоринга")
Rel(kafka, cache, "Обновление read-модели")
Rel(kafka, sms, "События уведомлений")

Rel(scoring, bki, "Запрос данных", "REST")

' Отделение
Rel(branchAgent, abs, "Создание/продолжение заявки")
Rel(abs, loanBpm, "Синхронизация заявки", "Internal REST API")

' Регистрация кредита
Rel(boCredits, loanBpm, "Принятие решений")
Rel(loanBpm, abs, "Регистрация договора", "Internal REST API")
@enduml
```