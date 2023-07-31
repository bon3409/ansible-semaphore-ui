Vagrant.configure("2") do |config|

    config.vm.define "app" do |app|
        app.vm.hostname = "app"
        app.vm.box = "ubuntu/jammy64"
        app.vm.box_version = "20230110.0.0"
        app.vm.network "private_network", ip: "192.168.50.50"
    end

    config.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
  end