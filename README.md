# Описание

Асинхронный микросервис выполняющий одновременно 3 задачи.

## Использованные библиотеки
asyncio, aiohttp, argparse, logging, json, requests.

## Выполняемые задачи

1. Сервис в одном асинхронном потоке получает раз в `N` минут из открытого источника (https://www.cbr-xml-daily.ru/daily_json.js) данные о курсе доллара, рубля и евро (по-умолчанию). `N` передаётся скрипту параметром вида `--period N`.

2. В аргументах скрипту так же передаётся начальный объём средств для каждой из валют в произвольном порядке для каждой из используемых валют.
Примеры запуска:

`$ python3 test.py --rub 1000 --usd 2000 --eur 3000 --period 10`

`$ python3 test.py --eur 52.5 --period 5 --rub 23.1 --usd 234.77`

3. В качестве дополнительного параметра может передаваться `--debug` с возможными значениями из списка: `0, 1, true, false, True, False, y, n, Y, N`.
В случае если параметр debug принимает положительное значение, выводится содержимое request/response для апи в консоль. В противном случае выводится сообщение о старте приложения, об успешном получении данных о курсах валют и о общей сумме средств (п.5). Используются разные уровни логирования (DEBUG, INFO, WARNING и т.п.).

4. Во втором асинхронном потоке сервер отвечает на HTTP запросы на порту 8080. Реализовано REST api, отвечающее на запросы следующего вида:

| тип запроса, url, payload | комментарий |
| ------------ | ------------ |
| GET /usd/get |
| GET /rub/get |
| GET /eur/get |
| GET /amount/get |
| POST /amount/set {"usd":10} |
| POST /amount/set {"rub":100.5, "eur":10, "usd":20} |
| POST /modify {"usd":5} | добавляет к текущему количеству usd 5 |
| POST /modify {"eur":10, "rub":-20} | добавляет к текущему количеству eur 10, уменьшает текущее количество rub на 20 |

На запрос `/amount/get` отвечает общей суммой средств для каждой из трёх валют с учётом текущего курса, количеством каждой из валют отдельно и текущим курсом. Разница в курсе покупки/продажи принебрегается.
Пример вывода:
```
rub: 100
usd: 200
eur: 300

rub-usd: 65.5
rub-eur: 73.4
usd-eur: 1.12

sum: 35220.0 rub / 537.52 usd / 479.93 eur
```

5. В третьем асинхронном потоке раз в минуту выводить в консоль те же данные, что в п.4, в случае, если изменился курс какой-либо из валют или количество средств относительно предыдущего вывода в консоль.

6. При инициализации есть возможность передать наименования валют.
