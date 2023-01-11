Vagrant.configure("2") do |config|
config.vm.define "server" do |master|
  master.vm.box = "ubuntu/bionic64"
  master.vm.hostname = "master-node"
  master.vm.network "public_network", ip: "192.168.5.100"
  master.vm.provider "virtualbox" do |vb|
      vb.memory = 2048
      vb.cpus = 2
  end
end
(1..1).each do |i|
config.vm.define "client0#{i}" do |node|
node.vm.box = "ubuntu/bionic64"
node.vm.hostname = "worker-node0#{i}"
node.vm.network "public_network", ip: "192.168.5.10#{i}"
node.vm.provider "virtualbox" do |vb|
vb.memory = 2048
vb.cpus = 2
end
end
end
end
