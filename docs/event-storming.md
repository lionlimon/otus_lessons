# Event Storming — карта событий

## Легенда цветов (классическая схема Бриджа/Брандолини)

| Цвет | Тип | Вопрос |
|---|---|---|
| 🟧 Оранжевый | Событие (Domain Event) | Что уже произошло |
| 🟦 Синий | Команда | Что кто-то попросил сделать |
| 🟨 Жёлтый, крупный/жирная рамка | Агрегат | Кто обрабатывает команду и хранит состояние |
| 🟡 Жёлтый, светлее/тонкая рамка | Актор | Кто инициировал команду — стрелка ведёт к событию/агрегату |
| 🟪 Фиолетовый | Политика | «Когда X, то автоматически Y» |
| 🟩 Зелёный | Read model | Что видит актор перед решением |
| 🟥 Розовый | Внешняя система | Кто снаружи участвует |

```mermaid
flowchart LR
    Client(["Клиент\n(внешний, не пользователь\nсистемы)"])
    Agent(["Агент поддержки"])
    Supervisor(["Супервизор"])
    Admin(["Администратор"])

    MsgForm["Отправлена форма\nобратной связи"]
    MsgChat["Первое сообщение\nклиента в чате"]
    ManualChannel["Обращение зафиксировано\nвручную агентом\n(нестандартный канал)"]
    P1{{"Политика:\nклиент обратился впервые\nпо новому поводу →\nсоздать обращение"}}

    AggTicket["Агрегат «Обращение»\n(Ticket)"]

    E1["Создано обращение"]

    subgraph LC["Жизненный цикл обращения (после создания,\nлюбой порядок, может повторяться)"]
        ES["Статус обращения изменён"]
        EEsc["Обращение эскалировано\nна другую линию поддержки"]
        EReassign["Обращение переназначено\nдругому агенту"]
        EPrio["Приоритет обращения изменён"]
        EComment["Добавлен комментарий\nк обращению"]
        MsgIn["Получено сообщение\nот клиента"]
        E3out["Отправлено сообщение\nклиенту"]
    end

    P3{{"Политика:\nобстоятельства (напр. приближение\nSLA) → пересчитать приоритет"}}

    E2["Обращение достигло\nкритического SLA"]
    P2{{"Политика:\nдедлайн приближается →\nэмитить событие"}}

    E4a["Обращение закрыто агентом"]
    E4b["Обращение закрыто\nавтоматически (по таймауту)"]

    PAudit{{"Политика:\nлюбое событие Тикетницы /\nАдминистрирования → писать в Аудит"}}
    AggAudit["Агрегат «Журнал аудита»\n(append-only)"]
    EView["Зафиксирован просмотр\nПДн по обращению"]

    subgraph ADMIN["Администрирование (отдельный трек —\nсвоя причина изменения)"]
        AUserCreated["Пользователь создан"]
        ARole["Роль пользователя изменена"]
        ADeactivate["Пользователь деактивирован"]
        APolicyConfig["Настройки маршрутизации/SLA\nизменены"]
    end

    AggUser["Агрегат «Пользователь»\n(User & Role)"]
    AggRouting["Агрегат «Политики\nмаршрутизации/SLA»"]

    Client --> MsgForm
    Client --> MsgChat
    Client --> MsgIn
    Agent --> ManualChannel
    Agent --> ES
    Agent --> EComment
    Agent --> E3out
    Agent --> E4a
    Agent --> EEsc
    Agent --> EReassign
    Agent --> EPrio
    Agent --> EView
    Supervisor --> EEsc
    Supervisor --> EReassign
    Supervisor --> EPrio
    Supervisor --> EView
    Admin --> AggUser
    Admin --> AggRouting
    Admin --> EView

    MsgForm --> P1
    MsgChat --> P1
    ManualChannel --> P1
    P1 --> AggTicket
    AggTicket ==> E1
    AggTicket ==> LC
    E1 --> P2 --> E2
    E2 -.-> P3 -.-> EPrio
    E1 --> LC
    LC --> E4a
    LC --> E4b

    AggUser ==> AUserCreated
    AggUser ==> ARole
    AggUser ==> ADeactivate
    AggRouting ==> APolicyConfig

    LC -.-> PAudit
    E4a -.-> PAudit
    E4b -.-> PAudit
    EView -.-> PAudit
    ADMIN -.-> PAudit
    PAudit -.-> AggAudit

    classDef event fill:#F2A65A,stroke:#C97C1E,color:#1a1a1a
    classDef policy fill:#C9A0DC,stroke:#7B4397,color:#1a1a1a
    classDef actor fill:#FFEB3B,stroke:#C9A227,color:#1a1a1a
    classDef aggregate fill:#FFC107,stroke:#8a6d00,color:#1a1a1a,stroke-width:3px

    class MsgForm,MsgChat,MsgIn,ManualChannel,E1,E2,E3out,E4a,E4b,ES,EEsc,EReassign,EPrio,EComment,EView,AUserCreated,ARole,ADeactivate,APolicyConfig event
    class P1,P2,P3,PAudit policy
    class Client,Agent,Supervisor,Admin actor
    class AggTicket,AggUser,AggRouting,AggAudit aggregate
```
