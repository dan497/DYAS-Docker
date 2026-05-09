# Docker Workshop: Flask, Redis y Docker Hub

Guía de desarrollo del taller de Docker. Cubre instalación, construcción de imágenes, ejecución de contenedores, orquestación con Redis mediante Docker Compose y publicación en Docker Hub.

---

## Paso 0: Configuración inicial

### Lo que necesitas:

* Docker Desktop instalado en tu máquina.
* Descárgalo desde: [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
* Selecciona la versión adecuada para tu sistema operativo.
* Una vez instalado, ábrelo y espera a que muestre: Docker is running.

### Confirmar instalación:

```bash
docker --version
```

La salida debe mostrar algo similar a: Docker version 24.0.5

---

## Paso 1: Primera imagen Docker (Hello App)

### 1.1 Preparar directorio y archivo:

```bash
mkdir -p ~/Sites/hello-world
cd ~/Sites/hello-world
echo "hello" > hello
```

### 1.2 Escribir el Dockerfile:

Archivo `Dockerfile`:

```dockerfile
FROM busybox
COPY /hello /
RUN cat /hello
```

### 1.3 Construir la imagen:

```bash
docker build -t helloapp:v1 .
```

### 1.4 Listar imágenes disponibles:

```bash
docker images
```

---

## Paso 2: Aplicación Flask con Redis

### 2.1 Crear el directorio del proyecto:

```bash
mkdir -p ~/Sites/friendlyhello
cd ~/Sites/friendlyhello
```

### 2.2 Archivo `app.py`:

```python
from flask import Flask
from redis import Redis, RedisError
import os, socket

redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)
app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

### 2.3 Archivo `requirements.txt`:

```text
Flask
Redis
```

### 2.4 Dockerfile para Flask:

```dockerfile
FROM python:3-slim
WORKDIR /app
COPY . /app
RUN pip install --trusted-host pypi.python.org -r requirements.txt
EXPOSE 80
ENV NAME World
CMD ["python", "app.py"]
```

### 2.5 Construir la imagen:

```bash
docker build -t friendlyhello .
```

### 2.6 Correr la app sin Redis:

```bash
docker run --rm -p 4000:80 friendlyhello
```

Abre en el navegador: [http://localhost:4000](http://localhost:4000)

---

## Paso 3: Redis con Docker Compose

### 3.1 Crear `docker-compose.yaml`:

```yaml
version: '3'

services:
  web:
    build: .
    ports:
      - "4000:80"
    depends_on:
      - redis

  redis:
    image: redis
    command: redis-server --appendonly yes
    volumes:
      - "./data:/data"
```

### 3.2 Levantar los servicios:

```bash
docker compose up
```

### 3.3 Comprobar en el navegador:

[http://localhost:4000](http://localhost:4000)

El contador de visitas debe incrementarse cada vez que se recarga la página.

### 3.4 Bajar los servicios:

```bash
docker compose down
```

---

## Paso 4: Publicar en Docker Hub

### 4.1 Registrarse en [https://hub.docker.com/](https://hub.docker.com/)

### 4.2 Autenticarse desde la terminal:

```bash
docker login
```

### 4.3 Crear el tag de la imagen:

```bash
docker tag friendlyhello danielriveros/friendlyhello
```

### 4.4 Subir la imagen al registro:

```bash
docker push danielriveros/friendlyhello
```

---

## Paso 5: Usar la imagen publicada

### Ejecutar desde Docker Hub con Compose:

```yaml
version: '3'

services:
  web:
    image: danielriveros/friendlyhello
    ports:
      - "4000:80"
    depends_on:
      - redis

  redis:
    image: redis
    command: redis-server --appendonly yes
    volumes:
      - "./data:/data"
```

```bash
docker compose up
```

Visita [http://localhost:4000](http://localhost:4000) y verifica que el contador funciona correctamente.

---

## Ejercicios realizados

### Ejercicio 1: Correr imagen propia desde Docker Hub

* Se actualizó `docker-compose.yaml` para usar `danielriveros/friendlyhello`
* `docker run --rm -p 4000:80 danielriveros/friendlyhello`
* Funcionó correctamente

### Ejercicio 2: Correr imagen de un compañero

* Se cambió la imagen a `medids0526/friendlyhello` en `docker-compose.yaml`
* `docker run --rm -p 4000:80 medids0526/friendlyhello`

---

## Autor

**Daniel Riveros**
Arquitectura de Software – DYAS – 2025
