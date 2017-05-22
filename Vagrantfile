# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # This is an overlaid network made by vyos 

  # Load Vyos box as the network infrastructure 
  config.vm.define "networker" do |n|
    config.vm.box = "vyos"
    # Using guest as :linux is problematic. Must change to ubuntu to pass the network-configuration error
    config.vm.guest= :ubuntu
    config.ssh.username="vyos"
    config.ssh.private_key_path=File.expand_path("../id_rsa", __FILE__) 
    config.vm.network :forwarded_port, guest: 22, host:2222, id: "ssh", disabled: true
    # File Sharing HGFS does not work in the latest Vagrant and Open-VMware-Tools. [todo]
    config.vm.synced_folder ".", "/vagrant", disabled: true  
    config.vm.provider "vmware_fusion" do |vmware|
      vmware.gui = TRUE
      vmware.vmx['ethernet0.startConnected'] = "TRUE"
      vmware.vmx['ethernet0.present'] = "TRUE"
      vmware.vmx['ethernet0.connectionType']="nat"
      vmware.vmx['ethernet1.startConnected'] = "TRUE"
      vmware.vmx['ethernet1.present'] = "TRUE"
      vmware.vmx['ethernet1.connectionType'] = "nat"
      vmware.vmx['ethernet2.startConnected'] = "TRUE"
      vmware.vmx['ethernet2.present'] = "TRUE"
      vmware.vmx['ethernet2.connectionType'] = "hostonly"
      vmware.vm.hostname = "vyos"
      vmware.vmx['vhv.enable']="true"
      vmware.vmx['memsize']="2048"
      vmware.vmx['numvcpus']="2"
      config.vm.provision "ansible" do |ansible|
        ansible.playbook = "provisioning/playbook.yml"
        ansible.groups = {
          "group1" => ["networker"],
          "group1:vars"=> {"v1"=>9, "v2"=>10}
        }
        ansible.verbose="vvv"
      end
    end
  end
  # Photon guest OS to make a load 
  (1..2).each do |i|
    config.vm.box = "photon-tester"
    config.ssh.username="vagrant"
#    config.ssh.private_key_path=File.expand_path("../id_rsa", __FILE__) 
    config.vm.define "photon-#{i}" do |node|
      config.vm.provider "vmware_fusion" do |vmware|
        vmware.gui = TRUE
        vmware.vm.hostname = "photon-#{i}"
        if i==1
          vmware.vmx['ethernet0.connectionType'] = "hostonly"
          vmware.vmx['ethernet0.virtualDev']="vmxnet3"
        else
          vmware.vmx['ethernet0.connectionType'] = "nat"
          vmware.vmx['ethernet0.virtualDev']="vmxnet3"
        end
        node.vm.provision "shell" do |s|
          s.path="provisioning/configAddress.sh"
          s.args="192.168.0.10 192.168.0.1 192.168.62.2"
        end
#        node.vm.provision "ansible" do |ansible|

#        end
      end
    end
  end  
end
