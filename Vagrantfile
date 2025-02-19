Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "256"
  end

  # Configuración del Proxy Inverso
  config.vm.define "proxy" do |proxy|
    proxy.vm.hostname = "www.example.test"
    proxy.vm.network "private_network", ip: "192.168.57.10"
    proxy.vm.synced_folder "C:\\Users\\DAW-B\\Documents\\GitHub\\Proxy", "/vagrant", type: "virtualbox"
    proxy.vm.provision "shell", inline: <<-SHELL
      apt-get update && apt-get install -y nginx
      echo "192.168.57.11 w1" >> /etc/hosts
      cat > /etc/nginx/sites-available/default <<EOF
server {
    listen 80;
    server_name example.test www.example.test;

    location / {
        proxy_pass http://192.168.57.11:8080/;
        add_header X-friend enrique;
    }
}
EOF
      systemctl restart nginx
    SHELL
  end

  # Configuración del Servidor Web
  config.vm.define "web" do |web|
    web.vm.hostname = "w1.example.test"
    web.vm.network "private_network", ip: "192.168.57.11"
    web.vm.provision "shell", inline: <<-SHELL
      apt-get update && apt-get install -y nginx
      cat > /etc/nginx/sites-available/default <<EOF
server {
    listen 8080;
    server_name w1;
    root /var/www/html;
    index index.html index.htm;

    location / {
        add_header Host w1.example.test;
        try_files \$uri \$uri/ =404;
    }
}
EOF
      echo '<!DOCTYPE html>
<html>
<head><title>Servidor w1</title></head>
<body><h1>Bienvenido a example.test</h1><p>Servidor w1</p></body>
</html>' > /var/www/html/index.html
      systemctl restart nginx
    SHELL
  end
end
