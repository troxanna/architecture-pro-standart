```puml
@startuml
skinparam defaultFontSize 10
skinparam shadowing false

!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

title Кэш продуктов и ставок — Диаграмма контейнеров (C4)

' External systems
System_Ext(abs, "АБС", "Oracle + PL/SQL", "Master: депозитные продукты и ставки")
System_Ext(kafka, "Kafka", "Event Broker", "Передача событий об изменении ставок")
System_Ext(website, "Сайт банка", "Канал просмотра депозитов")
System_Ext(ib, "Интернет-банк", "Канал просмотра депозитов")
System_Ext(crm, "CRM / Система кол-центра", "Рабочее место оператора")
System_Ext(fileGateway, "Файловый шлюз банка", "DMZ / Secure File Exchange")
System_Ext(partnerCC, "Партнёрский кол-центр", "Внешняя система (получает файл)")

System_Boundary(cacheBoundary, "Система кэша продуктов и ставок") {

  Container(api, "Rates & Products API", "Java Spring Boot", "Контрактный API для чтения ставок/продуктов (read-model)")

  Container(kafkaConsumer, "Kafka Consumer", "Spring Kafka", "Обрабатывает события об изменении ставок из Kafka")

  Container(exportJob, "File Export Service", "Spring Scheduler", "Формирование файла ставок для партнёра")

  ContainerDb(redis, "Redis", "Redis", "Read-model ставок и продуктов")

  ContainerDb(metaDb, "Metadata DB", "MS SQL / Oracle", "Метаданные: версии, журнал экспортов")

  Container(obs, "Observability", "Logs/Metrics/Tracing", "Метрики и централизованные логи")
}

' ===== Relationships =====

' Channels read only
Rel(website, api, "Чтение ставок", "HTTPS/TLS")
Rel(ib, api, "Чтение ставок", "HTTPS/TLS")
Rel(crm, api, "Чтение ставок", "HTTPS/TLS")

Rel(api, redis, "Читает read-model", "Redis protocol")

' Events from ABS
Rel(abs, kafka, "Публикует события об изменении ставок", "Kafka protocol")
Rel(kafka, kafkaConsumer, "Доставка событий", "Kafka protocol")

Rel(kafkaConsumer, redis, "Обновляет read-model", "Redis protocol")
Rel(kafkaConsumer, metaDb, "Фиксирует версию/время обновления", "SQL")
Rel(kafkaConsumer, obs, "Метрики/логи", "Internal")

' Export
Rel(exportJob, redis, "Читает данные для экспорта", "Redis protocol")
Rel(exportJob, metaDb, "Фиксирует версию файла и статус", "SQL")
Rel(exportJob, fileGateway, "Публикует файл ставок (CSV/JSON)", "Internal file publish")
Rel(exportJob, obs, "Метрики/логи экспорта", "Internal")

Rel(fileGateway, partnerCC, "Передача файла", "File transfer (pull)")

@enduml
```