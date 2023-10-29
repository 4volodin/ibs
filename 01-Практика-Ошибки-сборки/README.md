> #### Опишите словами причину ошибок и возможные пути их устранения

### 01. Ошибки в сборке npm (EPIPE)
Прежде всего следует обратить внимание на следущий участок кода:
```
[2023-07-25T15:36:36.499Z] [91mnpm ERR! code EPIPE
[2023-07-25T15:36:36.499Z] [0m[91mnpm [0m[91mERR![0m[91m syscall write
[2023-07-25T15:36:36.499Z] [0m[91mnpm ERR! errno -32
[2023-07-25T15:36:36.499Z] [0m[91mnpm ERR! write EPIPE

```
 Появление этой ошибки может быть вызвано многими факторами:

    - она указывает на проблемы с записью в поток, процесс npm был завершен до завершения операции записи
    - нехватка системных ресурсов

#### Troubleshooting:
    - проверить файл журнала /opt/app-root/src/.npm/_logs/2023-07-25T15_36_24_444Z-debug-0.log
    - добавить --verbose для подробного вывода
    - проверить версии зависимости внутри контейнера node, npm
    - проверить что нет записи в файл(директорию), который не существует внутри контейнера
    - googling

#### [Solution](https://stackoverflow.com/questions/74095146/gitlab-ci-npm-err-code-epipe-on-every-build-while-running-npm-install):

добавить --no-audit в (RUN npm install --no-audit && ...).

Установит зависимости из package-lock.json, пропустив аудит безопасности.
Аудит безопасности прерывал запись в поток вероятно из-за уязвимых зависимостей

> кстати, там еще есть ошибка - перед verbose установлено длинное тире, но до нее еще не дошло выполнение (об этом в следующем фиксе - грамотно составленное тестовое задание с внесенными изменениями на реальном проекте)

### 02. Ошибка в сборке npm (eslint)
Очевидная ошибка:
```
[2023-07-26T09:41:50.975Z] [0m[91mnpm[0m[91m ERR! arg[0m[91m Argument starts with non-ascii dash, this is probably invalid: —verbose
```
#### Troubleshooting:

Смотрим строку через hex редактор и действительно там стоит E2 80 94 перед verbose, что на самом деле неверно

Вставлен символ длинное тире через комбинацию на клавиатуре например Shift+Opt+- on macos (на практике это редкость, потому что линтер на стороне разработчика или в CI такое бы не пропустил)

#### Solution:

Заменяем длинное тире на два обычных тире --

### 03.Ошибка в сборке npm

Смотрим с конца и наблюдаем код, который вызвал ошибку:

```
[2023-07-26T06:23:27.975Z] + npm config set registry http://172.27.8.79:8081/repository/npm-group/
[2023-07-26T06:23:27.975Z] + npm config set always-auth true
[2023-07-26T06:23:28.545Z] npm ERR! `always-auth` is not a valid npm option
[2023-07-26T06:23:28.545Z] 
[2023-07-26T06:23:28.545Z] npm ERR! A complete log of this run can be found in: /home/jenkins/.npm/_logs/2023-07-26T06_23_10_371Z-debug-0.log
```
#### Troubleshooting:
```
npm ERR! `always-auth` is not a valid npm option
```
always-auth - это параметр, который обычно настраивается в файле .npmrc вместо использования команды npm config set. Этот параметр указывает, следует ли всегда использовать аутентификацию при доступе к репозиторию.

#### Solution:
установить always-auth в вашем файле ~/.npmrc
`always-auth = true`

### 04. Ошибки в Cradle (404)

По логам приложения, которое делает http запросы, можем видеть ошибки:
```
http-outgoing-11 >> "Content-Type: application/json[\r][\n]"
http-outgoing-11 >> "Content-Length: 224[\r][\n]"
http-outgoing-11 >> "Host: bcecosys-ms-gateway.apps.test.gazprom-neft.local[\r][\n]"
http-outgoing-11 >> "Connection: Keep-Alive[\r][\n]"
http-outgoing-11 >> "User-Agent: Apache-HttpClient/4.5.13 (Java/11.0.17)[\r][\n]"
http-outgoing-11 >> "Cookie: 51b72c07907c594bc63fcb9f59daac11=9a8a11fa2e44f084e210babd3bd1733a[\r][\n]"
...
http-outgoing-11 << "HTTP/1.1 404 Not Found[\r][\n]"
```
#### Troubleshooting:

"HTTP/1.1 404 Not Found": Этот заголовок указывает на статус ответа сервера, который равен 404 (Not Found), что означает, что сервер не смог найти запрошенный ресурс.
Это полезная информация для отладки

#### Solution:

 Вы должны проверить, что вы отправляете правильные данные и URL-адрес, и что сервер настроен правильно для обработки этого запроса.
