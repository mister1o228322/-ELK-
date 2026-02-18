# Задание 1

* Установка Elasticsearch:

Из-за проблем с репозиториями (ошибка 403 Forbidden) использовали Docker для установки

* Запустили контейнер Elasticsearch 7.17.15:

sudo docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 \
  -e "discovery.type=single-node" -e "xpack.security.enabled=false" \
  -e "cluster.name=netology_hw3_example" docker.elastic.co/elasticsearch/elasticsearch:7.17.15

* Настройка cluster_name:

Изменили имя кластера на netology_hw3_example через переменную окружения

* Проверка:

Выполнили curl -X GET 'localhost:9200/_cluster/health?pretty'

Получили ответ с "cluster_name" : "netology_hw3_example" и статусом GREEN

<img width="596" height="300" alt="image" src="https://github.com/user-attachments/assets/c5629fce-309e-4b04-bfed-3f607bf317b1" />

# Задание 2

* Установка Kibana:

* Столкнулся с проблемами DNS при подключении к Elasticsearch

* Решил через создание общей сети Docker и использование правильного IP

* Запустили Kibana в контейнере:


sudo docker run -d --name kibana --network elastic -p 5601:5601 \
  -e "ELASTICSEARCH_HOSTS=http://elasticsearch:9200" docker.elastic.co/kibana/kibana:7.17.15

* Проверка:

Дождались сообщения в логах "Kibana is now available"

* Открыли веб-интерфейс по адресу http://192.168.89.16:5601

* В Dev Tools выполнили запрос GET /_cluster/health?pretty

* Получили ответ с информацией о кластере

<img width="2557" height="1347" alt="image" src="https://github.com/user-attachments/assets/0d8f3bb9-9e67-4563-bbc8-bcefe2e5f527" />

<img width="2555" height="1350" alt="image" src="https://github.com/user-attachments/assets/ee107658-64f1-4fab-9f12-63d8cb79ccf3" />

<img width="2559" height="1310" alt="image" src="https://github.com/user-attachments/assets/3c51fceb-b620-4e06-a202-54b65bb0a57a" />

# Задание 3

* установка Nginx:

sudo apt update

sudo apt install nginx -y

Сгенерировали тестовые запросы для создания логов

* Установка Logstash:

Создали конфигурационный файл для парсинга логов Nginx с помощью grok

Настроили input для чтения /var/log/nginx/access.log

Настроили output для отправки в Elasticsearch

Запустили Logstash в Docker:


sudo docker run -d --name logstash --network elastic \
  -v ~/logstash-config:/usr/share/logstash/pipeline/ \
  -v /var/log/nginx:/var/log/nginx:ro docker.elastic.co/logstash/logstash:7.17.15

* Проблемы и решения:

Столкнулись с проблемой DNS (контейнеры в разных сетях)

Решили подключением Elasticsearch к сети elastic

Исправили права доступа к логам Nginx

Результат:

Logstash успешно читает логи и отправляет их в Elasticsearch

Создан индекс nginx-logs-logstash-2026.02.18 с 115 документами

<img width="2557" height="1342" alt="image" src="https://github.com/user-attachments/assets/7b17a6cc-cb00-4976-9903-17c421b4bfa2" />

<img width="2559" height="1141" alt="image" src="https://github.com/user-attachments/assets/fd74a3f2-3124-46bb-8602-adef4b817edf" />

# Задание 4

* Установка Filebeat:

Создали конфигурацию для чтения логов Nginx и отправки напрямую в Elasticsearch

Запустили Filebeat в Docker:


sudo docker run -d --name filebeat --network elastic \
  -v ~/filebeat-config/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro \
  -v /var/log/nginx:/var/log/nginx:ro docker.elastic.co/beats/filebeat:7.17.15

* Проблемы и решения:

Ошибка прав доступа к конфигурационному файлу (решили chmod 644)

Конфликт маппинга поля service (переименовали в log_source)

Проблемы с ILM (отключили для кастомных индексов)

* Результат:

Filebeat успешно отправляет логи в Elasticsearch

Создан индекс filebeat-7.17.15-2026.02.18-000001 со 112 документами

<img width="2555" height="1334" alt="image" src="https://github.com/user-attachments/assets/f5eb9069-0fbd-4e68-8790-524c2b165e25" />

<img width="2559" height="1306" alt="image" src="https://github.com/user-attachments/assets/f594901e-f1a1-441c-b3ea-8324ab67cd3d" />
