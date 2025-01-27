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
sudo chown -R vagrant:www-data /var/www/pythonApp
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