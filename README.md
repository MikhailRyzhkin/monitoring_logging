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

  - Для сборки логов будем использовать Loki: https://artifacthub.io/packages/helm/grafana/loki?modal=install
  - Добавляем репозиторий Loki и перечитываем репозитории:
  ```
  helm repo add grafana https://grafana.github.io/helm-charts && helm repo update
  ```
  Устанавливаем Loki в кластер k8s из helm chart:
  ```
  helm install --namespace loging --create-namespace loki grafana/loki --version 5.8.9
  ```
  
  - Для мониторинга кластера и приложения будем использовать Prometheus stack: https://artifacthub.io/packages/helm/prometheus-community/prometheus?modal=install
  - Добавляем репозиторий Prometheus stack и перечитываем репозитории:
  ```
  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts && helm repo update
  ```
  - Устанавливаем Prometheus stack в кластер k8s из helm chart:
  ```
  helm install --namespace monitoring --create-namespace prometheus prometheus-community/prometheus --version 23.1.0
  ```
   
  - Проверяем как установился наш стэк логирования и мониторинга в кластере k8s:
  ```
  kubectl get service -n loging && kubectl get service -n monitoring && kubectl get po -A
  ```

  - И смотрим как работают сервисы:

  - Так как по условию задачи весь мониторинг должен находиться на srv-сервере, а не в кластере k8s, то наиболее простой вариант развернуть кластер мониторинга в docker на этом сервере вне кластера k8s. 
  Стэк мониторинга - Grafana\Prometheus\Blackbox\Node Exporter\Alertmanager
  - В каталоге prometheus_stack описываем через docker compose весь наш стэк мониторинга
  - Находясь в каталоге prometheus_stack с файлом docker-compose.yml запускаем развёртывание стэка:
  ```
  docker compose up -d 
  ```
  - При удачном развёртывании стэка увидем контейнеры компонентов:
  ```
  docker ps -a 
  ```

  - И сами сервисы:
  
  - Подключаем как сервер srv, так и кластер k8s к визуализации метрик Grafana:

