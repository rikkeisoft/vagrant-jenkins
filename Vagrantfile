Vagrant.configure("2") do |config|
  config.vm.box = "rikkeisoft_jenkins_ubuntu"
  config.vm.box_url = "https://atlas.hashicorp.com/puppetlabs/boxes/ubuntu-16.04-64-puppet/versions/1.0.0/providers/virtualbox.box"

  config.ssh.insert_key = false

  config.ssh.forward_agent = true

  config.vm.network "private_network", ip: "10.10.69.69"
  config.vm.network "forwarded_port", guest: 80, host: 6969
  
  config.vm.synced_folder ".", "/vagrant", mount_options: ["dmode=777,fmode=777"], type: "virtualbox"

  config.vm.provision "shell", inline: <<-EOC
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list'
sudo apt-get update > /dev/null 2>&1
sudo apt-get -y install jenkins > /dev/null 2>&1
sudo apt-get -y install nginx > /dev/null 2>&1

cat <<-'EOF_VHOST' > /vagrant/default /etc/nginx/sites-available/default
server {
    listen 80;
    listen [::]:80 default ipv6only=on;
    server_name jenkins.dev;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header X-NginX-Proxy true;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
    }
}
EOF_VHOST

sudo service nginx start
sudo service jenkins start
EOC

end
