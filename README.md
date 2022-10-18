# Reverse proxy para el Jardín Botánico De Ushuaia

## Tecnologías

**Proyecto Nginx**

### Librerias y otros

- Nginx

## Dependencias

- Docker (Apéndice)

## Configuración

Los puertos 80 y 443 deben estar libres.

Definir las redirecciones en el archivo nginx.conf utilizando la siguiente plantilla.

Nota: _host.docker.internal_ es un alias al host que está corriendo el contenedor del reverse proxy.

```
upstream sitio_ejemplo_uno {
    server host.docker.internal:8000;
}
upstream sitio_ejemplo_dos {
    server 192.168.150.50:8000;
}

# ENTRANDO POR IP
server {
    listen 80 default_server;
    root /var/www/html;
    index index.html;
    client_max_body_size 500M;
    error_log /var/log/nginx/service-error.log debug;
    server_name _;
    location / {
	include proxy_params;

	# Direccion de proxy
        proxy_pass http://sitio_ejemplo_uno;
        proxy_ssl_server_name on;
    }
}

# ENTRANDO CON UN DOMINIO
server {
    listen 80;
    root /var/www/html;
    index index.html;
    error_log /var/log/nginx/service-error.log debug;
    server_name nombre_dominio_filtrado;

    # Direccion de proxy
    location / {
	include proxy_params;
        proxy_pass http://sitio_ejemplo_dos;
        proxy_ssl_server_name on;
    }
}
```

```bash
$ # Configurar el nombre del contenedor
$ cp .env.example .env
$ nano .env
```

## Iniciar

```bash
$ docker compose up -d prod
```

_Quitando la opción *-d* se ven los logs del contenedor._

## Detener

```bash
$ # Si estan corriendo con logs visibles
$ #     detener con Ctr+C
$ docker compose down
```

## Instalar certificados SSL

```bash
$ docker compose exec -it prod bash
$ certbot --nginx
```

# Apéndice

## Instalación de docker en ubuntu 18.04/20.04/22.04

Fuente: [Instalación docker ubuntu](https://docs.docker.com/engine/install/ubuntu).

```bash
$ sudo apt-get update
$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

$  echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
