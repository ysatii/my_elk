# Домашнее задание к занятию «ELK» - Мельник Юрий Александрович




## Задание 1. Elasticsearch
 

1. `Установите и запустите Elasticsearch, после чего поменяйте параметр cluster_name на случайный.`

### Приведите скриншот команды 'curl -X GET 'localhost:9200/_cluster/health?pretty', сделанной на сервере с установленным Elasticsearch. Где будет виден нестандартный cluster_name.

## Решение 1.  
Создадим машину на яндекс облаке с операционной системой Debian 10  

произведем ручную установку согласно инструкии https://www.elastic.co/guide/en/elasticsearch/reference/7.17/deb.html

```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.22-amd64.deb
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.22-amd64.deb.sha512
shasum -a 512 -c elasticsearch-7.17.22-amd64.deb.sha512 
sudo dpkg -i elasticsearch-7.17.22-amd64.deb
sudo -i service elasticsearch start
sudo -i service elasticsearch stop
```

![alt text](https://github.com/ysatii/my_elk/blob/main/img/image1.jpg)  
![alt text](https://github.com/ysatii/my_elk/blob/main/img/image1_2.jpg)  

Изменим имя кластера  
![alt text](https://github.com/ysatii/my_elk/blob/main/img/image1_3.jpg)  
![alt text](https://github.com/ysatii/my_elk/blob/main/img/image1_4.jpg)  

проверим работоспособность
![alt text](https://github.com/ysatii/my_elk/blob/main/img/image1_5.jpg)  
     

## Задание 2. Kibana
 
1. `Установите и запустите Kibana.`
### Приведите скриншот интерфейса Kibana на странице http://<ip вашего сервера>:5601/app/dev_tools#/console, где будет выполнен запрос GET /_cluster/health?pretty.



## Решение 2
 Установим и запустим Kibana.
```
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.17.22-amd64.deb
 
sudo dpkg -i kibana-7.17.22-amd64.deb
sudo systemctl start kibana.service
sudo systemctl enable kibana.service
```
![alt text](https://github.com/ysatii/my_elk/blob/main/img/image2.jpg)  

Исправим конфигурационный файл для прослушивания всех ip  
![alt text](https://github.com/ysatii/my_elk/blob/main/img/image2_1.jpg)  

Войдем в web интерфейс Kibana.  
![alt text](https://github.com/ysatii/my_elk/blob/main/img/image2_2.jpg)  

выполним запрос GET /_cluster/health?pretty.  
![alt text](https://github.com/ysatii/my_elk/blob/main/img/image2_3.jpg)  

## Задание 3. Logstash

1. `Установите и запустите Logstash и Nginx. С помощью Logstash отправьте access-лог Nginx в Elasticsearch.`

### Приведите скриншот интерфейса Kibana, на котором видны логи Nginx.

## Решение 3
1. `Создадим docker-compose файл для решения данной задачи`
```
 version: "3.9"
services:
  elasticsearch:
    image: elasticsearch:8.12.2
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - 9200:9200
    volumes:
      - ./deploy/esdata:/usr/share/elasticsearch/data
    # helth curl -s http://127.0.0.1:9200/_cluster/health
    # indexes curl http://127.0.0.1:9200/_cat/indices/ 

  kibana:
    image: kibana:8.12.2
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200 

  logstash:
    image: logstash:8.12.2
    user: root #сменил дефолтного юзера на рута
    environment:
       
      ES_HOST: "elasticsearch:9200"
    ports:
      - "5044:5044/udp"
    volumes:
      - ./configs/logstash/config.yml:/usr/share/logstash/config/logstash.yml
      - ./configs/logstash/pipelines.yml:/usr/share/logstash/config/pipelines.yml
      - ./configs/logstash/pipelines:/usr/share/logstash/config/pipelines
      - ./nginxlog:/usr/share/logstash/nginx
    depends_on:
          - elasticsearch
          - nginx
    
  nginx:
    image: nginx:1.25
    ports:
      - 80:80
    volumes:
      - ./nginxlog:/var/log/nginx 
    restart: always
```

Рассмотри кофигурацию nginx  
nginx:  
......  
volumes:  
      - ./nginxlog:/var/log/nginx - служит пробрасыания папки с логами в том nginxlog  



Рассмотри кофигурацию  logstash  
Секция input настроена на изменеия в файле access.log сервиса nginx  
Секция  filter настроена на обработку формата лога nginx, удаляя поле host, для анализа статусов в логах оно не нужно, 
Секция output передает данные в http://elasticsearch:9200 в сервис elasticsearch  
index => "nginx-%{+YYYY.MM.dd}" - создает индекс  
```
input {
  file {
    path => "/usr/share/logstash/nginx/access.log"
    start_position => "beginning"
  }
}


filter {
    grok {
        match => { "message" => "%{IPORHOST:remote_ip} - %{DATA:user_name}
        \[%{HTTPDATE:access_time}\] \"%{WORD:http_method} %{DATA:url}
        HTTP/%{NUMBER:http_version}\" %{NUMBER:response_code} %{NUMBER:body_sent_bytes}
        \"%{DATA:referrer}\" \"%{DATA:agent}\"" }
    }
    mutate {
        remove_field => [ "host" ]
    }
}

output {
  stdout {}
  elasticsearch {
    hosts => "http://elasticsearch:9200"
    index => "nginx-%{+YYYY.MM.dd}"
    # data_stream => "true"
  }
}

```
 
Нужные нам контейнеры в работе!! 
![alt text](https://github.com/ysatii/my_elk/blob/main/img/image3.jpg)  


обновим страницу http://localhost/  
![alt text](https://github.com/ysatii/my_elk/blob/main/img/image3_1.jpg)  
данное действие запишет новую строку в лог файл ngnix!  
Также logstash отправит информацию в elasticsearch  

просмотрим на лог контейнера logstash   
![alt text](https://github.com/ysatii/my_elk/blob/main/img/image3_2.jpg)  


В интерфейсе видим новый индекс nginx-2024.07.21  
![alt text](https://github.com/ysatii/my_elk/blob/main/img/image3_3.jpg)  


Создадим новое представление для данных  
![alt text](https://github.com/ysatii/my_elk/blob/main/img/image3_4.jpg)  


Данные поступают, это нужный нам лог /usr/share/logstash/nginx/access.log  
![alt text](https://github.com/ysatii/my_elk/blob/main/img/image3_5.jpg)  

## Задание 4. Filebeat

1. `Установите и запустите Filebeat. Переключите поставку логов Nginx с Logstash на Filebeat.`

### Приведите скриншот интерфейса Kibana, на котором видны логи Nginx, которые были отправлены через Filebeat.

## Решение 4











## Задание 5*. Доставка данных

1. `Настройте поставку лога в Elasticsearch через Logstash и Filebeat любого другого сервиса , но не Nginx. Для этого лог должен писаться на файловую систему, Logstash должен корректно его распарсить и разложить на поля.`

### Приведите скриншот интерфейса Kibana, на котором будет виден этот лог и напишите лог какого приложения отправляется.

## Решение 5
