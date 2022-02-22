Vagrant.configure("2") do |config|
  config.vm.define "v", primary: true do |v|
    v.vm.box = "bento/ubuntu-18.04"
    v.vm.hostname = 'ansible-master'
    v.vm.network :private_network, ip: "192.168.56.151"    
	
	v.vm.provider :virtualbox do |v|
	  v.customize ["modifyvm", :id, "--name", "ansible-master"]
	  v.customize ["modifyvm", :id, "--memory", 1024]
	  v.customize ["modifyvm", :id, "--cpus", 1]
    end	 
 end
end