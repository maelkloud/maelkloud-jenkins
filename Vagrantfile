# Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64" # Ubuntu 22.04
  config.vm.provider "virtualbox" do |vb|
    vb.name = "jenkins-ubuntu"
    vb.memory = "4096" # 4GB of RAM
  end

  # Disk resizing configuration
  config.vm.disk :disk, size: "50GB", primary: true

  # Network configuration
  config.vm.network "public_network", bridge: "en0: Wi-Fi (Wireless)"
  config.vm.network "forwarded_port", guest: 8080, host: 8080

  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update
    sudo apt-get install -y openjdk-11-jdk
    # Add the Jenkins repository
    curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null


    # Install Jenkins
    sudo apt-get update
    sudo apt-get install -y fontconfig 
    sudo apt-get install -y openjdk-11-jre
    sudo apt-get install -y jenkins
    sudo systemctl start jenkins
    sudo systemctl enable jenkins
  SHELL
end
