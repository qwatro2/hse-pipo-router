# Мобильное приложение для управления совместными тратами

## Описание

Приложение представляет из себя текстовый мессенджер с функционалом добавления чеков с тратами, распределения позиций между участниками и расчетом взаимных долгов.
В рамках НИСа хочу показать три микросервиса своего проекта:
- [shared-expenses-auth](https://github.com/qwatro2/shared-expenses-auth)
- [shared-expenses-gateway](https://github.com/qwatro2/shared-expenses-gateway)
- [shared-expenses-groups](https://github.com/qwatro2/shared-expenses-groups)

### shared-expenses-auth

Сервис отвечает за пользовательскую аутентификацию и межсервисную авторизацию. Выдает access и refresh токены мобильному приложению, а также предоставляет другим сервисам публичный метод для проверки токена и его обмена на информацию о клиентах. Имеет БД, в которой хранятся данные о пользователях и о сервисах и их доступах. 

### shared-expenses-gateway

Сервис является шлюзом для доступа к бэкенду из мобильного приложения. Все запросы в этот сервис должны содержать пользовательский access-токен, который проверяется в сервисе shared-expenses-auth, затем запрос обогащается данными о пользователе и прокидывается в бизнес-сервис

### shared-expenses-groups

Один из бизнесовых сервисов, отвечает за домен управления группами. Все методы защищены скоупами groups.read или groups.write, используется OAuth 2.0 Client Credentials авторизация через shared-expenses-auth сервис.

## Выполнение требований

### Группа требований 1

| Требование | Выполняется? | Где или как посмотреть |
|------------|--------------|------------------------|
| Приложение поддерживает аутентификацию пользователей и контроль доступа к API | ✅ | Поднять сервисы, открыть сваггер shared-expenses-gateway, увидеть замочек и 401 при попытке отправить запрос без bearer-токена. Создать пользователя в shared-expenses-auth, получить access-токен и выполнить запрос к shared-expenses-gateway с этим токеном |
| Приложение имеет gRPC или HTTP API (минимум четыре бизнес-метода: создание,
получение, изменение, удаление) | ✅ | Не знаю, насколько важно было сделать именно CRUD, но в целом на трех сервисах есть и GET, и POST, и PUT, и DELETE запросы. Посмотреть можно в сваггерах |
| Все сервисы и API покрыты тестами (unit и функциональными)
 | ✅ | В сервисах shared-expenses-gateway и shared-expenses-groups удалось довести до честных 100% - [раз](https://github.com/qwatro2/shared-expenses-gateway/pull/4#issuecomment-4152954470) и [два](https://github.com/qwatro2/shared-expenses-groups/pull/4#issuecomment-4153332071), в shared-expenses-auth до [98.99%](https://github.com/qwatro2/shared-expenses-auth/pull/5#issuecomment-4153327361), потому что есть исключения при работе с токенами, которые не получилось воспроизвести |
| Приложение использует внешнюю БД для хранения пользователей и бизнесинформации | ✅ | например, что в [application.yml](https://github.com/qwatro2/shared-expenses-auth/blob/main/app/src/main/resources/application.yml#L15) используется постгревый connection-url и драйвер |
| Схема базы данных создаётся при запуске или деплое приложения, поддерживается
версионирование схемы | ✅ | используется [Liquibase](https://github.com/qwatro2/shared-expenses-auth/blob/main/app/src/main/resources/db/changelog/db.changelog-master.yaml) |
| Схема базы данных отражается в код при сборке Несоответствие ORM-моделей,
запросов и схемы приводит к ошибке сборки | ❌ | Не реализовывал, потому что вместо ORM использую jdbcTemplate |

Итого, выполнено 5 требований из 6 на 10/3 балла

### Группа требований 2

| Требование | Выполняется? | Где или как посмотреть |
|------------|--------------|------------------------|
| Приложение поддерживает логирование | ✅ | Используется [Slf4j](https://github.com/qwatro2/shared-expenses-auth/blob/main/app/src/main/java/org/qwatro2/shared/expenses/auth/api/aop/InboundRequestAdvice.java#L16), можно поднять сервис, кинуть запрос и увидеть как минимум AOP-логи |
| Приложение поддерживает метрики | ✅ | Используется [io.micrometer:micrometer-registry-prometheus](https://github.com/qwatro2/shared-expenses-auth/blob/main/gradle/libs.versions.toml#L26), можно поднять сервис, кинуть запрос, затем открыть `/actuator/prometheus` и увидеть метрики |
| Приложение может быть запущено в Kubernetes | ✅ | Да, для каждого приложения написаны helm-чарты - [пример](https://github.com/qwatro2/shared-expenses-auth/tree/main/helm/shared-expenses-auth) |
| Поддерживается сборка логов приложения и всех взаимодействующих с приложением
инфраструктурных объектов в Kubernetes | ❌ | Нет, не успеваю( |

Итого, выполнено 3 требования из 4 на 3 балла

### Группа требований 3

| Требование | Выполняется? | Где или как посмотреть |
|------------|--------------|------------------------|
| Каждый коммит в мастер-ветку собирается при помощи CI/CD системы | ✅ | Выполняется, [пример](https://github.com/qwatro2/shared-expenses-auth/actions) |
| По всем API-методам есть Swagger-документация, доступная из приложения | ✅ | Выполняется, в каждом проекте есть сам [openapi-контракт](https://github.com/qwatro2/shared-expenses-auth/blob/main/app/src/main/resources/swagger/shared-expenses-auth.yaml) и доступен сваггер по пути `/swagger-ui/index.html` |

Итого, выполнено 2 требования из 2 на 2 балла
