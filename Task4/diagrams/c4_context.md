```puml
@startuml
' C4-PlantUML (Context diagram)
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml

skinparam defaultFontSize 10
skinparam shadowing false

title MVP: Депозитные ставки для внутреннего и партнёрского кол-центра — Диаграмма контекста (C4)

' People
Person(customer, "Клиент", "Звонит в кол-центр, уточняет условия депозитов")
Person(ccAgent, "Сотрудник кол-центра", "Консультирует клиента по депозитным ставкам и условиям")
Person_Ext(partnerAgent, "Сотрудник партнёрского кол-центра", "Консультирует клиента по ставкам (по файлу)")

' External systems
System_Ext(abs, "АБС", "Oracle + PL/SQL", "Master: депозитные продукты и ставки")
System_Ext(partnerCC, "Партнёрский кол-центр", "Внешняя система партнёра (только файловый обмен)")

' Bank systems (scope of change)
System_Boundary(bank, "Банк «Стандарт» (в рамках данной доработки)") {

  System(cache, "Кэш ставок и продуктов", "Redis + API", "Read-model ставок/продуктов для всех каналов + формирование файла для партнёра")

  System(crm, "CRM / Система кол-центра", "React + Java + PostgreSQL", "Рабочее место оператора: просмотр ставок и консультации")

  System(fileGateway, "Файловый шлюз банка", "Secure file zone / DMZ", "Защищённая зона передачи файлов внешним партнёрам")
}

' Relationships
Rel(customer, ccAgent, "Звонит, уточняет условия")
Rel(ccAgent, crm, "Работает в CRM")

Rel(crm, cache, "Получает актуальные ставки (только чтение)", "HTTPS/TLS")

Rel(abs, cache, "Передаёт депозитные продукты и ставки", "Kafka")

Rel(cache, fileGateway, "Публикует файл ставок (CSV/JSON)", "Internal")
Rel(fileGateway, partnerCC, "Передача файла ставок", "File transfer")

Rel(partnerAgent, partnerCC, "Использует полученные ставки")
Rel(customer, partnerAgent, "Звонит, уточняет условия")

@enduml
```