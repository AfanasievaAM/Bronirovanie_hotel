# Bronirovanie_hotel
Система резервирования даты в эко-отеле

**Общее описание**

1.	API – сервис для системы бронирования даты: API – сервис предоставляет набор API для бронирования пользователями нужного времени для отдыха. Обрабатывает запросы HTTP, взаимодействует с БД для управления наличием дат бронирования и резервированием. 
2.	Сервис управления дат бронирования: данный сервис отвечает за управление дат бронирования. Осуществляет обработку запросов от API – сервиса для осуществления бронирования и освобождения дат заездов, обновляет расписание в БД
3.	Сервис для сбора и обработки данных: данный сервис отвечает за анализ и обработку информации о бронированиях дат заездов в отель. Выполняется на данном этапе генерация отчётов, подсчет статистики.
4.	Веб-интерфейс для пользователей: Веб-интерфейс предоставляется пользователям, а в частотности посетителям, для осуществления бронирования даты заезда в эко-отель. Осуществляет возможность просмотра доступных слотов времени (дат), выбирать удобные даты, а также отменять (или удалять), отменять время бронирования.

**Техническое описание системы:**

Система состоит из трех сервисов:
- API – сервис 
- Сервис для сбора и обработки данных
- Веб-интерфейс

Каждый сервис работает строго в своем контейнере, который запускается в Kubernetes.

*Разработка основных сервисов:*

1. API – сервис для системы резервирования дат в отеле:
Используемые компоненты:
- Deployment
- Service
- Secret
- ConfigMap

Образ: api-res-service:v1

Назначение сервиса:
предоставляет API для взаимодействия с системой резервирования брони в отеле, взаимодействует с БД, обрабатывает запросы, взаимодействует с сервисом управления дат бронирования.
Deployment: Настроен с requests и limits. Определяет шаблон подов, управляет запуском подов в кластере.
Service: Обеспечивает доступ внутри к API – сервису. Направляет запросы входящие к API – сервису.
Secret: Хранит конфиденциальные данные, ключи шифрования, пароли.
ConfigMap: Здесь хранятся конфигурационные данные для API – сервиса.


*2. Сервис управления дат бронирования:*

Используемые компоненты:
- Deployment
- ConfigMap
- Job

Образ: uprav-reserv-service:v1

Назначение сервиса:
Отвечает за управление резервированиями номеров в эко-отели. На данном этапе осуществляется резервирование или освобождение ячеек дат.

Deployment: Определение шаблонов подов для сервиса управления дат бронирования
ConfigMap: Здесь хранятся конфигурационные данные для сервиса управления дат бронирования.
Job: Обеспечивает запуск и контроль задач.

*3. Сервис для сбора и обработки данных:*

Используемые компоненты:
- Deployment
- ConfigMap
- Job

Образ: data-sbor-service:v1

Назначение сервиса:
Обрабатывает данные и выполняет операции по вычислению, запускает задачи для обработки данных.

Deployment: Определение шаблонов подов для сервиса сбора и обработки данных
ConfigMap: Здесь хранятся конфигурационные данные для сервиса сбора и обработки данных
Job: На данном этапе определение задач для сервиса сбора и обработки данных.

*4. Веб-интерфейс для пользователей:*

Используемые компоненты:
- Deployment
- Service
- Ingress

Образ: res-web-service:v1

Назначение:
Данный сервис создан для пользователей, у которых есть возможность просмматривать доступные даты бронирований, могут отменять или переносить на другие даты.

Deployment: Определение шаблонов подов для Веб-интерфейса
Service: С помощью Kubernetes обеспечивается доступ к Веб-интерфейсу
Ingress: Создан для направления внешних входящих запросов к Веб-интерфейсу.


**Зависимости вне К8s**

- API – сервис взаимодествует с REST сервисами
- API – сервис обращается к БД

Сервис авторизации пользователей:
Порт: 8000 
Сервер: auten-service.example.com
Требования: 1 CPU, 2 GB RAAM, 20GB

Сервис управления дат бронирования:
Порт: 8080
Сервер: uprav-service.example.com
Требования: 1 CPU, 2 GB RAAM, 20GB

Очередь сообщений:
здесь API – сервис использует очередь сообщений для беспрерывной работы сервиса по обработке входящих сообщений; запросы, требующие много врмени, отправляются в фоновый режим обработки данных, что не перегружает сервис и не снижает работоспособности; нагрузка системы будет оптимизирована; при завершении одной обработки запроса с ошибкой, не блокируются запросы оставшиеся на обработку. 

Порт: 5874
Сервер: message.example.com
Требования: Требования: 1 CPU, 4 GB RAAM, 20GB


**Дополнительные конфигурации:**

*Кэширование данных :*

Используемые компоненты:
- Deployment
- ConfigMap
- Job

Образ: kech-service:v1

Назначение сервиса:
Необходимо для производительности и масштабируемости системы.

Deployment: Определение шаблонов подов для сервиса кэширования данных
ConfigMap: Здесь хранятся конфигурационные данные для сервиса кэширования данных
Job: На данном этапе определение задач для сервиса кэширования данных


*Внесение предоплаты:*

Используемые компоненты:
- Deployment
- ConfigMap
- Service

Образ: oplata-service:v1

Назначение сервиса:
Сбор денежных средств и транспортировка на счёт фирмы, осуществляющей данную деятельность.

Deployment: Определение шаблонов подов для сервиса внесения предоплаты
ConfigMap: Здесь хранятся конфигурационные данные для внесения предоплаты
Service: Обеспечивает доступ внутри к API – сервису.


*Интеграция с внешними сервисами:*

Используемые компоненты:
- Deployment
- ConfigMap
- Job

Образ: vnech-service:v1

Назначение сервиса:
Коммуникация с другими внешними сервисами: с картами, навигацией

Deployment: Определение шаблонов подов для сервиса интеграции с внешними сервисами
ConfigMap: Здесь хранятся конфигурационные данные интеграции с внешними сервисами
Job: На данном этапе определение задач для сервиса интеграции с внешними сервисами

