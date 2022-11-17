# Monta tu propio servidor Git con Gitea y Docker

Este documento está basado en este artículo: [Monta tu propio servidor de Git con Gitea y Docker](https://www.crashell.com/estudio/monta_tu_propio_servidor_de_git_con_gitea_y_docker/)

[![Monta tu propio servidor de Git con Gitea y Docker](https://img.youtube.com/vi/bTaRIzhgDIk/maxresdefault.jpg)](https://youtu.be/bTaRIzhgDIk "Monta tu propio servidor de Git con Gitea y Docker")

## ¿Qué es Gitea?

Es un servidor de Git autohospedado, rápido y fácil de usar. Similar a la interfaz gráfica de GitHub y con muchas funcionalidades que comparten con BitBucket y GitLab. Este se encuentra escrito en Go. Es una aplicación ligera y se puede instalar en sistemas de baja potencia, así que, si estás buscando una alternativa a alguno de los sistemas de gestión de Git en la nube antes mencionados, Gitea es una gran opción.

El objetivo de este proyecto es proporcionar la forma más fácil de establecer un servicio de Git autohospedado (que puedes instalar y gestionar desde tu propia infraestructura). 

## Instalación de Gitea por medio de contenedores Docker

La instalación se lleva a cabo por medio de Docker Compose. Igualmente, en la [documentación de Gitea](https://docs.gitea.io/en-us/install-with-docker/), puede encontrar otras formas y alternativas al gestor de base de datos.

```yaml
version: "3"

networks:
  gitea:
    external: false

services:
  server:
    image: gitea/gitea:1.17.3
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - GITEA__database__DB_TYPE=mysql
      - GITEA__database__HOST=db:3306
      - GITEA__database__NAME=gitea
      - GITEA__database__USER=gitea
      - GITEA__database__PASSWD=gitea
    restart: always
    networks:
      - gitea
    volumes:
      - ./gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3000:3000"
      - "222:22"
    depends_on:
      - db

  db:
    image: mysql:8
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=gitea
      - MYSQL_USER=gitea
      - MYSQL_PASSWORD=gitea
      - MYSQL_DATABASE=gitea
    networks:
      - gitea
    volumes:
      - ./mysql:/var/lib/mysql
```

Ejecutas el Docker Compose mediante la siguiente instrucción: `docker-compose up -d`. Donde instalará los paquetes en los contenedores necesarios.

```bash
$ docker-compose up -d
[+] Running 21/21
 - server Pulled                                                                                                                                    45.0s 
   - 213ec9aee27d Pull complete                                                                                                                      1.4s 
   - 3b125bd762f1 Pull complete                                                                                                                     10.5s 
 - db Pulling                                                                                                                                       
   - 6e6c408eba18 Pull complete                                                                                                                     10.5s 
   - 3b125bd762f1 Pull complete                                                                                                                     10.5s
[+] Running 3/3
 - Network gitea-docker-compose_gitea   Created                                                                                                      0.0s
 - Container gitea-docker-compose-db-1  Started                                                                                                      0.8s
 - Container gitea                      Started                                                                                                      0.9s
```

Se levantan dos contenedores, la cual uno es el servidor de Gitea y el otro es el servidor donde se encuentra el sistema gestor de base de datos MySQL; corriendo en los puertos correspondientes.

```bash
$ docker-compose ps
NAME                        COMMAND                  SERVICE             STATUS              PORTS
gitea                       "/usr/bin/entrypoint…"   server              running             0.0.0.0:3000->3000/tcp, 0.0.0.0:222->22/tcp
gitea-docker-compose-db-1   "docker-entrypoint.s…"   db                  running             3306/tcp, 33060/tcp
```

## Primeros pasos con Gitea

Para llegar a la interfaz gráfica web de Gitea, es tan sencillo como abrir en su navegador la ruta: [https://127.0.0.1:3000](https://127.0.0.1:3000). Este le llevará a la configuración inicial, donde señala el tipo de base de datos, servidor, nombre de usuario, contraseña y demás credenciales. 

![imagen](https://user-images.githubusercontent.com/7296281/202072137-3447c3fa-e024-475b-9fa3-cfcba2e1e5b0.png)

Aplicar una configuración inicial es muy importante, en este caso, establecer credenciales de acceso, como una cuenta personal. Los datos escritos en las _screenshots_, son de ejemplo.

![imagen](https://user-images.githubusercontent.com/7296281/202072363-066525e3-d82a-46ac-a8c5-0cf31547f518.png)

Después de registrarse, tanto que esto hubiese sido desde la configuración inicial, como desde el panel de registro en el inicio de sesión, lo que faltaría es acceder a tu cuenta para contemplar las bondades que te trae Gitea.

![imagen](https://user-images.githubusercontent.com/7296281/202072737-c99242ed-1702-4d15-a05b-567adf7c1ff5.png)


## Migrar un repositorio de GitHub a Gitea

Es posible migrar repostorios que estén en diferentes espacios. En este ejemplo estoy usando este mismo repositorio.

![imagen](https://user-images.githubusercontent.com/7296281/202074185-6ed155c0-4cfa-4d54-9752-008beeeb9674.png)

Esta es la forma en que se observa un repositorio en Gitea; y es fácil de ver que se parece mucho a la interfaz de GitHub.

![imagen](https://user-images.githubusercontent.com/7296281/202074429-e53b5e0e-65ec-4e0a-a338-4b0345652a4f.png)

Con esto ya estás listo para trabajar en un entorno local, para instalar tu propio servidor de Git con interfaz gráfica; trabajar en equipo y poder llevar este proyecto a la nube, a tu propia infraestructura. 

```bash
$ docker-compose down --volumes
[+] Running 3/3
 - Container gitea                      Removed                                                                                               1.4s
 - Container gitea-docker-compose-db-1  Removed                                                                                               1.5s
 - Network gitea-docker-compose_gitea   Removed
```

Esta última instrucción termina con los contenedores que habían sido levantados.