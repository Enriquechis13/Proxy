# Proxy Inverso con Nginx y Vagrant

Este proyecto muestra cómo configurar un proxy inverso con Nginx utilizando dos máquinas virtuales Debian gestionadas por Vagrant. El proxy redirige las peticiones a un servidor web backend.

## Arquitectura
La configuración incluye:
1. **Servidor Proxy**: `proxy` (Nginx), dirección `192.168.57.10`.
2. **Servidor Web**: `w1` (Nginx), dirección `192.168.57.11`.

Las peticiones a `http://www.example.test` llegan al proxy, que las redirige al servidor web `w1` en el puerto 8080.

### Diagrama de Red (PlantUML)
```plantuml
@startuml
!define osaPuml https://raw.githubusercontent.com/Crashedmind/PlantUML-opensecurityarchitecture2-icons/master
!include osaPuml/Common.puml
!include osaPuml/User/all.puml
!include osaPuml/Server/all.puml

Usuario: <$osa_user_green_operations> Juan
www: <$osa_server_proxy> Proxy
w1: <$osa_server_web> Web

Usuario -> www
www -> w1
w1 -> www
www -> Usuario
@enduml
```

## Requisitos
- Vagrant
- VirtualBox
- Sistema anfitrión con conexión a red privada

## Despliegue con Vagrant
El `Vagrantfile` configura las dos máquinas virtuales:

## Explicación Detallada

1. **Configuración del Servidor Web (w1)**

- IP: `192.168.57.11`
- Puerto: `8080`
- Página de prueba en `/var/www/html/index.html`.
- Configuración Nginx:
```nginx
server {
    listen 8080;
    server_name w1;
    root /var/www/html;
    index index.html index.htm;

    location / {
        add_header Host w1.example.test;
        try_files $uri $uri/ =404;
    }
}
```

2. **Configuración del Proxy Inverso**

- IP: `192.168.57.10`
- Puerto: `80`
- Reenvío al servidor `w1` en `192.168.57.11:8080`.
```nginx
server {
    listen 80;
    server_name example.test www.example.test;

    location / {
        proxy_pass http://192.168.57.11:8080/;
        add_header X-friend enrique;
    }
}
```

3. **Flujo de Tráfico**

1. El usuario accede a `http://www.example.test`.
2. El proxy (Nginx) en `192.168.57.10` recibe la petición.
3. El proxy redirige la petición a `http://192.168.57.11:8080/` (Servidor w1).
4. `w1` responde con la página HTML.
5. El proxy devuelve la página al navegador del usuario.

## Prueba de Conectividad
1. **Levantar las máquinas:**

```bash
vagrant up
```

2. **Agregar en el anfitrión (`/etc/hosts` o `C:\Windows\System32\drivers\etc\hosts`):**

```
192.168.57.10 www.example.test
```

3. **Acceder desde el navegador:**

```
http://www.example.test
```

![Imagen proxy](img/Captura.PNG)

4. **Verificar con `curl` desde el proxy:**

```bash
curl http://www.example.test
```

![Imagen proxy](img/Captura1.5.PNG)

## Comprobaciones y Logs

1. **Acceso y Verificación de Logs**

- Acceder a `http://192.168.57.10` desde el navegador.

- Ver logs en la máquina `proxy`:

```bash
sudo tail /var/log/nginx/access.log
```

![Imagen proxy](img/Captura2.PNG)

- Ver logs en la máquina `web`:

```bash
sudo tail /var/log/nginx/access.log
```

![Imagen proxy](img/Captura3.PNG)

2. **Cabeceras HTTP**

- En Firefox, pulsar `Ctrl+Shift+I` (Herramientas de desarrollo).
- Ir a la pestaña `Network` y marcar `Disable cache`.
- Recargar la página y examinar la petición GET.

![Imagen proxy](img/Captura4.PNG)

Si accedemos mediante un nombre de host `www.192.168.57.10.nip.io`, veremos que la cabecera Host cambia.

![Imagen proxy](img/Captura5.PNG)

### 3. **Comprobación de Cabeceras X-friend y Host**
- El proxy añade la cabecera `X-friend`.
- El servidor web `w1` añade la cabecera `Host`.

Ejemplo de configuración en `/etc/nginx/sites-available/default`:

**En Proxy:**

```nginx
location / {
    proxy_pass http://192.168.57.11:8080;
    add_header X-friend enrique;
}
```
![Imagen proxy](img/Captura6.PNG)

**En Servidor Web (w1):**

```nginx
location / {
    add_header Host w1.example.test;
    try_files $uri $uri/ =404;
}
```

![Imagen proxy](img/Captura7.PNG)

- Reiniciar Nginx:
```bash
sudo systemctl restart nginx
```

- La verificacion del navegador (pestaña `Network`), son las dos capturas anteriores.


## Ampliación con Docker y Docker Compose

En esta ampliación, configuraremos el entorno utilizando contenedores Docker. Crearemos un contenedor para el servidor web (`w1`), otro para el proxy inverso (`proxy`) y un archivo `docker-compose.yml` para la gestión.

**1. Instalación de Docker y Docker Compose**

Ejecutar los siguientes comandos en ambas máquinas virtuales:

```bash
apt-get update
apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
echo "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update
apt-get install -y docker-ce docker-ce-cli containerd.io
```

![Imagen proxy](img/Captura7.PNG)

Habilitar y arrancar Docker:

```bash
systemctl enable docker
systemctl start docker
```

Instalar Docker Compose:

```bash
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

Verificar la instalación:

```bash
docker-compose --version
```

**2. Configuración de Docker Compose**

Crear el archivo `/vagrant/docker-compose.yml` con la siguiente configuración:

```yaml
version: "3.8"

services:
  web:
    image: nginx:latest
    container_name: w1
    volumes:
      - ./web/html:/usr/share/nginx/html
      - ./web/nginx/default.conf:/etc/nginx/conf.d/default.conf
    networks:
      - mynetwork

  proxy:
    image: nginx:latest
    container_name: proxy
    ports:
      - "80:80"
    volumes:
      - ./proxy/nginx/default.conf:/etc/nginx/conf.d/default.conf
    networks:
      - mynetwork

networks:
  mynetwork:
    driver: bridge
```

**3. Configuración del contenido web y Nginx**

Crear la carpeta `/vagrant/html` y un archivo `index.html` dentro:

```bash
mkdir -p /vagrant/html
cat > /vagrant/html/index.html <<EOF
<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='UTF-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1.0'>
    <title>example.test</title>
</head>
<body>
    <h1>example.test</h1>
    <h2>Bienvenido</h2>
    <p>Servidor w1</p>
</body>
</html>
EOF
```

Crear la configuración de Nginx para el proxy inverso en `/vagrant/nginx.conf`:

```nginx
server {
    listen 80;
    server_name example.test www.example.test;

    location / {
        proxy_pass http://w1:80;
        add_header X-friend enrique;
    }
}
```

![Imagen proxy](img/CapturaAMPLIACION1.PNG)

**4. Despliegue de Contenedores**

Ir al directorio `/vagrant` y ejecutar:

```bash
docker-compose up -d
```

Verificar los contenedores en ejecución:

```bash
docker ps
```

![Imagen proxy](img/CapturaAMPLIACION3.PNG)

**5. Prueba y Verificación**

Acceder a `http://www.example.test`

![Imagen proxy](img/CapturaAMPLIACION2.PNG)

### **6. Detener y Eliminar Contenedores**

Para detener los contenedores:

```bash
docker-compose down
```

Con esto, hemos replicado la práctica utilizando contenedores, simplificando la gestión y asegurando un entorno reproducible.



