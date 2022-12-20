# -*- mode: ruby -*-
# vi: set ft=ruby :

$default_network_interface = `ip route | awk '/^default/ {printf "%s", $5; exit 0}'`

manager_resources = {"memory" => "256", "cpus" => "1"}
worker_resources = {"memory" => "256", "cpus" => "1"}

machines = {
  "manager" => {"resources" => manager_resources},
#  "worker3" => {"resources" => manager_resources},
#  "worker2" => {"resources" => manager_resources},
  "worker1" => {"resources" => manager_resources}
}

Vagrant.configure("2") do |config|

  machines.each do |name, conf|
    config.vm.define "#{name}" do |machine|
      machine.vm.box = "ubuntu/jammy64"

      machine.vm.network "public_network", bridge: "#$default_network_interface"

      machine.vm.provider "virtuabox" do |vb|
        vb.name = "#{name}"
        vb.memory = conf["memory"]
        vb.cpus = conf["cpus"]
      end

      machine.vm.provision "shell", inline: <<-SHELL
        curl -o get-docker.sh https://get.docker.com
        sh get-docker.sh
      SHELL

      if name == "manager"
        machine.vm.provision "shell", inline: <<-SHELL
          CIDR=$(ip a show label enp0s8 | awk '/^\s*inet / {printf "%s", $2; exit 0}')
          ADDR=$(echo $CIDR | sed -e 's/\\/[0-9]*//')
          sudo docker swarm init --advertise-addr $ADDR
          JOIN_COMMAND=$(sudo docker swarm join-token worker | grep docker)
          echo $JOIN_COMMAND > /vagrant/join.sh
        SHELL
      else
        machine.vm.provision "shell", path: "join.sh"
      end

    end
  end
end
