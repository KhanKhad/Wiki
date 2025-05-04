1. Создаем папку xray_checker.

2. Создаем docker-compose.yml со следующим конфигом:

services:
  xray-checker:
    image: kutovoys/xray-checker
    environment:
      - SUBSCRIPTION_URL=
      - METRICS_BASE_PATH=/checker
    ports:
      - "2112:2112"

  prometheus:
    image: prom/prometheus:latest
    depends_on:
      - xray-checker
    volumes:
      - ./prometeus/configuration/:/etc/prometheus/
      - ./prometeus/data/:/prometheus/
    command:
      - --config.file=/etc/prometheus/prometheus.yml
      - --web.external-url=/prometheus/
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana
    user: root
    depends_on:
      - prometheus
    ports:
      - 3000:3000
    volumes:
      - ./grafana:/var/lib/grafana
      - ./grafana/provisioning/:/etc/grafana/provisioning/
    environment:
      - GF_SERVER_ROOT_URL=http://localhost:3000/grafana
      - GF_SERVER_SERVE_FROM_SUB_PATH=true


3. Настраиваем рутинг в nginx:

#metrics
#location /checker {
#	proxy_redirect off;
#	proxy_set_header Host $host;
#	proxy_set_header X-Real-IP $remote_addr;
#	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
#	proxy_pass  http://127.0.0.1:2112;
#	break;
#}
#location /prometheus {
#	proxy_redirect off;
#	proxy_set_header Host $host;
#	proxy_set_header X-Real-IP $remote_addr;
#	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
#	proxy_pass  http://127.0.0.1:9090;
#	break;
#}

location /grafana {
	proxy_redirect off;
	proxy_set_header Host $host;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_pass  http://127.0.0.1:3000;
	break;
}

4. В каталоге xray_checker создаем папки /grafana и /prometeus

5. В каталоге /prometeus создаем две папки /configuration и /data

6. В каталоге /configuration создаем файл prometheus.yml

scrape_configs:
  - job_name: "xray-checker"
    metrics_path: "/checker/metrics"
    static_configs:
      - targets: ["xray-checker:2112"]
    scrape_interval: 1m

7. переходим в каталог /prometeus и меняем права data: chown 65534:65534 data

8. Переходим в каталог xray_checker и прописываем docker compose up -d

Если что-то идет не так, идем сюда(https://dockerhosting.ru/blog/zapusk-prometheus-v-docker/#%D0%97%D0%B0%D0%BF%D1%83%D1%81%D0%BA_Prometheus_Node_Exporter_%D0%B2_Docker) (Есть копия)
и сюда (https://xray-checker.kutovoy.dev/)


Отслеживание статуса сервера, просмотр статистики и отправка уведомлений в Telegram(если сервер упал или встал) с помощью Grafana, Prometheus и Node Exporter
http://docs.google.com/document/d/13J3ojlHjtEfmNgNKTxxRNiTx1AugFXPN-c6Ap5XwGl8/edit?tab=t.0#heading=h.kd6ffmluf0ct. Есть копия в соседней папке