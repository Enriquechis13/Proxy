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

      ## Descomentar para realizar la tarea de ampliacion
      
      # apt-get update

      # apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release

      # curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

      # echo "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

      # apt-get update
      # apt-get install -y docker-ce docker-ce-cli containerd.io

      # systemctl enable docker
      # systemctl start docker

      # curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
      # chmod +x /usr/local/bin/docker-compose

      # docker-compose --version

      # echo "version: '3.7'
      # services:
      #   w1:
      #     image: nginx:latest
      #     container_name: w1
      #     ports:
      #       - '8080:80'
      #     volumes:
      #       - ./html:/usr/share/nginx/html:ro
      #   proxy:
      #     image: nginx:latest
      #     container_name: proxy
      #     ports:
      #       - '80:80'
      #     volumes:
      #       - ./nginx.conf:/etc/nginx/nginx.conf:ro
      #     depends_on:
      #       - w1" > /vagrant/docker-compose.yml

      # mkdir -p /vagrant/html
      # echo "<!DOCTYPE html>
      # <html lang='en'>
      # <head>
      #     <meta charset='UTF-8'>
      #     <meta name='viewport' content='width=device-width, initial-scale=1.0'>
      #     <title>example.test</title>
      # </head>
      # <body>
      #     <h1>example.test</h1>
      #     <h2>Bienvenido</h2>
      #     <p>Servidor w1</p>
      # </body>
      # </html>" > /vagrant/html/index.html

      # echo "server {
      #     listen 80;
      #     server_name example.test www.example.test;

      #     location / {
      #         proxy_pass http://w1:8080;
      #     }
      # }" > /vagrant/nginx.conf

      # cd /vagrant
      # docker-compose up -d
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
