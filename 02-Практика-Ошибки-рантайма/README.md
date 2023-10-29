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
