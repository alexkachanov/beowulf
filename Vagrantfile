# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: "echo Hello. Preparing Simplebox!"
  config.vbguest.auto_update = false
  config.vm.box = "almalinux/9"

  # ==========================================================================
  # creating node0 that wll be used to control cluster
  config.vm.define "node0" do |machine|
    machine.vm.define "node0"
    machine.vm.hostname = "node0"

    machine.vm.network "private_network", ip: "172.17.177.99"
    machine.vm.network "forwarded_port", guest: 443, host: 8440

    machine.vm.provider :virtualbox do |vb|
        vb.name = "node0"
	vb.cpus = 6
        vb.memory = 8000
	vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end

    # stopping internal firewall
    machine.vm.provision "shell", inline: "systemctl stop firewalld"
    machine.vm.provision "shell", inline: "echo '172.17.177.99 node0' | tee -a /etc/hosts"
    machine.vm.provision "shell", inline: "echo '172.17.177.11 node1' | tee -a /etc/hosts"
    machine.vm.provision "shell", inline: "echo '172.17.177.12 node2' | tee -a /etc/hosts"

    machine.vm.provision "shell", inline: "yum install openssh-server -y"
    machine.vm.provision "shell", inline: "yum install mpich-devel -y"
    machine.vm.provision "shell", inline: "yum install mpitests-mpich -y"

    machine.vm.provision "shell", inline: "adduser mpiuser --uid 1001"
    #machine.vm.provision "shell", inline: "echo 'export PATH=/usr/lib64/mpich/bin:${PATH}' >> ~/.bash_profile", privileged: false
    #machine.vm.provision "shell", inline: "echo 'export LD_LIBRARY_PATH=/usr/lib64/mpich/lib:${LD_LIBRARY_PATH}' >> ~/.bash_profile", privileged: false
    #machine.vm.provision "shell", inline: "ssh-keygen -q -N '' -t rsa -f ~/.ssh/id_rsa", privileged: false
    #machine.vm.provision "shell", inline: "ssh-copy-id node0", privileged: false
    #machine.vm.provision "shell", inline: "cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys", privileged: false


    machine.vm.provision "shell", inline: "yum install nfs-utils -y"
    machine.vm.provision "shell", inline: "systemctl enable --now nfs-server.service"
    
    # Sharing the NFS folder (the home folder of the mpiuser created for the MPI’s purposes) is done by adding the line
    machine.vm.provision "shell", inline: "echo '/home/mpiuser *(rw,sync,no_subtree_check)' | tee -a /etc/exports"
    machine.vm.provision "shell", inline: "service nfs-server.service restart"
    # important for RHEL/CentOS/AlmaLinux versions: SELinux can be a obstacle
    machine.vm.provision "shell", inline: "setsebool -P use_nfs_home_dirs=true"
    machine.vm.provision "shell", inline: "exportfs -arv"

    machine.vm.post_up_message = "node0 is ready"

  end

  # ==========================================================================
  # creating node1 that wll be used to execute jobs
  config.vm.define "node1" do |machine|
    machine.vm.define "node1"
    machine.vm.hostname = "node1"

    machine.vm.network "private_network", ip: "172.17.177.11"
    machine.vm.network "forwarded_port", guest: 443, host: 8441

    machine.vm.provider :virtualbox do |vb|
        vb.name = "node1"
	vb.cpus = 6
        vb.memory = 8000
	vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
    machine.vm.provision "shell", inline: "systemctl stop firewalld"
    machine.vm.provision "shell", inline: "echo '172.17.177.99 node0' | tee -a /etc/hosts"
    machine.vm.provision "shell", inline: "echo '172.17.177.11 node1' | tee -a /etc/hosts"
    machine.vm.provision "shell", inline: "echo '172.17.177.12 node2' | tee -a /etc/hosts"

    machine.vm.provision "shell", inline: "yum install openssh-server -y"
    machine.vm.provision "shell", inline: "yum install mpich-devel -y"

    machine.vm.provision "shell", inline: "adduser mpiuser --uid 1001"

    machine.vm.provision "shell", inline: "yum install nfs-utils -y"

    # Then the NFS shared folder needs to be mounted on each node
    machine.vm.provision "shell", inline: "mount node0:/home/mpiuser /home/mpiuser"
    # important for RHEL/CentOS/AlmaLinux versions: SELinux can be a obstacle
    machine.vm.provision "shell", inline: "setsebool -P use_nfs_home_dirs=true"

    machine.vm.post_up_message = "node1 is ready"

  end

end