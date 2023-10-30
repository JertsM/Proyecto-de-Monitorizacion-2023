# Proyecto-de-Monitorizacion-2023
Este proyecto de monitorización sienta las bases para el funcionamiento correcto de los gráficos levantados dentro de un docker-compose.

## Instalación de Docker

Para empezar, se instalará docker y docker-compose, dejaré un listado con los comandos necesarios para instalar todo correctamente:

```bash
sudo apt update
```

Instalamos unos paquetes que funcionan de base para emplear paquetes a través de HTTPS:
```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

A continuación, se añade la clave GPG para el repo de Docker en mi sistema:
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Agregamos el repositorio de Docker a las fuentes de APT:
```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```

Actualizamos el paquete de base de datos con los paquetes de Docker actualizados:
```bash
sudo apt update
```

Nos aseguramos de instalarlo desde el repositorio de Docker y no desde el repo de Ubuntu:
```bash
apt-cache policy docker-ce
```

Como último paso, instalamos Docker:
```bash
sudo apt install docker-ce
```

### Instalación de Docker-Compose

Una vez instalado docker, para poder levantar contenedores a partir de un 'docker-compose' es necesario instalarlo con un comando de forma independiente:

```bash
sudo apt install docker-compose
```

## Creación de archivos de configuración

Para empezar, crearemos el archivo 'docker-compose.yml', en el directorio que nosotros queramos (en mi caso en la carpeta raíz de mi usuario de Ubuntu). En él escribiremos las imágenes que tiene que descargar, los puertos que empleará y demás parámetros que añadiremos según nos convengan.

Para este proyecto trabajaré con 'Prometheus', 'Grafana', 'node_exporter' y 'Cadvisor':

### Archivo docker-compose.yml
```bash
version: '3'

services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus:/etc/prometheus
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    depends_on:
      - prometheus

  node_exporter:
    image: prom/node-exporter
    volumes:
      - /proc:/host/proc:io
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--path.rootfs=/rootfs'
    ports:
      - "9100:9100"

  cadvisor:
    image: google/cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker:/var/lib/docker:ro
```

### Archivo prometheus.yml

A continuación, debemos crear un archivo de configuración de Prometheus al que llamaremos 'prometheus.yml', en el que declararemos que capture las métricas de 'Cadvisor' y 'node_exporter':

```bash
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node_exporter:9100']
```

## Acceso y configuración de Grafana

Una vez creados los archivos de configuración, desde la terminal nos situamos en la carpeta donde esté 'docker-compose.yml' y levantamos los servicios del contenedor:

```bash
docker-compose up -d
```

Una vez que los contenedores están en funcionamiento, accedemos a Grafana desde el navegador empleando la dirección *http://localhost:3000*. Las credenciales son "admin" para el usuario y la contraseña.

Una vez dentro de Grafana, nos dijirimos a la configuración para añadir a Prometheus como fuente de datos. Además, debemos proporcionar la URL *http://prometheus:9090* como endpoint de acceso.

Por último, solo queda crear los dashboards, los cuáles se ajustarán a nuestras necesidades. También se pueden importar dashboards predefinidos de páginas como *https://grafana.com/*.
