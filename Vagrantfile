Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  config.vm.define "dns-primary" do |primary|
    primary.vm.hostname = "dns-primary"
    primary.vm.network "public_network",
      ip: "192.168.20.10",
      bridge: "Intel(R) Gigabit ET Dual Port Server Adapter"
    primary.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 1
    end
  end

  config.vm.define "dns-secondary" do |secondary|
    secondary.vm.hostname = "dns-secondary"
    secondary.vm.network "public_network",
      ip: "192.168.20.11",
      bridge: "Intel(R) Gigabit ET Dual Port Server Adapter"
    secondary.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 1
    end
  end
end
