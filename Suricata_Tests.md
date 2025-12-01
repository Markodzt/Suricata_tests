
## Integración de Suricata con Loki, Promtail, Grafana   

---

## 1. Resumen del trabajo realizado

### Test 1:
 Suricata, Promtail, Loki, Grafana
### Test 2: 
Filebeat, Elasticsearch y Kibana

---

## Testeo 1 : Suricata - Promtail - Loki - Grafana

###  Preparación del sistema

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget gnupg apt-transport-https software-properties-common unzip
```

---

### Instalación y configuración de Suricata

```bash
sudo apt install -y suricata suricata-update
sudo suricata-update
sudo suricata -T -c /etc/suricata/suricata.yaml
sudo systemctl enable suricata
sudo systemctl restart suricata
```

Rutas de logs relevantes:

```
/var/log/suricata/eve.json
/var/log/suricata/suricata.log
```

---

### Instalación y configuración de Loki

```bash
cd /usr/local/bin
wget https://github.com/grafana/loki/releases/download/v3.0.0/loki-linux-amd64.zip
unzip loki-linux-amd64.zip
chmod +x loki-linux-amd64
sudo mv loki-linux-amd64 loki
```

Directorios:

```bash
sudo mkdir -p /etc/loki
sudo mkdir -p /var/lib/loki/{index,cache,chunks}
sudo chmod -R 755 /var/lib/loki
```

Servicio systemd:

```bash
sudo nano /etc/systemd/system/loki.service
sudo systemctl daemon-reload
sudo systemctl enable loki
sudo systemctl start loki
```

---

### Instalación y configuración de Grafana

```bash
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://packages.grafana.com/gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/grafana.gpg
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install grafana -y
```

Iniciar servicio:

```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

Acceso:

```
http://localhost:3000
```

---

### Instalación y configuración de Promtail

```bash
wget https://github.com/grafana/loki/releases/download/v3.0.0/promtail-linux-amd64.zip
sudo unzip promtail-linux-amd64.zip
chmod +x promtail-linux-amd64
sudo mv promtail-linux-amd64 promtail
```

Servicio:

```bash
sudo nano /etc/promtail/promtail-config.yaml
sudo nano /etc/systemd/system/promtail.service
sudo systemctl daemon-reload
sudo systemctl enable promtail
sudo systemctl start promtail
```

---

## Test 2 : Suricata - Filebeat - Elasticsearch - Kibana

### Repositorio Elastic

```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update
```

---

### Elasticsearch

Instalación:

```bash
sudo apt install -y elasticsearch
```

Configuración mínima:

```yaml
cluster.name: suricata-cluster
node.name: node-1
network.host: 0.0.0.0
discovery.type: single-node
xpack.security.enabled: false
```

---

### Kibana

Instalación:

```bash
sudo apt install -y kibana
```

Configuración:

```yaml
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://localhost:9200"]
```

Iniciar servicio:

```bash
sudo systemctl enable kibana
sudo systemctl start kibana
```

Acceso:

```
http://localhost:5601
```

---

### Filebeat con módulo Suricata

Instalación:

```bash
sudo apt install -y filebeat
sudo filebeat modules enable suricata
```

Configuración basica del módulo:

```yaml
- module: suricata
  eve:
    enabled: true
    var.paths:
      - /var/log/suricata/eve.json
```

Carga de dashboards y pipelines:

```bash
sudo filebeat setup --dashboards --pipelines --index-management
sudo systemctl enable filebeat
sudo systemctl start filebeat
```

---

## Suricata en modo independiente

```bash
sudo apt install suricata suricata-update -y
sudo suricata-update
sudo suricata -T -c /etc/suricata/suricata.yaml
sudo systemctl enable suricata
sudo systemctl start suricata
```

Logs:

```
/var/log/suricata/eve.json
```

---

## Comandos utiles para supervisión

```bash
sudo systemctl status suricata
sudo systemctl status loki
sudo systemctl status promtail
sudo systemctl status grafana-server
sudo systemctl status elasticsearch
sudo systemctl status kibana
sudo systemctl status filebeat
```

---
