require 'json'

file = File.read('requirements.json')

config_hash = JSON.parse(file)

Vagrant.configure("2") do |config|

  config_hash['machines'].each do |machine|

    config.vm.define machine['name'] do |machine_config|

      machine_config.vm.box = machine['box']

      machine_config.vm.hostname = machine['name']

      machine['ips'].each do |ip|
        machine_config.vm.network "private_network", ip: ip
      end

      machine_config.vm.provision "shell", inline: <<-SHELL

        apt-get update

        apt-get install -y \
            ca-certificates \
            wireguard \
            curl

        apt-get upgrade -y

      SHELL

      if machine['provision_scripts'].is_a?(Array)
        machine['provision_scripts'].each do |script|
          machine_config.vm.provision "shell", privileged: true, path: script
        end
      end

      machine_config.vm.provider "virtualbox" do |vb|
        vb.gui = false
        vb.name = machine['name']
        vb.memory = machine['memory']
        vb.cpus = machine['cpus']
      end

    end

  end

end
