# Домашнее задание к занятию "`Кластеризация и балансировка нагрузки`" - `Поздникин Евгений`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

### Задание 1

Конфиг файл /etc/haproxy/haproxy.cfg
```
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256>
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

listen stats  # веб-страница со статистикой
        bind                    :888
        mode                    http
        stats                   enable
        stats uri               /stats
        stats refresh           5s
        stats realm             Haproxy\ Statistics

listen web_tcp

        bind :1325
        mode tcp
        balance roundrobin
        server s1 127.0.0.1:8888 check inter 3s
        server s2 127.0.0.1:9999 check inter 3s
```

##### Видим перенаправление запросов на разные серверы при обращении к HAProxy
![Console](https://github.com/PozdnikinEugene/sflt-homeworks-2.md/blob/main/img/1-1.png)

##### Статистика в веб панеле :888/stats

![web](https://github.com/PozdnikinEugene/sflt-homeworks-2.md/blob/main/img/1-2.png)



<details>
  <summary>Запуск simple python сервера 1 </summary>
  
  ```
  cd;\
[ -d http1 ] || mkdir http1; cd http1;\
[ -f index.html ] || echo "Server 1 Port 8888" > index.html;\
python3 -m http.server 8888 --bind 0.0.0.0
  ```
</details>

<details>
  <summary>Запуск simple python сервера 2 </summary>
  
  ```
  cd;\
[ -d http1 ] || mkdir http1; cd http1;\
[ -f index.html ] || echo "Server 1 Port 8888" > index.html;\
python3 -m http.server 8888 --bind 0.0.0.0
  ```
</details>

#### Установка Haproxy
 ```
#apt install haproxy
 ```
<details>
  <summary>Внесение доп настроек в файл конфигурации Haproxy </summary>
  
  ```
echo "
listen stats  # веб-страница со статистикой
        bind                    :888
        mode                    http
        stats                   enable
        stats uri               /stats
        stats refresh           5s
        stats realm             Haproxy\ Statistics

listen web_tcp

        bind :1325
        mode tcp
        balance roundrobin
        server s1 127.0.0.1:8888 check inter 3s
        server s2 127.0.0.1:9999 check inter 3s"  >> /etc/haproxy/haproxy.cfg
  ```
</details>

#### Перечитать конфигурацию после внесения изменений 

```
# systemctl reload haproxy
```

<details>
  <summary>Цикл curl </summary>
  
  ```
for i in {1..10}; do curl http://localhost:1325; sleep 2; done
  ```
</details>



---

### Задание 2


Конфиг файл /etc/haproxy/haproxy.cfg 
```
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256>
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

listen stats  # веб-страница со статистикой
        bind                    :888
        mode                    http
        stats                   enable
        stats uri               /stats
        stats refresh           5s
        stats realm             Haproxy\ Statistics

frontend example  # секция фронтенд
        mode http
        bind :8088
    acl ACL_example.local hdr(host) -i example.local
    use_backend web_servers if ACL_example.local

backend web_servers    # секция бэкенд
        mode http
        balance roundrobin
        option httpchk
        http-check send meth GET uri /index.html
        server s1 127.0.0.1:8888 check weight 2
        server s2 127.0.0.1:9999 check weight 3
        server s3 127.0.0.1:7777 check weight 4
```

##### Видим перенаправление запросов на разные серверы при обращении к HAProxy по домену, по отличаюемуся домену так и без него. 
![Console](https://github.com/PozdnikinEugene/sflt-homeworks-2.md/blob/main/img/2-1.png)

##### Статистика в веб панеле :888/stats. Видна статистика и веса

![web](https://github.com/PozdnikinEugene/sflt-homeworks-2.md/blob/main/img/2-2.png)

 <details>
  <summary>Запуск simple python сервера 3 </summary>
  
  ```
cd;\
[ -d http1 ] || mkdir http1; cd http1;\
echo "Server 3 Port 7777" > index.html;\
python3 -m http.server 7777 --bind 0.0.0.0
  ```
</details>

<details>
  <summary> Внесение доп настроек в файл конфигурации Haproxy </summary>
  
  ```
frontend example  # секция фронтенд
        mode http
        bind :8088
	acl ACL_example.local hdr(host) -i example.local
	use_backend web_servers if ACL_example.local

backend web_servers    # секция бэкенд
        mode http
        balance roundrobin
        option httpchk
        http-check send meth GET uri /index.html
        server s1 127.0.0.1:8888 weight 2 check
        server s2 127.0.0.1:9999 weight 3 check
        server s3 127.0.0.1:7777 weight 4 check
  ```
</details>

#### Перечитать конфигурацию после внесения изменений 

```
# systemctl reload haproxy
```

<details>
  <summary>Цикл curl </summary>
  
  ```
 for i in {1..10}; do curl -H 'Host:example.local' http://localhost:8088; sleep 2; done
  ```
</details>
