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
    p.vm.provision "shell", name: "python-pip", inline: <<-SHELL
      apt-get update && sudo apt-get install -y python3-pip
    SHELL
    p.vm.provision "shell", name: "app1", privileged: false, inline: <<-SHELL
      pip3 install pipenv
      PATH=$PATH:/home/$USER/.local/bin
      pip3 install python-dotenv
      sudo mkdir -p /var/www/pythonApp
      sudo chown -R $USER:www-data /var/www/pythonApp
      sudo chmod -R 775 /var/www/pythonApp
      sudo cp -v /vagrant/.env /var/www/pythonApp/
      cd /var/www/pythonApp
      pipenv install flask gunicorn
      cp -v /vagrant/app1/application.py /var/www/pythonApp/
      cp -v /vagrant/app1/wsgi.py /var/www/pythonApp/
      sudo systemctl start nginx      
      sudo cp -v /vagrant/app1/flask_app.service /etc/systemd/system/
      sudo systemctl daemon-reload
      sudo systemctl enable flask_app
      sudo systemctl start flask_app
      sudo cp -v /vagrant/app1/app.conf /etc/nginx/sites-available/
      sudo ln -s /etc/nginx/sites-available/app.conf /etc/nginx/sites-enabled/
      sudo systemctl restart nginx
    SHELL
    p.vm.provision "shell", name: "app2", privileged: false, inline: <<-SHELL
      cd /var/www
      sudo git clone https://github.com/Azure-Samples/msdocs-python-flask-webapp-quickstart
      sudo chown -R $USER:www-data /var/www/msdocs-python-flask-webapp-quickstart
      sudo chmod -R 775 /var/www/msdocs-python-flask-webapp-quickstart
      sudo cp -v /vagrant/.env /var/www/msdocs-python-flask-webapp-quickstart/
      cd /var/www/msdocs-python-flask-webapp-quickstart
      pipenv install -r requirements.txt
      cp -v /vagrant/app2/wsgi.py /var/www/msdocs-python-flask-webapp-quickstart/
      sudo systemctl start nginx
      sudo cp -v /vagrant/app2/flask_app2.service /etc/systemd/system/
      sudo systemctl daemon-reload
      sudo systemctl enable flask_app2
      sudo systemctl start flask_app2
      sudo cp -v /vagrant/app2/app2.conf /etc/nginx/sites-available/
      sudo ln -s /etc/nginx/sites-available/app2.conf /etc/nginx/sites-enabled/
      sudo systemctl restart nginx
    SHELL
  end # practica
end # cofig