## Это финальный 3й Спринт каcающийся Мониторинга и Логирования нашего сервиса и его компонентов

### Первым этапом нам предстоит создать docker-compose файл с необходимыми компонентами и конфигурациями

### Для мониторинга мы будем использовать стек Grafana с такими компонентами как Prometheus и Alertmanager в связки с нужными нам экспорртерами.

Эспорты по списку:
- blackbox-exporter  - Служит для внешней проверки на доступность нашего сервиса Django
- node-exporter - мониторим ифраструктурные метрики самих хостов (CPU, Memory, File System)
- cadvisor - служит для мониторинга Docker контейнеров.

Prometheus выстуавет как DataSource собираем метрик нашими экспортерами. Так же содержит в себе файл конфигураций prometheus/prometheus.yml  c job и target, alert rules. 

### Установка компонентов происходит путем скачивания репозитория https://github.com/4Anchor/sprint3.git

Выполняем команду:

```
docker-compose up -d
```
Смотрим все ли поднялось ине рестартуют ли контейнеры с ошибками

```
docker-compose ps
```
Если на этом этапе все нормально, то уже можно перейти в среду визуализации Grafana:

http://158.160.38.8:3000/

Логин: <отправлю в лс>
Пароль: < отправлю в лс>

Grafana содержит дашборды согласно подключенным к ней экспортерам. 

Alert Rules можно просмотреть на странице Prometheus : http://158.160.38.8:9090/rules
 - мониторим на валидность кода 200 и на общее падение Вэб сайта, в случае срабатывание правила  алерт направится в Alertmanager (http://158.160.38.8:9093/#/alerts) который в свою очередь отправит аллерт в соответствующий телеграм канал.

 В alertmanager.yml требуется подставить свои значения в поле:
 ```
 bot_token:'TOKEN'                                                                                                                                               
 chat_id: -CHAT_ID  
 ```
Результаты аллертов можно будет увидеть на скриншотах прилагаемых к данному спринту. (AlertRule.png, ResolveRule.png)

### На этом этам мониторинга завершен, приступаем к реализации системы логирования.

### Логирование реализованно по средствам использование Grafa Stack Loki + Promtail 

1. Для сбора логов с Kubernetes cluster нам потребуется установить и настроить агента Promtail:

Ссылка на инструкция по инсталяции прилагаю взята из официальной инструкции https://grafana.com/docs/loki/latest/clients/promtail/installation/

Деплой в кластер производился рекомендованым способом DaemonSet (recommended). Манифест к данному проекту находится по ссылке https://github.com/4Anchor/components.git
 
После скачивания репозитория требуется выполнить команды:
Создать namespace monitoring
```
kubectl create namespace monitoring
```
Задеплоить сам promtail:

```
kubectl apply -f promtail.yaml -n monitoring
```
Так же рекомндуется создать отдельную Cluster Role promtail и привязать ее:

```
kubectl apply -f promtail-cluster-role.yaml
kubectl apply -f promtail-cluster-rolebinding.yaml
```
Убедитесь что поды запущены и передают собранные данные на сервер Loki (находится на srv-0)

```
kubectl get pods -n monitoring
```
NAME                       READY   STATUS    RESTARTS       AGE
promtail-daemonset-b7bgc   1/1     Running   0              18h

```
kubectl logs имя_пода -n monitoring
```
Так же не забудьте добавить в Grafana Data sources, в нашем случае Prometheus (http://192.168.10.27:9090) и Loki (http://192.168.10.27:3100)

После чего можно перейти на вкладку Explore выбрать Data sources Loki в списки jobs выбрать gjango-app, и вы увидите все логи которые собирает ваш под и передает на сервис логирования Loki.
http://158.160.38.8:3000/goto/18ZwkbC4R?orgId=1
### На этом настройка этапа сбора логов считается завершенной. 



