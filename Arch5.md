```mermaid
graph TB
    %% Настройка стилей для зон безопасности
    classDef untrusted fill:#ffcccc,stroke:#cc0000,stroke-width:2px;
    classDef dmz fill:#fff2cc,stroke:#d6b656,stroke-width:2px;
    classDef secure fill:#d9ead3,stroke:#38761d,stroke-width:2px;
    classDef external fill:#e6f2ff,stroke:#3b82f6,stroke-width:2px;

    %% ZONE 0: Браузер пользователя (Вне облака)
    subgraph ZONE_0["[ TRUST ZONE 0: UNTRUSTED / CLIENT SIDE ]"]
        User(("👤 Пользователь <br> (Вбивает текст в чат)"))
        ReactSPA["⚛️ React SPA <br> (Выполняется в браузере)"]
        User --> ReactSPA
    end
    class ZONE_0 untrusted;

    %% ГРАНИЦА ОБЛАКА (Cloud Provider VPC)
    subgraph CLOUD_VPC["☁️ CLOUD PROVIDER VPC"]
        style CLOUD_VPC fill:#f8f9fa,stroke:#7f8c8d,stroke-width:3px,stroke-dasharray: 5 5

        %% ZONE 1: Nginx (Публичная подсеть)
        subgraph ZONE_1["[ TRUST ZONE 1: DMZ ]"]
            Nginx["🌐 Nginx Ingress Proxy <br> (Принимает API запросы)"]
        end
        class ZONE_1 dmz;

        %% ZONE 2: Защищенный контур (Приватная подсеть)
        subgraph ZONE_2["[ TRUST ZONE 2: SECURE CORE ]"]
            GoBackend["🐹 Бэкенд на Go <br> (Проверка JWT и Прав доступа)"]
            GoOrchestrator["🤖 AI Orchestrator <br> (Input/Output Guardrails)"]
            
            VectorDB[("🗄️ Векторная БД <br> (База знаний RAG)")]
            AppDB[("💾 Основная БД <br> (Данные заказов)")]
        end
        class ZONE_2 secure;
    end

    %% ZONE 3: Внешний ИИ
    subgraph ZONE_3["[ TRUST ZONE 3: EXTERNAL API ]"]
        ExternalLLM["🧠 Внешняя модель <br> (LLM API)"]
    end
    class ZONE_3 external;

    %% ==========================================
    %% ПОТОК ХОДА СООБЩЕНИЯ (DATA FLOW)
    %% ==========================================
    
    %% Шаг 1-3: Отправка из React через Nginx в Go
    ReactSPA -- "1. Текст запроса + JWT" --> Nginx
    Nginx -- "2. Проксирование API" --> GoBackend
    
    %% Шаг 4: Бэкенд отдает в оркестратор
    GoBackend -- "3. Валидный сеанс -> Текст" --> GoOrchestrator
    
    %% Шаг 5-6: Поиск контекста в RAG
    GoOrchestrator -- "4. Поиск регламента" --> VectorDB
    VectorDB -. "5. Возврат куска текста" .-> GoOrchestrator
    
    %% Шаг 7: Сборка промпта и отправка в модель
    GoOrchestrator -- "6. Финальный Промпт" --> ExternalLLM
    
    %% Шаг 8: Модель просит вызвать функцию (Tool Calling)
    ExternalLLM -. "7. JSON: Вызови Tool(id=555)" .-> GoOrchestrator
    
    %% Шаг 9-11: Оркестратор просит бэкенд выполнить функцию БЕЗОПАСНО
    GoOrchestrator -- "8. Запрос данных" --> GoBackend
    GoBackend --> AppDB
    AppDB -. "9. Проверка прав + Данные заказа" .-> GoBackend
    GoBackend -. "10. Ответ функции" .-> GoOrchestrator
    
    %% Шаг 12-13: Пересборка ответа через модель
    GoOrchestrator -- "11. Данные функции" --> ExternalLLM
    ExternalLLM -. "12. Финальный текст ответа" .-> GoOrchestrator
    
    %% Шаг 14-16: Возврат пользователю в интерфейс React
    GoOrchestrator -- "13. Чистый текст после Guardrails" --> GoBackend
    GoBackend -- "14. XSS Санитизация вывода" --> Nginx
    Nginx -- "15. Отображение ответа в чате" --> ReactSPA
```
