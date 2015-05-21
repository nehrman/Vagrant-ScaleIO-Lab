# Created by Jonas Rosland, @virtualswede & Matt Cowger, @mcowger
# Modified by Nicolas Ehrman
# Many thanks to this post by James Carr: http://blog.james-carr.org/2013/03/17/dynamic-vagrant-nodes/

# vagrant box
vagrantbox="centos7-mini"

# vagrant box url
vagrantboxurl="file://Users/Nicolas/vagrant/centos7-x64.box"

# scaleio admin password
password="Scaleio123"
# add your domain here
domain = 'testlab.local'

# add your nodes here
nodes = ['tb','mdm','sds','sdc']

# add your IPs here
network = "172.16.204"

clusterip = "#{network}.101"
tbip = "#{network}.102"
firstmdmip = "#{network}.103"
secondmdmip = "#{network}.104"

# Install ScaleIO cluster automatically or IM only
clusterinstall = "True" #If True a fully working ScaleIO cluster is installed. False mean only IM is installed on node MDM1.

# version of installation package
version = "1.31-2333.2"

#OS Version of package
os="el7"

# installation folder
siinstall = "/opt/scaleio/siinstall"

# packages folder
packages = "/opt/scaleio/siinstall/ECS/packages"

# package name is now EMC-ScaleIO from 1.31
packagename = "EMC-ScaleIO"

# Disk Size in MB
disk_size="102400MB"

# Define Device
device = "/dev/sdb"

# loop through the nodes and set hostname
scaleio_nodes = []
subnet=10
nodes.each { |node_name|
  (1..10).each {|n|
  	nodenum = "#{n}".rjust(2,'0')
    subnet += 1
    scaleio_nodes << {:hostname => "#{node_name}#{nodenum}",  :ip => "#{network}.#{subnet}",
    }
  }
}

Vagrant.configure("2") do |config|
  if Vagrant.has_plugin?("vagrant-proxyconf")
    #config.proxy.http     = "http://proxy.example.com:3128/"
    #config.proxy.https    = "http://proxy.example.com:3128/"
    #config.proxy.no_proxy = "localhost,127.0.0.1,.example.com"
  end
  scaleio_nodes.each do |node|
    config.vm.define node[:hostname] do |node_config|
      node_config.ssh.password = 'vagrant'
      node_config.vm.box = "#{vagrantbox}"
      node_config.vm.box_url = "#{vagrantboxurl}"
      node_config.vm.host_name = "#{node[:hostname]}.#{domain}"
      node_config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "2048"]
      node_config.vm.provider :vmware_fusion do |vmw|
            vdiskmanager = '/Applications/VMware\ Fusion.app/Contents/Library/vmware-vdiskmanager'
     	 dir = "#{ENV['PWD']}/.vagrant/additional-disks"
     	 unless File.directory?( dir )
      	 Dir.mkdir dir
      	end
      	file_to_disk = "#{dir}/#{node[:hostname]}-hd2.vmdk"
      	unless File.exists?( file_to_disk )
        `#{vdiskmanager} -c -s #{disk_size} -a lsilogic -t 0 #{file_to_disk}`
      	end
      	vmw.vmx['scsi0:1.filename'] = file_to_disk
      	vmw.vmx['scsi0:1.present']  = 'TRUE'
      	vmw.vmx['scsi0:1.redo']     = ''
      	vmw.vmx["memsize"] = "2048"
  		vmw.vmx["numvcpus"] = "1"
      	end
      end
      if node[:hostname] == "tb01"
        node_config.vm.network "private_network", ip: "#{tbip}"
        node_config.vm.provision "shell" do |s|
          s.path = "scripts/tb.sh"
          s.args   = "-o #{os} -v #{version} -n #{packagename} -d #{device} -f #{firstmdmip} -s #{secondmdmip} -i #{siinstall} -c #{clusterinstall}"
        end
      end

      if node[:hostname] == "mdm01"
        node_config.vm.network "private_network", ip: "#{firstmdmip}"
        node_config.vm.network "forwarded_port", guest: 6611, host: 6611
        node_config.vm.provision "shell" do |s|
          s.path = "scripts/mdm1.sh"
          s.args   = "-o #{os} -v #{version} -n #{packagename} -d #{device} -f #{firstmdmip} -s #{secondmdmip} -i #{siinstall} -p #{password} -c #{clusterinstall} -h #{node[:hostname]} "
        end
      end

      if node[:hostname] == "mdm02"
        node_config.vm.network "private_network", ip: "#{secondmdmip}"
        node_config.vm.provision "shell" do |s|
          s.path = "scripts/mdm2.sh"
          s.args   = "-o #{os} -v #{version} -n #{packagename} -d #{device} -f #{firstmdmip} -s #{secondmdmip} -i #{siinstall} -t #{tbip} -p #{password} -c #{clusterinstall} -h #{node[:hostname]} "
        end
      end
            if "#{node[:hostname]}".include? 'sds'
        node_config.vm.network "private_network", ip: "#{node[:ip]}"
        node_config.vm.provision "shell" do |s|
          s.path = "scripts/sds.sh"
          s.args   = "-o #{os} -v #{version} -n #{packagename} -d #{device} -f #{firstmdmip} -s #{secondmdmip} -i #{siinstall}  -p #{password} -c #{clusterinstall} -b #{node[:ip]} -h #{node[:hostname]} "
        end
      end
            if "#{node[:hostname]}".include? 'sdc'
        node_config.vm.network "private_network", ip: "#{node[:ip]}"
        node_config.vm.provision "shell" do |s|
          s.path = "scripts/sdc.sh"
          s.args   = "-o #{os} -v #{version} -n #{packagename} -d #{device} -f #{firstmdmip} -s #{secondmdmip} -i #{siinstall} -t #{tbip} -p #{password} -c #{clusterinstall} -b #{node[:ip]} -h #{node[:hostname]} "
        end
      end
    end
  end
end