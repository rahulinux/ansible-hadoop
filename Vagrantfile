# -*- mode: ruby -*-
# vim: set ft=ruby :

require "ipaddr"
require "fileutils"

box = "bento/centos-7.2"
domain = "hdp.local"
vm_storage_path = File.dirname(__FILE__) + '/disks'


masters =[
 {
    :hostname => "nn",
    :ip => "192.168.100.10",
    :ram => 1512,
    :cpu => 2,
    :disk2_size => "1",
    :disk_format => "false"
  }
]


data_nodes={
    :hostname => "dn",
    :ram => 1024,
    :cpu => 4,
    :disk2_size => "1",
    :count => 3,
    :disk_format => "false"
}

ip = IPAddr.new "#{masters[0][:ip]}"

unless File.directory?(vm_storage_path)
   puts "Creating Dir for disk"
   FileUtils.mkdir_p(vm_storage_path)
end

Vagrant.configure(2) do |config|

    masters.each do |machine|
        config.vm.define machine[:hostname] do |node|
            node.vbguest.auto_update = false
            node.vm.box = "#{box}"
            node.vm.hostname = "#{machine[:hostname]}" + "." + "#{domain}" 
            node.vm.network "private_network", ip: machine[:ip]
            node.vm.provider "virtualbox" do |vb|
                vb.customize ["modifyvm", :id, "--memory", machine[:ram]]
                vb.customize ["modifyvm", :id, "--cpus", machine[:cpu]]
                disk = "#{vm_storage_path}/#{machine[:hostname]}/box-disk2.vmdk"
                unless File.exists?(disk)
                    vb.customize  ["createmedium", "--filename", disk, "--size",machine[:disk2_size].to_i * 1024 ]
		    machine[:disk_format] = "true"
                end 
                vb.customize  ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 1, "--device", 0, "--type", "hdd", "--medium", disk ]
            end
        end
    end

    1.upto(data_nodes[:count]).each.with_index(1) do |machine,index|
        config.vm.define "#{data_nodes[:hostname]}-#{index}" do |node|
            node.vbguest.auto_update = false
            node.vm.box = "#{box}"
            node.vm.hostname = "#{data_nodes[:hostname]}-#{index}" + "." + "#{domain}" 
            node.vm.network "private_network", ip: "#{ip.succ}"
            node.vm.provider "virtualbox" do |vb|
                vb.customize ["modifyvm", :id, "--memory",data_nodes[:ram]]
                vb.customize ["modifyvm", :id, "--cpus",data_nodes[:cpu]]
                disk = "#{vm_storage_path}/#{data_nodes[:hostname]}-#{index}/box-disk2.vmdk"
                unless File.exists?(disk)
                    vb.customize  ["createmedium", "--filename", disk, "--size",data_nodes[:disk2_size].to_i * 1024 ]
		    data_nodes[:disk_format] = "true"
                end 
                vb.customize  ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 1, "--device", 0, "--type", "hdd", "--medium", disk ]
             end
            ip = IPAddr.new "#{ip.succ}"
            if (index == data_nodes[:count])
            then
                config.vm.provision "ansible" do |ansible|
                  ansible.limit = "all,localhost"
                  ansible.groups = {
                    "master-nodes" => ["#{masters[0][:hostname]}"],
                    "slave-nodes" => [ "#{data_nodes[:hostname]}" + "[1:index]" ],
                    "all_groups:children" => [ "master-nodes", "slave-nodes" ]
                  }
                  ansible.playbook = "playbooks/site.yaml"
                  ansible.verbose = "vvv"
                end
            end
        end
    end
end
