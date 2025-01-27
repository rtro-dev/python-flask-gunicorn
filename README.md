# Despliegue de una aplicación en Python con Flask en Gunicorn

<br>

## Contenidos
- [Nginx](#nginx)  
- [Python *pip*](#python-pip)  
- [](#)  

<br>

## Nginx

Se realiza la instalación de Nginx en la provisión de la sigueinte forma:
```Vagrantfile
p.vm.provision "shell", name: "nginx", inline: <<-SHELL
    apt-get update
    apt-get install -y nginx git
    mkdir -p /var/www/nginx_server/html
    chown -R www-data:www-data /var/www/nginx_server/html
    chmod -R 755 /var/www/nginx_server
    systemctl restart nginx
SHELL
```

## Python *pip*

Actualización e instalación del gestor de paquetes:  
`sudo apt-get update && sudo apt-get install -y python3-pip`

Instalación de *pipenv*:  
`pip3 install pipenv`

<img src="./htdocs/1.png">

Comprobar que se ha instalado correctamente mostrando su versión:  
```bash
PATH=$PATH:/home/$USER/.local/bin
pipenv --version
```

<img src="./htdocs/2.png">

Instalación de *python-dotenv* para cargar las variables de entorno:  
`pip3 install python-dotenv`

Se crea el directorio en el que se guardará el proyecto:  
`sudo mkdir -p /var/www/pythonApp`

Se cambian el grupo y los permisos del directorio:
```bash
sudo chown -R $USER:www-data /var/www/pythonApp
sudo chmod -R 775 /var/www/pythonApp
```

Creación del archivo oculto *.env* que almacenará las variables de entorno:  
`nano /var/www/pythonApp/.env`  
Y se le añaden las variables para indicar el archivo *.pi* y el entorno:
```bash
FLASK_APP=wsgi.py
FLASK_ENV=production
```

<img src="./htdocs/3.png">

## Entorno virtual

Es necesario situarse en el directorio del proyecto:  
`cd /var/www/pythonApp`

Inicializar el entorno:  
`pipenv shell`

<img src="./htdocs/4.png">

Aparecerá el nombre del entorno al inicio del prompt del shell:  
`(pythonApp) vagrant@practica:/var/www/pythonApp$`

Instalación de dependencias necesarias:  
`pipenv install flask gunicorn`

<img src="./htdocs/5.png">

Creación de aplicación Flask a modo de prueba:  
`touch application.py wsgi.py`  
Se edita el contenido:  
`nano /var/www/pythonApp/application.py`
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    '''Index page route'''
    return '<h1>App desplegada</h1>'
```

`nano /var/www/pythonApp/wsgi.py`
```python
from application import app

if __name__ == '__main__':
   app.run(debug=False)
```

Ejecución de la aplicación en la dirección *0.0.0.0* para que escuche en todas sus intefaces:  
`flask run --host '0.0.0.0'`

<img src="./htdocs/6.png">

Acceso a la aplicación desde el navegador: *http://192.168.11.11:5000*

<img src="./htdocs/7.png">

Comprobación con Gunicorn tras parar el servidor con *Ctrl+C*:  
`gunicorn --workers 4 --bind 0.0.0.0:5000 wsgi:app`  
- **--workers 4** establece el número de hilos.  
- **--bind 0.0.0.0:5000** establece la escucha por todas sus interfaces.  
- **wsgi:app** donde el primer término es el nombre del archivo *wsgi.py* y el segundo es la instancia de la aplicación Flask dentro del archivo.

<img src="./htdocs/8.png">

Es necesario averiguar la ruta desde la que se ejecuta *Gunicorn* en el entorno virtual, será necesaria más adelante para el servicio de *systemd* para arrancar la aplicación. Para averiguar la ruta:  
`which gunicorn`  
Respuesta:  
`/home/vagrant/.local/share/virtualenvs/pythonApp--CG28rcg/bin/gunicorn`

<img src="./htdocs/9.png">

Salir del entorno virtual:  
`deactivate`

## Gunicorn

Primeramente, iniciar y comprobar Nginx:
```bash
sudo systemctl start nginx
sudo systemctl status nginx
```

Fuera del entorno virtual, se debe crear un archivo *systemd* para que Gunicorn se ejecute como un servicio más del sistema.

Creación del fichero:  
`sudo nano /etc/systemd/system/flask_app.service`  
Contenido:  
```bash
[Unit]
Description=flask app service - App con flask y Gunicorn
After=network.target
[Service]
User=vagrant
Group=www-data
Environment="PATH=/home/vagrant/.local/share/virtualenvs/pythonApp--CG28rcg/bin"
WorkingDirectory=/var/www/pythonApp
ExecStart=/home/vagrant/.local/share/virtualenvs/pythonApp--CG28rcg/bin/gunicorn --workers 3 --bind unix:/var/www/pythonApp/pythonApp.sock wsgi:app

[Install]
WantedBy=multi-user.target
```
- **User** establece el usuario con permisos sobre el directorio.  
- **Group** establece el usuario con permisos sobre el directorio.  
- **Environment** establece el directorio *bin* dentro del entorno virtual.  
- **WorkingDirectory** establece el directorio del proyecto
- **ExecStart** establece el path donde se encuentra el ejecutable de Gunicorn dentro del entorno virtual, junto a las opciones y comandos con los que se iniciará.

Se recargan los archivos de configuración de *systemd*:  
`sudo systemctl daemon-reload`

Se habilita y se inicia el servicio:
```bash
sudo systemctl enable flask_app
sudo systemctl start flask_app
```

<img src="./htdocs/10.png">

### Configuración de Nginx

Creación del archivo de cofiguración del sitio web:  
`sudo nano /etc/nginx/sites-available/app.conf`  
Contenido:
```bash
server {
  listen 80;
  server_name pythonapp.izv www.pythonapp.izv;

  access_log /var/log/nginx/app.access.log;
  error_log /var/log/nginx/app.error.log;

  location / {
    include proxy_params;
    proxy_pass http://unix:/var/www/pythonApp/pythonApp.sock;
  }
}
```

Creación del link simbólico:  
`sudo ln -s /etc/nginx/sites-available/app.conf /etc/nginx/sites-enabled/`  
Comprobación:  
`ls -l /etc/nginx/sites-enabled/ | grep app.conf`

<img src="./htdocs/11.png">

Comprobación de que la configuración de Nginx no contiene errores:  
`sudo nginx -t`

<img src="./htdocs/12.png">

Reinicio y comprobación del servicio:
```bash
sudo systemctl restart nginx
sudo systemctl status nginx
```

Se edita el archivo /etc/hosts para que asocie la IP de la máquina virtual al servidor.  
En Windows está en el siguiente directorio:  
`C:\Windows\System32\drivers\etc\hosts`  
Se añade la asociación:  
`192.168.11.11 pythonapp.izv www.pythonapp.izv`

Se comprueba que se ha desplegado correctamente accediendo a la dirección:  
- *http://pythonapp.izv/*  
- *http://www.pythonapp.izv/*

<img src="./htdocs/13.png">