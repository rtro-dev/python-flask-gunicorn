# Despliegue de una aplicación en Python con Flask en Gunicorn

<br>

## Contenidos
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