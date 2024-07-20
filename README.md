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


## Задание 4. Filebeat

1. `Установите и запустите Filebeat. Переключите поставку логов Nginx с Logstash на Filebeat.`

### Приведите скриншот интерфейса Kibana, на котором видны логи Nginx, которые были отправлены через Filebeat.

## Решение 4











## Задание 5*. Доставка данных

1. `Настройте поставку лога в Elasticsearch через Logstash и Filebeat любого другого сервиса , но не Nginx. Для этого лог должен писаться на файловую систему, Logstash должен корректно его распарсить и разложить на поля.`

### Приведите скриншот интерфейса Kibana, на котором будет виден этот лог и напишите лог какого приложения отправляется.

## Решение 5
