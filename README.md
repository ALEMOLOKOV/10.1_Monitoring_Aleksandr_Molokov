# 10.1_Monitoring_Aleksandr_Molokov

1. Вас пригласили настроить мониторинг на проект. На онбординге вам рассказали, что проект представляет из себя 
платформу для вычислений с выдачей текстовых отчетов, которые сохраняются на диск. Взаимодействие с платформой 
осуществляется по протоколу http. Также вам отметили, что вычисления загружают ЦПУ. Какой минимальный набор метрик вы
выведите в мониторинг и почему?

# Ответ
Исходя из условий в мониторинг я бы добавил следующий минимальный набор метрик
 - общая загрузка CPU (CPU LA) (т.к. указано что при работе приложения идет загруцка CPU)
 - колличество загруженной и свободной RAM (RAM /swap) (т.к. влияет на быстродействои выполнения операций)
 - нагрузка на диск, число операций с диском (IOPS) (т.к. отчеты сохраняются на диск необходимо контролировать состояние диска)
 - остаточная емкость диска (FS) (т.к. отчеты сохраняются на диск, необходим видеть оставшийся объем)
 - состояние диска (inods) (т.к. отчеты сохраняются на диск, необходимо контролировать состояние диска)
 - общее количество http запросов к приложению (контроль нагрузки сети)
 - количество успешно выданных ответов (анализ работы платформы с точки зрения бизнеса)
 - количество неуспешных ответов пользователям (ошибки 4хх и 5хх) (анализ работы платформы с точки зрения бизнеса, пердоставление информации и контроль за простоем системы из-за проведения плановых и внеплановых ремонтных работ)
 
2. Менеджер продукта посмотрев на ваши метрики сказал, что ему непонятно что такое RAM/inodes/CPUla. Также он сказал, 
что хочет понимать, насколько мы выполняем свои обязанности перед клиентами и какое качество обслуживания. Что вы 
можете ему предложить?

# Ответ
Я бы предложил добавить метрики для оценки качества обслуживания. 
Необходимо утвердить SLA (соглашение об уровне обслуживания) в рамках которого будут указаны SLO (целевой уровень качества обслуживания) для тех или иных метрик. После чего менеджерам будет проще ориентироваться в состоянии продукта, так как их будут интересовать только разницы значений SLO и SLI (показатель качества обслуживания). Если значения SLI той или иной метрики не противоречат установленным для неё SLO тогда проект в норме. Это позволяет менеджеру не зная для чего та или иная метрика, но позволяет им понять общую картину сотояния работоспособности проекта.

3. Вашей DevOps команде в этом году не выделили финансирование на построение системы сбора логов. Разработчики в свою 
очередь хотят видеть все ошибки, которые выдают их приложения. Какое решение вы можете предпринять в этой ситуации, 
чтобы разработчики получали ошибки приложения?

# Ответ
Я бы предложил написать собственный скрипт для сбора логов.

4. Вы, как опытный SRE, сделали мониторинг, куда вывели отображения выполнения SLA=99% по http кодам ответов. 
Вычисляете этот параметр по следующей формуле: summ_2xx_requests/summ_all_requests. Данный параметр не поднимается выше 
70%, но при этом в вашей системе нет кодов ответа 5xx и 4xx. Где у вас ошибка?

# Ответ
Для более точного вычисления необходимо добавить коды 1хх и 3хх
Формула должна выглядеть следующим образом т.к. SLA должна быть на одном уровне с SLI
(summ_1xx_requests + summ_2xx_requests + summ_3xx_requests)/summ_all_requests

5. Опишите основные плюсы и минусы pull и push систем мониторинга.
#
6. Какие из ниже перечисленных систем относятся к push модели, а какие к pull? А может есть гибридные?

    - Prometheus 
    - TICK
    - Zabbix
    - VictoriaMetrics
    - Nagios
#
7. Склонируйте себе [репозиторий](https://github.com/influxdata/sandbox/tree/master) и запустите TICK-стэк, 
используя технологии docker и docker-compose.

В виде решения на это упражнение приведите выводы команд с вашего компьютера (виртуальной машины):

    - curl http://localhost:8086/ping
    - curl http://localhost:8888
    - curl http://localhost:9092/kapacitor/v1/ping

А также скриншот веб-интерфейса ПО chronograf (`http://localhost:8888`). 

P.S.: если при запуске некоторые контейнеры будут падать с ошибкой - проставьте им режим `Z`, например
`./data:/var/lib:Z`
#
8. Перейдите в веб-интерфейс Chronograf (`http://localhost:8888`) и откройте вкладку `Data explorer`.

    - Нажмите на кнопку `Add a query`
    - Изучите вывод интерфейса и выберите БД `telegraf.autogen`
    - В `measurments` выберите mem->host->telegraf_container_id , а в `fields` выберите used_percent. 
    Внизу появится график утилизации оперативной памяти в контейнере telegraf.
    - Вверху вы можете увидеть запрос, аналогичный SQL-синтаксису. 
    Поэкспериментируйте с запросом, попробуйте изменить группировку и интервал наблюдений.

Для выполнения задания приведите скриншот с отображением метрик утилизации места на диске 
(disk->host->telegraf_container_id) из веб-интерфейса.
#
9. Изучите список [telegraf inputs](https://github.com/influxdata/telegraf/tree/master/plugins/inputs). 
Добавьте в конфигурацию telegraf следующий плагин - [docker](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/docker):
```
[[inputs.docker]]
  endpoint = "unix:///var/run/docker.sock"
```

Дополнительно вам может потребоваться донастройка контейнера telegraf в `docker-compose.yml` дополнительного volume и 
режима privileged:
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

После настройке перезапустите telegraf, обновите веб интерфейс и приведите скриншотом список `measurments` в 
веб-интерфейсе базы telegraf.autogen . Там должны появиться метрики, связанные с docker.

Факультативно можете изучить какие метрики собирает telegraf после выполнения данного задания.
