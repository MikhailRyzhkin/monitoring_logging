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

![Dashboards-1](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/1db8864c-3061-4826-8979-70429058c9d5)

![Dashboards-2](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/f62856f5-50a3-4f87-9919-60c1e876cc41)


  - Смотрим логи нашего приложения в нэймспейсе diplom из этой же самой grafana:

![logging-loki](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/e3299c1d-7882-4b9c-8cbf-8d425a2f2d04)


  - Адрес grafana для просмотра логов и визуализированных метрик:
  ```
  http://51.250.80.225:3000/
  ```
  - В docker-compouse.yaml файле на сервере srv вносим свои данные телеграмм, перезапускаем контейнер и моделируем срабатывание аллерта

![токены телеги](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/f58fe588-6c37-45ce-a74a-d0a9467b598f)

![telegramm-allert](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/72218f65-d511-46b0-955d-77eb18783f1a)

![Оповещение в телеграм](https://github.com/MikhailRyzhkin/monitoring_logging/assets/69116076/0334c554-4c17-45bd-be18-70a89af13a7d)











