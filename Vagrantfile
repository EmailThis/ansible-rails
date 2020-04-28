Vagrant.configure(2) do |config|
    
    config.vm.box = "hashicorp/bionic64"

    config.vm.network "private_network", ip: "192.168.50.2"
    
    config.vm.provision"ansible" do |ansible| 
        ansible.compatibility_mode = '2.0'
        ansible.playbook = "provision.yml"
        ansible.extra_vars = { ansible_python_interpreter:"/usr/bin/python3" }
    end
    
end