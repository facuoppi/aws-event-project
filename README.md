# Formulario de Gestión de Libros

Este proyecto consiste en un formulario web desarrollado en Flask que permite a los usuarios registrar información sobre libros, como título, autor, género, año de publicación y editorial. Los datos ingresados en el formulario se almacenan en una base de datos DynamoDB de AWS.

Además, se implementa un flujo de datos automatizado utilizando los servicios de AWS. Un stream de DynamoDB alimenta un pipeline que envía los datos a una cola SQS. Desde allí, un Lambda es invocado para realizar transformaciones en los datos y luego almacenarlos en un bucket de Amazon S3.

Finalmente, los datos almacenados en S3 pueden ser consultados utilizando Amazon Athena para análisis y generación de informes.

Todo el sistema está desplegado y ejecutándose en una instancia EC2 de AWS, lo que proporciona una plataforma robusta y escalable para la gestión eficiente de libros y análisis de datos.

## Arquitectura en AWS

![Oppido Facundo-AWS](https://github.com/facuoppi/aws-event-project/assets/94979941/5d8dd275-d1d7-40d8-93df-7de378e856d6)

## Desplegando una Aplicación Flask en EC2 con Gunicorn y Nginx

Te llevaré paso a paso en la configuración de una aplicación Flask en una instancia de EC2, utilizando Gunicorn como el servidor WSGI y Nginx como un proxy inverso.

Vamos a profundizar un poco más en cada paso:

### Paso 1: Instalar Python Virtualenv

```bash
sudo apt-get update
sudo apt-get install python3-venv
```

Este paso se encarga de asegurarse de que tu instancia EC2 tenga todas las herramientas necesarias para crear y gestionar entornos virtuales para Python.

### Paso 2: Configurar el Entorno Virtual

```bash
mkdir project
cd project
python3 -m venv venv
source venv/bin/activate
```

Acá creamos un directorio para el proyecto y configuramos un entorno virtual dentro de él. Activar el entorno virtual aisla las dependencias del proyecto, evitando conflictos con otros proyectos de Python en la misma máquina.

### Paso 3: Instalar Flask

```bash
pip install flask
```

Esto instala el framework Flask dentro del entorno virtual, permitiéndote desarrollar aplicaciones web usando Python.

### Paso 4: Instalar Flask-WTF

```bash
pip install Flask-WTF
```
Extensión para Flask que proporciona integración con el paquete WTForms, una biblioteca de Python para la creación de formularios web. Flask-WTF simplifica la creación y validación de formularios HTML en aplicaciones Flask.

### Paso 5: Instalar boto3

```bash
pip install boto3
```
Interfaz de cliente de Python para interactuar con servicios en la nube de Amazon Web Services (AWS).

### Paso 6: Crear una API Simple con Flask (Clonar Repositorio de Github)

```bash
git clone https://github.com/facuoppi/aws-event-project.git
cd ..
mv project/aws-event-project/* project/
rm -r project/aws-event-project
```

Clonas el código de tu aplicación Flask desde un repositorio de GitHub.

```bash
cd project/
python app.py
```

Verificamos que la aplicación funcione asegura que tu API de Flask esté correctamente configurada.

### Paso 7: Instalar Gunicorn

```bash
pip install gunicorn
```

Gunicorn, o Green Unicorn, es un servidor WSGI para ejecutar aplicaciones Flask. Instalarlo es un paso crucial para desplegar una aplicación Flask lista para producción.

```bash
gunicorn -b 0.0.0.0:8000 app:app
```

Ejecutas Gunicorn, uniéndolo a la dirección 0.0.0.0:8000 y especificando el punto de entrada de tu aplicación Flask (app:app).

### Paso 8: Usar systemd para Administrar Gunicorn

Creas un archivo de unidad systemd para administrar el proceso de Gunicorn como un servicio.

```bash
sudo nano /etc/systemd/system/project.service
```

El archivo de unidad especifica el usuario, el directorio de trabajo y el comando para iniciar Gunicorn como un servicio.

```ini
[Unit]
Description=Gunicorn instance for a de-project
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/project
ExecStart=/home/ubuntu/project/venv/bin/gunicorn -b 0.0.0.0:8000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

De esta forma permitimos el trafico a nuestra aplicación.
Después de crear el archivo de unidad, habilitas e inicias el servicio de Gunicorn.

```bash
sudo systemctl daemon-reload
sudo systemctl start project
sudo systemctl enable project
```

### Paso 9: Ejecutar el Servidor Web Nginx

```bash
sudo apt-get install nginx
```

Nginx es un servidor web que actuará como un proxy inverso para tu aplicación Flask, reenviando las solicitudes a Gunicorn.

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

Iniciar y habilitar Nginx asegura que se ejecute automáticamente después de un reinicio del sistema.

```bash
sudo nano /etc/nginx/sites-available/default
```

Configuras Nginx editando su archivo de configuración predeterminado, especificando el servidor upstream (Gunicorn) y la ubicación para reenviar las solicitudes.

```nginx
upstream flask_project {
    server 127.0.0.1:5000;
}

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    server_name _;

    location / {
        proxy_pass http://flask_project;
        try_files $uri $uri/ =404;
    }

    location /exito {
        alias /home/ubuntu/project/templates/;
        index exito.html;
        try_files $uri =404;
        allow all;
    }

    location /registrar_libro {
        proxy_pass http://flask_project;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Después de editar la configuración, reinicias Nginx para aplicar los cambios.

```bash
sudo systemctl restart nginx
```

Visitar la dirección IP pública de tu instancia EC2 en un navegador confirma que tu aplicación Flask ahora es accesible a través de Nginx, completando el proceso de implementación.

## Fin
Muchas gracias por visitar, espero te haya servido.
Puedes encontrarme en [LinkedIn](https://www.linkedin.com/facuoppi).

## Licencia

Este proyecto está licenciado bajo la [Licencia MIT](LICENSE). Consulta el archivo `LICENSE` para obtener más detalles.
