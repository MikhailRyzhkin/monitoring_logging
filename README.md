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

  - Я отказался от loghouse по причине отсутствия развития проекта. Я отказался от стэка ELK по причине его "тяжеловестности" и ограничений для РФ
  - Для сборки логов будем использовать Loki: https://artifacthub.io/packages/helm/grafana/loki?modal=install

![Loki-архитектура](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/62e4c632-d58e-48aa-be69-4ddbdd9f017f)

  - Loki хорошо вписывается и интегрируется в предполагаемый стэк мониторинга prometheus. Логи и мониторинг будет в одной grafana.
  - Добавляем репозиторий Loki и перечитываем репозитории:
  ```
  helm repo add bitnami https://charts.bitnami.com/bitnami -n logging-monitoring && helm repo update
  ```
  Устанавливаем Loki в кластер k8s из helm chart:
  ```
  helm install --namespace logging-monitoring loki bitnami/grafana-loki --set global.dnsService=coredns --set spec.type=NodePort --set loki.auth_enabled=false
  ```

![logging-loki-0](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/d22e5e8f-abc4-4ca2-b43d-7490a795a79f)

  - Для мониторинга кластера и приложения будем использовать Prometheus stack: https://artifacthub.io/packages/helm/prometheus-community/prometheus?modal=install

![Prometheus архитектура](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/52ce48c5-1263-44a2-88f4-6589cb38c39a)
  
  - Так как, по условию задачи, весь мониторинг должен находиться на srv-сервере, а не в кластере k8s, то наиболее простой вариант развернуть кластер мониторинга в docker на этом сервере srv вне кластера k8s. 
  Стэк мониторинга - Grafana\Prometheus\Blackbox\Node Exporter\Alertmanager\Loki
  - В каталоге prometheus_stack описываем через docker compose весь наш стэк мониторинга
  - Находясь в каталоге prometheus_stack с файлом docker-compose.yml запускаем развёртывание стэка:
  ```
  docker compose up -d 
  ```
  - При удачном развёртывании стэка увидем контейнеры компонентов:
  ```
  docker ps -a 
  ```

![docker-prometheus-1](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/704be404-011e-4bac-9509-830d2f8780e8)
  
  - Подключаем как сервер srv, так и кластер k8s к визуализации метрик и логирования в единой Grafana:

![Prometheus - Data sources](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/0ab25168-5d27-4bf8-870e-df4c2e6e0e10)

  - Из реджистри готовых дэшбордов: https://grafana.com/grafana/dashboards/ выбираем нужные нам дэшборды и по id ставим.

![Dashboards](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/6248ac33-d0d1-4a7a-9d11-c10ed2d004e8)

![Dashboards-1](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/07738b2f-e7f0-4e36-b233-1e578ad9da76)

![Dashboards-2](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/53498f5a-9aaf-4a5d-a8f6-4fac56db5eb9)

  - Смотрим логи нашего приложения в нэймспейсе diplom из этой же самой grafana:

![logging-loki](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/67689a5c-e597-4d3a-be69-3f22d5de3f8e)

  - Адрес grafana для просмотра логов и визуализированных метрик:
  ```
  http://51.250.80.225:3000/
  ```











