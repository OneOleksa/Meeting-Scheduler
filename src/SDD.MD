SDD: Система обліку зустрічей (MeetFlow)Огляд і глосарій
Система «MeetFlow» є класичним Java Web-застосунком (Servlet + JSP), який працює на сервері Apache Tomcat і використовує PostgreSQL для зберігання даних. Організатор створює вільні слоти через вебперегляд, а учасник бронює їх на JSP-сторінках.Поза скоупом першої версії: складні фреймворки (Spring/Hibernate), REST API, мобільні додатки.Глосарій:ТермінЗначення в цьому документіServletJava-клас, який обробляє HTTP-запити від клієнта та керує бізнес-логікою.JSPШаблон сторінки (JavaServer Pages) для рендерингу HTML-інтерфейсу в браузері.DAO (Data Access Object)Шар взаємодії з базою даних PostgreSQL через JDBC.Функціональні вимоги
| ID | User story | Пріоритет |
| :--- | :--- | :--- |
| US-1 | Як організатор, я хочу додати вільний слот через вебформу, щоб інші могли його бачити. | Must |
| US-2 | Як учасник, я хочу переглянути список вільних слотів у вигляді таблиці на JSP-сторінці. | Must |
| US-3 | Як учасник, я хочу натиснути кнопку «Забронювати» на сторінці, щоб закріпити слот за собою. | Must |
| US-4 | Як користувач, я хочу бачити список своїх зустрічей у своєму профілі. | Must |
| US-5 | Як організатор, я хочу скасувати зустріч через інтерфейс, звільнивши слот. | Should |Нефункціональні вимоги
| ID | Категорія | Вимога (з метрикою) |
| :--- | :--- | :--- |
| NFR-1 | Продуктивність | Завантаження JSP-сторінок та обробка запитів сервлетами виконуються за $< 200$ мс. |
| NFR-2 | Сумісність | Застосунок розгортається на Apache Tomcat 10+ і працює з PostgreSQL 15+. |
| NFR-3 | Безпека даних | Усі параметри з вебформ обробляються з ухиленням від SQL-ін'єкцій (використання PreparedStatement у JDBC). |АрхітектураФрагмент кодаflowchart TD
Browser[Веббраузер користувача] -->|HTTP запит| Tomcat[Apache Tomcat Server]
Tomcat --> Servlet[Java Servlets / Controller]
Servlet --> Service[Business Logic Services]
Service --> DAO[JDBC DAO Layer]
DAO --> DB[(PostgreSQL Database)]
Servlet --> JSP[JSP Сторінки / View]
JSP --> Browser
Компоненти:JSP Сторінки (View) — інтерфейс користувача для відображення розкладу та форм.Servlets (Controller) — приймають HTTP-запити, викликають сервіси та перенаправляють на JSP.Services & DAO — бізнес-логіка та виконання SQL-запитів до PostgreSQL із використанням моделей, згенерованих через Lombok.Модель данихФрагмент кодаerDiagram
USERS ||--o{ SLOTS : creates
USERS ||--o{ MEETINGS : books
SLOTS ||--o| MEETINGS : "turns into"

    USERS {
        int id PK
        string name
        string role
    }

    SLOTS {
        int id PK
        int organizer_id FK
        timestamp start_time
        timestamp end_time
        boolean is_booked
    }

    MEETINGS {
        int id PK
        int slot_id FK
        int attendee_id FK
        string status
    }
Ключові сценарії6.1. Бронювання слота через вебсторінкуФрагмент кодаsequenceDiagram
autonumber
participant User as Користувач (Браузер)
participant Servlet as BookingServlet
participant Service as MeetingService
participant DAO as SlotDAO
participant DB as PostgreSQL

    User->>Servlet: Клік "Забронювати" (POST /book?slotId=5)
    Servlet->>Service: bookSlot(5, userId)
    Service->>DAO: Оновлення статусу слота в БД
    DAO->>DB: UPDATE slots SET is_booked = true...
    DB-->>DAO: Успішно
    DAO-->>Service: Підтверджено
    Service-->>Servlet: Успіх
    Servlet-->>User: Редирект на сторінку успішного бронювання (JSP)
Ескіз публічних інтерфейсів (URL-ендпоінти)
| HTTP Метод | URL Шлях | Призначення |
| :--- | :--- | :--- |
| GET | /slots | Перегляд списку вільних слотів (US-2) |
| POST | /slots/add | Створення нового слота організатором (US-1) |
| POST | /meetings/book | Бронювання обраного слота (US-3) |
| GET | /profile | Перегляд зустрічей поточного користувача (US-4) |Ризики та альтернативиРизик: Проблеми з підключенням до бази даних через відсутність пулу з'єднань. Пом'якшення: Використання DataSource або Apache DBCP для управління з'єднаннями з PostgreSQL.Альтернатива: Використання повноцінного фреймворку Spring MVC. Відкинуто задля збереження класичного стеку без Spring відповідно до вимог.