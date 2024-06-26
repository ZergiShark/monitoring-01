1. Вас пригласили настроить мониторинг на проект. На онбординге вам рассказали, что проект представляет из себя платформу для вычислений с выдачей текстовых отчетов, которые сохраняются на диск. Взаимодействие с платформой осуществляется по протоколу http. Также вам отметили, что вычисления загружают ЦПУ. Какой минимальный набор метрик вы выведите в мониторинг и почему?

Ответ: метрики по CPU,ОЗУ,состоянию ФС,Доступности сервиса(порта 80 в данном случае),ошибок на сетевом интерфесе.Так же интересно количество запущен процессов в системе.

2. Менеджер продукта посмотрев на ваши метрики сказал, что ему непонятно что такое RAM/inodes/CPUla. Также он сказал, что хочет понимать, насколько мы выполняем свои обязанности перед клиентами и какое качество обслуживания. Что вы можете ему предложить?

Ответ: CPU - нагруженность(задержки на комманд процесса) процессора (нагрузка на приложение). inodes - загруженность файлово системы (нагрузка ПО на формирование отчетов:кол-во отчетов). RAM - загруженность оперативной памяти.

Предолжу заключить SLA c клиентом,внутри опираться на SLO и SLI,определить Error budget а так же начать учитывать MTBF (Mean Time Between Failures) — среднее время между сбоями и MTTR (Mean Time To Recovery) среднее время восстановления.

3. Вашей DevOps команде в этом году не выделили финансирование на построение системы сбора логов. Разработчики в свою очередь хотят видеть все ошибки, которые выдают их приложения. Какое решение вы можете предпринять в этой ситуации, чтобы разработчики получали ошибки приложения?

Ответ: В зависимости от выделеных ресурсов, но думаю на сбор syslog можно найти денег.В идеале Sentry + ELK

4. Вы, как опытный SRE, сделали мониторинг, куда вывели отображения выполнения SLA=99% по http кодам ответов. Вычисляете этот параметр по следующей формуле: summ_2xx_requests/summ_all_requests. Данный параметр не поднимается выше 70%, но при этом в вашей системе нет кодов ответа 5xx и 4xx. Где у вас ошибка?

Ответ: Коды ответов 300-399 - редиректы, так же нужно учитывать как успешные Так же коды 100-199 - информационныие сообщения, которые в свою очередь не являются ошибками. код 2** - разумно использовать как проверку доступности сервиса.

5. Опишите основные плюсы и минусы pull и push систем мониторинга.

Основные плюсы и минусы pull и push систем мониторинга:

Pull модель мониторинга:
Плюсы:
- Гибкость: возможность выбирать конкретные метрики, которые необходимы для сбора.
- Эффективность: сервер получает только те метрики, которые запрашивает.
- Поддержка различных источников мониторинга.

Минусы:
- Задержка в получении метрик: сервер должен отправить запрос на получение метрик.
- Возможность пропуска данных, если сервер мониторинга не запрашивает метрики в нужное время.

Push модель мониторинга:
Плюсы:
- Более быстрое обнаружение проблем: метрики мгновенно отправляются на сервер мониторинга.
- Возможность получения метрик в реальном времени.

Минусы:
- Больше нагрузка на сервер мониторинга: постоянный поток метрик, которые нужно обработать.
- Проблемы с безопасностью: необходимо учесть, что данные метрик могут быть перехвачены или подделаны.

6. Какие из ниже перечисленных систем относятся к push модели, а какие к pull? А может есть гибридные?
    
    - Prometheus
    - TICK
    - Zabbix
    - VictoriaMetrics
    - Nagios

Подходы к мониторингу:

- Prometheus: pull модель.
- TICK: push модель.
- Zabbix: push модель.
- VictoriaMetrics: push модель.
- Nagios: push модель.

Гибридные модели также возможны, например, с использованием Telegraf в связке с другими системами мониторинга, такими как Prometheus или InfluxDB.

7. Склонируйте себе [репозиторий](https://github.com/influxdata/sandbox/tree/master) и запустите TICK-стэк, используя технологии docker и docker-compose.

В виде решения на это упражнение приведите скриншот веб-интерфейса ПО chronograf (`http://localhost:8888`).

P.S.: если при запуске некоторые контейнеры будут падать с ошибкой - проставьте им режим `Z`, например `./data:/var/lib:Z`

Ответ: ![screenshot](/screenshots/1.png)

8. Перейдите в веб-интерфейс Chronograf ([http://localhost:8888](http://localhost:8888/)) и откройте вкладку Data explorer.
    
    - Нажмите на кнопку Add a query
    - Изучите вывод интерфейса и выберите БД telegraf.autogen
    - В `measurments` выберите cpu->host->telegraf-getting-started, а в `fields` выберите usage_system. Внизу появится график утилизации cpu.
    - Вверху вы можете увидеть запрос, аналогичный SQL-синтаксису. Поэкспериментируйте с запросом, попробуйте изменить группировку и интервал наблюдений.

Для выполнения задания приведите скриншот с отображением метрик утилизации cpu из веб-интерфейса.

Ответ: ![screenshot](/screenshots/2.png)
9. Изучите список [telegraf inputs](https://github.com/influxdata/telegraf/tree/master/plugins/inputs). Добавьте в конфигурацию telegraf следующий плагин - [docker](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/docker):

```
[[inputs.docker]]
  endpoint = "unix:///var/run/docker.sock"
```

Дополнительно вам может потребоваться донастройка контейнера telegraf в `docker-compose.yml` дополнительного volume и режима privileged:

```
  telegraf:
    image: telegraf:1.4.0
    privileged: true
    volumes:
      - ./etc/telegraf.conf:/etc/telegraf/telegraf.conf:Z
      - /var/run/docker.sock:/var/run/docker.sock:Z
    links:
      - influxdb
    ports:
      - "8092:8092/udp"
      - "8094:8094"
      - "8125:8125/udp"
```

После настройке перезапустите telegraf, обновите веб интерфейс и приведите скриншотом список `measurments` в веб-интерфейсе базы telegraf.autogen . Там должны появиться метрики, связанные с docker.

Факультативно можете изучить какие метрики собирает telegraf после выполнения данного задания.
Ответ: ![screenshot](/screenshots/3.png)