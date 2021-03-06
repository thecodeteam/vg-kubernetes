# Created by Jonas Rosland, @virtualswede & Matt Cowger, @mcowger
# Many thanks to this post by James Carr: http://blog.james-carr.org/2013/03/17/dynamic-vagrant-nodes/

# vagrant box
vagrantbox="centos_7.3"

# vagrant box url
vagrantboxurl="https://github.com/CommanderK5/packer-centos-template/releases/download/0.7.3/vagrant-centos-7.3.box"

# scaleio admin password
password="Scaleio123"
# add your domain here
domain = 'scaleio.local'

# add your nodes here
nodes = ['master', 'node01','node02']

# add your IPs here
network = "192.168.50"

clusterip = "#{network}.10"
master_ip = "#{network}.11"
node01_ip = "#{network}.12"
node02_ip = "#{network}.13"

# Install ScaleIO cluster automatically or IM only
#If True a fully working ScaleIO cluster is installed. False mean only IM is installed on node MDM1.
if ENV['SCALEIO_INSTALL']
  scaleio_install = ENV['SCALEIO_INSTALL'].to_s.downcase
else
  scaleio_install = "true"
end

if ENV['SCALEIO_PASSWORD']
  scaleio_password = ENV['SCALEIO_PASSWORD']
else
  scaleio_password = "Scaleio123"
end

# Install and Configure Kubernetes Automatically
if ENV['K8S_VERSION']
  k8s_version = ENV['K8S_VERSION'].to_s.downcase
else
  k8s_version = "1.9.2"
end

# Enable Alpha Support for CSI
if ENV['K8S_CSI']
  k8s_csi = ENV['K8S_CSI'].to_s.downcase
else
  k8s_csi = "true"
end

# In some cases more memory is needed for applications.
# this environment variable is used to set the mount of RAM for MDM2 and TB. MDM1 always gets 3GB.
# must be set in 1024 amounts
if ENV['VM_MEMORY']
  vmram = ENV['VM_MEMORY'].to_s.downcase
else
  vmram = "1024"
end

# Verify that the ScaleIO package has the correct size
if ENV['VERIFY_FILES']
  verifyfiles = ENV['VERIFY_FILES'].to_s.downcase
else
  verifyfiles = "true"
end

# loop through the nodes and set hostname
scaleio_nodes = []
subnet=10
nodes.each { |node_name|
  (1..1).each {|n|
    subnet += 1
    scaleio_nodes << {:hostname => "#{node_name}"}
  }
}

Vagrant.configure("2") do |config|
  if Vagrant.has_plugin?("vagrant-proxyconf")
    #config.proxy.http     = "http://proxy.example.com:3128/"
    #config.proxy.https    = "http://proxy.example.com:3128/"
    #config.proxy.no_proxy = "localhost,127.0.0.1,.example.com"
  end
  #config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  scaleio_nodes.each do |node|
    config.vm.define node[:hostname] do |node_config|
      node_config.vm.box = "#{vagrantbox}"
      node_config.vm.box_url = "#{vagrantboxurl}"
      node_config.vm.host_name = "#{node[:hostname]}"
      node_config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", vmram]
        #vb.customize ['storagectl', :id, '--name', 'SATA Controller', '--portcount', 2, '--hostiocache', 'on']
        #vb.customize ['createhd', '--filename', ".vagrant/machines/#{node[:hostname]}/dockervol.vmdk", '--format', 'vmdk', '--size', 15 * 1024]
        #vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', ".vagrant/machines/#{node[:hostname]}/dockervol.vmdk"]
      end

      if node[:hostname] == "master"
        node_config.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "3072"]
        end

        node_config.vm.network "private_network", ip: "#{master_ip}"
        # node_config.vm.network "forwarded_port", guest: 6611, host: 6611

        # adding certificates needed for k8s installation
        node_config.vm.provision "file", source: "scripts/certs", destination: "/home/vagrant/certs"
        node_config.vm.provision "file", source: "scripts/examples", destination: "/home/vagrant"


        node_config.vm.provision "shell" do |s|
          s.path = "scripts/master.sh"
          s.args = "-k8sv #{k8s_version} -csi #{k8s_csi} -msip #{master_ip} -n1ip #{node01_ip} -n2ip #{node02_ip} -si #{scaleio_install} -vf #{verifyfiles}"
        end
      end

      if node[:hostname] == "node01"
        node_config.vm.network "private_network", ip: "#{node01_ip}"
        # adding certificates needed for k8s installation
        node_config.vm.provision "file", source: "scripts/certs", destination: "/home/vagrant/certs"

        node_config.vm.provision "shell" do |s|
          s.path = "scripts/node01.sh"
          s.args = "-k8sv #{k8s_version} -csi #{k8s_csi} -msip #{master_ip} -n1ip #{node01_ip} -n2ip #{node02_ip} -si #{scaleio_install} -vf #{verifyfiles}"
        end
      end

      if node[:hostname] == "node02"
        node_config.vm.network "private_network", ip: "#{node02_ip}"
        # adding certificates needed for k8s installation
        node_config.vm.provision "file", source: "scripts/certs", destination: "/home/vagrant/certs"

        node_config.vm.provision "shell" do |s|
          s.path = "scripts/node02.sh"
          s.args = "-k8sv #{k8s_version} -csi #{k8s_csi} -msip #{master_ip} -n1ip #{node01_ip} -n2ip #{node02_ip} -si #{scaleio_install} -sp #{scaleio_password} -vf #{verifyfiles}"
        end
      end

    end
  end
end
