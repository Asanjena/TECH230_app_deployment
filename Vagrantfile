
Vagrant.configure("2") do |config|

  # configures the VM settings
  config.vm.box = "ubuntu/xenial64" 

  # sets private ip
  config.vm.network "private_network", ip:"192.168.10.100"

  # provision the VM to have Nginx
  config.vm.provision "shell", path: "provision.sh"
  
  #put/syncs app folder from local macine to the VM
  config.vm.synced_folder "app", "/home/vagrant/app"

end
