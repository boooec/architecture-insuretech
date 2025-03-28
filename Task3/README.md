# Анализ приложения InsureTech

## Проблемы текущей архитектуры

1. Сервис `ins-product-aggregator` обрабатывает запросы синхронно, что приводит к увеличению времени обработки при добавлении новых страховых компаний.

2. Несоответствие периодичности запросов между `core-app` и `ins-comp-settlement`, что вызывает рассинхронизацию данных.

3. Ночная нагрузка на сервис `ins-comp-settlement` резко возрастает. Учитывая рост компании, обработка запросов в будущем будет занимать все больше времени.

4. Сложные взаимодействия и высокая связность между сервисами `core-app`, `ins-comp-settlement` и `ins-product-aggregator`.

## Предлагаемые решения

Переход на Event-Driven архитектуру поможет решить перечисленные проблемы:

- снижение нагрузки за счет обработки данных порциями, по мере их поступления, а не синхронно
- своевременное обновление данных
- снижение связности между компонентами

Использование брокера сообщений позволит сервису `ins-comp-settlement` получать информацию о новых страховках из `core-app` в реальном времени, а не раз в сутки. Необходимо внести изменения в сервис `ins-product-aggregator`, чтобы он самопроизвольно опрашивал внешние сервисы через заданные интервалы. Это обеспечит актуализацию локальных реплик `core-app` и `ins-comp-settlement` без необходимости ожидания ответа от `ins-product-aggregator`. Такой подход также снизит количество ошибок и повысит стабильность системы.

Поскольку `ins-comp-settlement` занимается расчетами и передачей данных о страховых случаях в сторонние компании, важно гарантировать, что сервис получит сообщение о страховке и только один раз. Для этого на стороне `core-app` можно применить паттерн *Transactional Outbox*.
