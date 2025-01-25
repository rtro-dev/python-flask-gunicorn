# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "practica" do |p|
    p.vm.box = "debian/bullseye64"
    p.vm.hostname = "practica"
    p.vm.network "forwarded_port", guest: 8080, host: 8080
    p.vm.network "private_network", ip: "192.168.11.11"
    p.vm.provision "shell", name: "nginx", inline: <<-SHELL
      apt-get update
      apt-get install -y nginx git
      mkdir -p /var/www/nginx_server/html
      chown -R www-data:www-data /var/www/nginx_server/html
      chmod -R 755 /var/www/nginx_server
      systemctl restart nginx
    SHELL
  end # practica
end # cofig