Vagrant.configure("2") do |config|

    # Global provisioning command copy your ssh key into all vms
    config.vm.provision "file", source: "~/.ssh", destination: "/tmp"

    # Setup Docker Host
    config.vm.define :dockerhost do |dockerhost|

        dockerhost.vm.define "dockerhost"
        dockerhost.vm.box = "minimal/trusty64"
        dockerhost.vm.hostname = "doc-srv-01"
        dockerhost.vm.box_check_update = true
        dockerhost.vm.network "public_network", bridge: "eth0"
        dockerhost.vm.network "private_network", ip: "192.168.33.20"
        dockerhost.vm.post_up_message = "Welcome to Docker Host"
        dockerhost.vm.synced_folder "~/Sites", "/Sites"
        dockerhost.vm.provision "file", source: "hosts/docker/pitchDocker", destination: "/tmp"
        dockerhost.vm.network "forwarded_port", guest: 80, host: 8080

        # Configure VBox settings
        dockerhost.vm.provider :virtualbox do |vb|
            vb.name = "Docker"
            vb.gui = false
            vb.memory = 2576
            vb.cpus = 3
            vb.customize [
                "modifyvm", :id,
                "--cpuexecutioncap", "75"
            ]
        end

        # Install docker-engine && docker-compose && shipyard management interface
        dockerhost.vm.provision "shell", inline: <<-SHELL
            cp -Rf /tmp/.ssh/* /root/.ssh/
            sudo apt-get update
            sudo apt-get upgrade -y --force-yes
            sudo apt-get install apt-transport-https ca-certificates build-essential curl wget htop -y --force-yes
            sudo apt-get update
            sudo apt-get purge lxc-docker
            sudo apt-cache policy docker-engine
            wget -qO- https://get.docker.com/ | sh
            sudo usermod -aG docker $(whoami)
            sudo apt-get install -y --force-yes python-pip --fix-missing
            pip install docker-compose
            curl -s https://shipyard-project.com/deploy | bash -s
            cd /tmp/pitchDocker
            docker build -t jkirkby91/pitchbase -f pitchbase/Dockerfile .
            cd /Sites/pitchDocker
            docker-compose up -d
        SHELL
    end

    # Setup loadbalancer/reverse proxy for containers
    config.vm.define :loadbalancer do |loadbalancer|
        loadbalancer.vm.define "loadbalancer"
        loadbalancer.vm.hostname = "lb-srv-01"
        loadbalancer.vm.box = "minimal/trusty64"
        loadbalancer.vm.box_check_update = true
        loadbalancer.vm.network "private_network", ip:"192.168.33.10"
        loadbalancer.vm.network "public_network", bridge: "eth0"

        # Configure VBox settings
        loadbalancer.vm.provider :virtualbox do |vb|
            vb.name = "LoadBalancer"
            vb.gui = false
            vb.memory = 2576
            vb.cpus = 3
            vb.customize [
                "modifyvm", :id,
                "--cpuexecutioncap", "75"
            ]
        end

        # Run some provisions on the vm
        loadbalancer.vm.provision "shell", inline: <<-SHELL
            sudo apt-get update
            sudo apt-get install -y linux-image-extra-$(uname -r) apt-transport-https ca-certificates build-essential nginx htop --fix-missing
            cp -Rf /tmp/.ssh/ /root/
        SHELL
    end

    # Setup our MySql server
    config.vm.define :mysql do |mysql|
        mysql.vm.define "MySQL"
        mysql.vm.hostname = "db-srv-01"
        mysql.vm.box = "minimal/trusty64"
        mysql.vm.box_check_update = true
        mysql.vm.network "private_network", ip:"192.168.33.20"
        mysql.vm.network "public_network", bridge: "eth0"

        # Configure VBox settings
        mysql.vm.provider :virtualbox do |vb|
            vb.name = "MySQL"
            vb.gui = false
            vb.memory = 2576
            vb.cpus = 3
            vb.customize [
                "modifyvm", :id,
                "--cpuexecutioncap", "75"
            ]
        end

        # Run some provisions on the vm
        mysql.vm.provision "shell", inline: <<-SHELL
            debconf-set-selections <<< 'mysql-server-5.5 mysql-server/root_password password root'
            debconf-set-selections <<< 'mysql-server-5.5 mysql-server/root_password_again password root'
            sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
            sudo apt-get update
            sudo apt-get install -y linux-image-extra-$(uname -r) apt-transport-https ca-certificates build-essential mysql-server htop
            cp -Rf /tmp/.ssh/ /root/
        SHELL
    end
end
