# Stratum proxy
* Поддержка разных алгоритмов майнинга через один порт.
* Поддержка майнинга несколькими майнерами на 1 аккаунт.
* Счетчики шар для каждого майнера, пользователя, пула и алгоритма.
* Хэшрейт каждого майнера.
* Автоматическое определение алгоритма майнинга для правильного расчета хэшрейта.
* Регистрация на прокси через API.
* Метрики в стандартном формате Prometheus.

# Поддерживаемые пулы.
Автоматическое определение алгоритма майнинга происходит на основе пары <pool_host>:<pool_port>, поэтому прокси поддерживает подключение только к определенному набору пулов, сохраненному в базе данных. API для расширения списка алгоритмов и пулов пока отсутствует.

# REST API управления.
REST API доступно по адресу `http://<web.addr>/api/v1` прокси и сейчас в API есть только 1 команда для регистрации учетных данных для подключения к пулу.
### POST /users
Передаваемые данные:
```json
{
    "pool": "<host>:<port>",
    "user": "<username>",
    "password": "<password>"
}
```
Ответ придет в виде:
```json
{
    "name": "<name>",
    "error": ""
}
```
Полученный `name` используется для того, чтобы прокси опознал подключение и подключился к правильному пулу с правильным аккаунтом. Строка подключения к прокси будет выглядеть так:
```
-o stratum+tcp://<proxy_host>:<proxy_stratum_port> -u <name> -p <любой, игнорируется>
```
Учетные записи не удаляются (в дальнейшем планируется сделать автоматическое удаление после периода бездействия).

# Доступные метрики.
Метрики доступны по адресу `http://<web.addr>/metrics` и включают в себя набор стандартных метрик `Prometheus` и кастомные метрики для мониторинга работы воркеров.
## Список кастомных метрик.
* `proxy_worker_up{"proxy"="<proxy_host>:<proxy_port>", "worker"="<worker_host>:<worker_port>", "user"="<name>"}` - статус воркера. Появляется при подключении воркера к прокси.
* `proxy_pool_up{"proxy"="<proxy_host>:<proxy_port>", "hash"="<hash>", "pool"="<pool_host>:<pool_port>"}` - статус пула. Появляется при подключении прокси к пулу.
* `proxy_worker_sended{"proxy"="<proxy_host>:<proxy_port>", "worker"="<worker_host>:<worker_port>", "user"="<name>", "hash"="<hash>", "pool"="<pool_host>:<pool_port>"}` - счетчик шар, отправленных майнером.
* `proxy_worker_accepted{"proxy"="<proxy_host>:<proxy_port>", "worker"="<worker_host>:<worker_port>", "user"="<name>", "hash"="<hash>", "pool"="<pool_host>:<pool_port>"}` - счетчик шар, принятых пулом.
* `proxy_worker_speed{"proxy"="<proxy_host>:<proxy_port>", "worker"="<worker_host>:<worker_port>", "user"="<name>", "hash"="<hash>", "pool"="<pool_host>:<pool_port>"}` - скорость воркера в хэшах в секунду. Окно измерения хэшрейта - 5 минут, интервал измерения  - 1 минута.
* `proxy_worker_difficulty{"proxy"="<proxy_host>:<proxy_port>", "worker"="<worker_host>:<worker_port>", "user"="<name>", "hash"="<hash>", "pool"="<pool_host>:<pool_port>"}` - сложность, установленная пулом для воркера.
