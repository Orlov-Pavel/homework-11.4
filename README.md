# Домашнее задание к занятию «Микросервисы: масштабирование»
1. Под перечисленные параметры подходит kubernetes, либо основанные на нём системы:
   Требование | функционал kubernetes
   ---------- | ---------------------
   Поддержка контейнеров | kubernetes поддерживает различные runtime для контейнеров. Обычно это Docker или Containerd
   Обеспечивать обнаружение сервисов и маршрутизацию запросов | В kubernetes есть абстракция services которая динамично пробрасывает заданные порты к нужным контейнерам, кроме того поверх них можно организвовать ingress для проброса в определенный контейнер в зависимости от DNS имени запроса.
   Обеспечивать возможность горизонтального масштабирования | При запуске сервиса в качестве replicaSet или Deployment есть возможность одной командой увеличить или уменьшить количество запущенных pod'ов
   Обеспечивать возможность автоматического масштабирования | Данный функционал покрывается при помощи Horizontal pod autoscaler. Он позволяет увеличить или уменьшить количество подов в зависимости от различных метрик. Например нагрузки на CPU или память. Также, при необходимости, можно организовать автоскейлинг на основе кастомных метрик.
   Обеспечивать явное разделение ресурсов, доступных извне и внутри системы | Kubernetes позволяет ограничить количество ресурсов используемых запущенными контейнерами.
   Обеспечивать возможность конфигурировать приложения с помощью переменных среды, в том числе с возможностью безопасного хранения чувствительных данных таких как пароли, ключи доступа, ключи шифрования и т. п. | В Kubernetes можно использовать явное указание переменных среды в manifest файлах либо в configMap. Также можно использовать секреты для чувствительных данных.

2. За основу взял 3 виртуальных машины.  
   
   Конфигурационный файл для мастер редиса (Одинаковый на всех машинах):
   ```
   port 7000
   dir /var/lib/redis/7000/
   appendonly yes
   protected-mode no
   cluster-enabled yes
   cluster-node-timeout 5000
   cluster-config-file /etc/redis/cluster/7000/nodes_7000.conf
   pidfile /var/run/redis_7000.pid
   ```

   Конфигурационный файл для слейв редиса (Одинаковый на всех машинах):
   ```
   port 7001
   dir /var/lib/redis/7001
   appendonly yes
   protected-mode no
   cluster-enabled yes
   cluster-node-timeout 5000
   cluster-config-file /etc/redis/cluster/7001/nodes_7001.conf
   pidfile /var/run/redis_7001.pid
   ```

   Файл сервиса для мастер редиса (Одинаковый на всех машинах):
   ```
   [Unit]
   Description=Redis cluster for netology
   After=network.target
   
   [Service]
   ExecStart=/usr/bin/redis-server /etc/redis/cluster/7000/redis_7000.conf --supervised systemd
   ExecStop=/bin/redis-cli -h 127.0.0.1 -p 7000 shutdown
   Type=notify
   User=root
   Group=root
   RuntimeDirectory=redis
   RuntimeDirectoryMode=0755
   LimitNOFILE=65535
   
   [Install]
   WantedBy=multi-user.target
   ```

   Файл сервиса для слейв редиса (Одинаковый на всех машинах):
   ```
   [Unit]
   Description=Redis cluster for netology
   After=network.target
   
   [Service]
   ExecStart=/usr/bin/redis-server /etc/redis/cluster/7001/redis_7001.conf --supervised systemd
   ExecStop=/bin/redis-cli -h 127.0.0.1 -p 7001 shutdown
   Type=notify
   User=root
   Group=root
   RuntimeDirectory=/etc/redis/cluster/7001
   RuntimeDirectoryMode=0755
   LimitNOFILE=65535
   
   [Install]
   WantedBy=multi-user.target
   ```

   Инициализация кластера redis:  
   ![redis cluster init](./pictures/redis%20cluster%20init.PNG)  

   Информация по запущенным сервисам редиса:
   ![redis cluster info](./pictures/redis%20cluster%20info.PNG)