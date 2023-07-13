# monitoring_logging

Спринт 3

ЗАДАЧА

```
1. Настройка сборки логов.

Представьте, что вы разработчик, и вам нужно оперативно получать информацию с ошибками работы приложения.
Выберите инструмент, с помощью которого такой функционал можно предоставить. 
Нужно собирать логи работы пода приложения. Хранить это всё можно либо в самом кластере Kubernetes, либо на srv-сервере.

2. Выбор метрик для мониторинга.

Так, теперь творческий этап. Допустим, наше приложение имеет для нас некоторую важность. 
Мы бы хотели знать, когда пользователь не может на него попасть — время отклика, сертификат, статус код и так далее. 
Выберите метрики и инструмент, с помощью которого будем отслеживать его состояние.
Также мы хотели бы знать, когда место на srv-сервере подходит к концу.
Важно! Весь мониторинг должен находиться на srv-сервере, чтобы в случае падения кластера мы все равно могли узнать об этом.

3. Настройка дашборда.

Ко всему прочему хотелось бы и наблюдать за метриками в разрезе времени. 
Для этого мы можем использовать Grafana и Zabbix — что больше понравилось.

4. Алертинг.

А теперь добавим уведомления в наш любимый мессенджер, точнее в ваш любимый мессенджер. 
Обязательно протестируйте отправку уведомлений. Попробуйте «убить» приложение самостоятельно, и засеките время от инцидента до получения уведомления. 
Если время адекватное, то можно считать, что вы справились с этим проектом!
```

РЕШЕНИЕ

Подзадача 1: Настройка сборки логов.
  - Для сборки логов используем стэк Fluentd/Clickhouse/Loghouse, устанавленный с помощью helm chart.
  - Fluentd (https://www.fluentd.org/) - это инструмент для сбора и обогащения логов из различных источников, а также для маршрутизации их в различные приемники.

  ![Fluentd](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/9c10e9a3-667f-431e-b272-cade49eb000d)

  - Loghouse (https://github.com/flant/loghouse) - это инструмент просмотра логов в кластере. В качестве сборщика логов используется Fluentd, а качестве хранилища данных – Clickhouse (https://clickhouse.tech/).

  ![loghouse](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/4e20d740-6367-402f-895d-f53728942ec1)

  - Устанавливаем репозиторий Loghouse и перечитываем его для обновления:
  ```
  helm repo add loghouse https://flant.github.io/loghouse/charts/  && helm repo update
  ```
  - Смотрим список доступных чартов:
  ```
  helm search repo loghouse
  ```
  ![Установка loghous-0](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/0b181574-6815-4c32-b1ea-9f747d1f60bd)

  - В файле “loghouse-values.yml” мы задаем PVC для хранения логов. Клонируем репозиторий и переходим в него.
  - Установим Loghouse (Fluentd и Clickhouse устанавливаются вместе с ним) в нэймспейс loghouse:
  ```
  helm install --namespace loghouse --create-namespace -f loghouse/loghouse-values.yml loghouse loghouse/loghouse
  ```  
  - Проверяем как установился наш стэк логирования в кластере k8s:
  ```
  kubectl get pods -n loghouse && kubectl get service -n loghouse && kubectl get po -A
  ```

  ![loghouse-1](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/5db00c18-8f18-4e4a-88a4-8bc788689d5e)

  Loghouse web UI будет доступен по ссылке: http://loghouse.local (пользователь: admin, пароль: PASSWORD).
  - Настраиваем дэшборд и для примера попробуем поискать логи нашего тестового приложения с помощью запроса: ~app="app" на diplom


  Подзадача 2: Выбор метрик для мониторинга.
  - Так как по условию задачи весь мониторинг должен находиться на srv-сервере, а не в кластере k8s, то наиболее простой вариант развернуть кластер мониторинга в docker на этом сервере. 
  Стэк мониторинга - Grafana\Prometheus\Blackbox\Node Exporter\Alertmanager
  - В каталоге prometheus_stack описываем через docker compose весь наш стэк мониторинга
  - Находясь в каталоге prometheus_stack с файлом docker-compose.yml запускаем развёртывание стэка:
  ```
  docker-compose up -d 
  ```
  - При удачном развёртывании стэка увидем контейнеры компонентов:

  - Для мониторинга кластера k8s пишем манифесты, расположенные в подкаталоге k8s-monitoring
  - Переходим в этот подкатолог k8s-monitoring и запускаем развёртывание на нодах кластера по этим манифестам:
  ```
  kubectl create namespace monitoring && kubectl apply -f . -n monitoring && kubectl get pods -n monitoring && kubectl get pods -A
  ```
  - Подключаем как сервер srv, так и кластер k8s к визуализации метрик Grafana:

  Подзадача 3: Настройка дашборда.
