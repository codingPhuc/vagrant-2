Vagrant.configure("2") do |config|
  config.vm.define "web01" do |web01|
    web01.vm.box = "ubuntu/bionic64"  # Use the Ubuntu Bionic 64-bit box
    web01.vm.hostname = "vagrant"  # Set the hostname of the VM
    # web01.vm.network "private_network", ip: "192.168.56.18"
    web01.vm.network "forwarded_port", guest: 5678, host: 8080  # Forward port 80 on the guest to port 8080 on the host
    web01.vm.provider "vmware_desktop" do |vmware|
      vmware.gui = false  # Run the VM without a GUI
      # vmware.allowlist_verified = true
      vmware.memory = "4096"
      vmware.cpus = "2"
    end
    

    # Provision the VM with a shell script
    web01.vm.provision "shell", inline: <<-SHELL
      sudo apt update -y
      sudo apt upgrade -y
      # Install Docker
      sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
      sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
      sudo apt-get update -y
      sudo apt-get install -y docker-ce

      # Add user 'vagrant' to the Docker group
      sudo usermod -aG docker vagrant

      # Install and start Nginx
      sudo apt install nginx -y
 
    SHELL

    # Configure Nginx to proxy requests to a Docker container

    
    web01.vm.provision "shell", name: "config", inline: <<-'SHELL'
      cat <<EOL > /etc/nginx/sites-available/default
      server {
        listen 81;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade \$http_upgrade;
          proxy_set_header Connection "upgrade";
          proxy_set_header Host \$host;
          proxy_cache_bypass \$http_upgrade;
        }
      }
EOL
    SHELL
    web01.vm.provision "shell", name: "n8n", inline: <<-'SHELL'
    sudo systemctl stop nginx
    sudo systemctl enable nginx
    docker volume create n8n_data
    docker rm -v -f $(docker ps -qa)
    docker run --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
  SHELL
  end
end
