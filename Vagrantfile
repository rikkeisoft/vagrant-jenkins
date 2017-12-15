Vagrant.configure("2") do |config|
  config.vm.box = "puppetlabs/ubuntu-16.04-64-puppet"
  config.vm.box_version = "1.0.0"

  config.ssh.insert_key = false

  config.ssh.forward_agent = true

  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = true
  end

  config.vm.network "private_network", ip: "10.10.69.69"
  config.vm.network "forwarded_port", guest: 80, host: 6969
  config.vm.network "forwarded_port", guest: 9000, host: 9000

  config.vm.synced_folder ".", "/vagrant", mount_options: ["dmode=777,fmode=777"], type: "virtualbox"

  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = false

    # Customize the amount of memory on the VM:
    vb.cpus = 2
    vb.memory = "4096"
    vb.customize ["modifyvm", :id, "--ioapic", "on"]
    vb.customize ["modifyvm", :id, "--memory", 4096]
  end

  # https://github.com/Varying-Vagrant-Vagrants/VVV/issues/694
  $enable_serial_logging = false

  config.vm.provision "shell", inline: <<-EOC
curl --remote-name --location https://apt.puppetlabs.com/DEB-GPG-KEY-puppet
gpg --keyid-format 0xLONG --with-fingerprint ./DEB-GPG-KEY-puppet
sudo apt-key add DEB-GPG-KEY-puppet
rm -f DEB-GPG-KEY-puppet

sudo apt-get update && sudo apt-get install -y git apt-transport-https ca-certificates software-properties-common vim

# Make more swap space to prevent OOM killer
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Enable packet forwarding for IPv4
sudo sysctl -w net.ipv4.ip_forward=1
sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g' /etc/sysctl.conf

# Add Docker GPG Key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Add Jenkins GPG Key
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 7F438280EF8D349F

# Install DockerCE, Nginx, Jenkins
sudo apt-get update && sudo apt-get -y install docker-ce nginx jenkins

# Create docker group
sudo groupadd docker
sudo usermod -aG docker vagrant

# Install Docker Compose
curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

sudo systemctl enable docker
sudo service docker start

# Set storage driver to overlay2
cat <<-'EOF_DOCKER_DAEMON' > /etc/docker/daemon.json
{
  "storage-driver": "overlay2"
}
EOF_DOCKER_DAEMON

sudo service docker restart

sudo service nginx start

cat <<-'EOF_VHOST' > /etc/nginx/sites-available/default
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

sudo service nginx restart
sudo service jenkins restart
  EOC

end
