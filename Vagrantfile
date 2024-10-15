Vagrant.configure("2") do |config|

  # Specify that we require a Ubuntu 22.04 VM
  config.vm.box = "ubuntu/jammy64"

  # Specify a static private IP (switch to DCHP if convenient)
  config.vm.network "private_network", ip: "192.168.33.103"

  # Specify machine parameters as specified in the task
  config.vm.provider "virtualbox" do |vb|
    
    vb.name = "IaC CRUD VM 161RDB025"
    vb.memory = "2048"
    vb.cpus = 2

  end

  # Provision shell scripts to set up environment after initial install
  config.vm.provision :shell, path: "provision/nginx.sh"
  config.vm.provision :shell, path: "provision/php.sh"
  config.vm.provision :shell, path: "provision/mysql.sh"

end
