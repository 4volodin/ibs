Опишите словами причину ошибок и возможные пути их устранения

### 01. Ошибка в kafkaRuntimeError

Основные ошибки и предупреждение:
```
Couldn't resolve server efi-cluster-kafka-bootsSADtrap:9094 from bootstrap.servers as DNS resolution failed for efi-cluster-kafka-bootsSADtrap
...
Unable to create the publisher or subscriber during initialization: org.apache.kafka.common.KafkaException: Failed to construct kafka consumer
...
org.apache.kafka.common.config.ConfigException: No resolvable bootstrap urls given in bootstrap.servers
```
#### Troubleshooting:

Первопричинная ошибка(хотя в логах стоит что это warning) в том, что kafka customer не смог отрезолвить dns имя и далее не смог создать customer и как следствие 
Failed to start application

#### Solution:

Для решения этой проблемы следует:

    - проверить конфигурацию сервера Kafka в приложении.
    - проверить что свойство bootstrap.servers правильно настроено на фактический адрес сервера Kafka.
    - кроме того, проверить, что DNS-разрешение работает правильно для этого адреса.

### 02. Ошибка в python3

#### Troubleshooting:
Ошибка `"OperationalError at /api/auth/user" с сообщением "FATAL: password authentication failed for user skzs"` вероятно Python не смог подключиться к базе данных из-за неправильных учетных данных для пользователя "skzs".

#### Solution:

    -  проверить в коде настройки подключения к СУБД
    -  проверить учетные данные для skzs

### 03. Ошибки в discovery

Ошибка в логе связана с таймаутом сокета в процессе обмена данными между сервисами через механизм обнаружения (discovery) на базе Netflix Eureka. В частности, ошибка указывает на то, что произошел таймаут при чтении данных из сокета в процессе обмена информацией между узлами Eureka. Это может произойти, когда один из узлов Eureka не может своевременно получить ответ от другого узла:
```
It seems to be a socket read timeout exception, it will retry later. if it continues to happen and some eureka node occupied all the cpu time, you should set property 'eureka.server.peer-node-read-timeout-ms' to a bigger value

```
#### Troubleshooting:

`java.net.SocketTimeoutException` указывает на таймаут сокета, а Read timed out означает, что операция чтения данных из сокета не завершилась в течение установленного времени.

Причины могут быть разнообразные:

    - сетевые проблемы
    - высокая нагрузка
    - настройка времени ожидания

#### Solution:
    - увеличить timeout в Eureka
    - проверить состояние сети
    - поставить на мониторинг узлы
    - масштабировать если нагрузка не достаточна
