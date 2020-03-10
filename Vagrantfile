IMAGE_NAME = "debian/stretch64"
N = 2

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 2
    end

    config.vm.define "k8s-master" do |master|
        master.vm.box = IMAGE_NAME
        master.vm.network "private_network", ip: "192.168.50.10"
        master.vm.hostname = "k8s-master"
        master.vm.provision "shell", inline: "apt install python -y"
        master.vm.provision "shell", inline: "apt update"
        master.vm.provision "ansible" do |ansible|
            ansible.playbook = "kubernetes-setup/master-playbook.yml"
            ansible.extra_vars = {
                node_ip: "192.168.50.10"
            }
        end
        master.vm.provider "virtualbox" do |vb|
            vb.name = "k8s-master"
            vb.customize ["modifyvm", :id, "--groups", "/k8s-nodes"]
        end
        master.vm.provision "shell", path: "scripts/setup_ssh.sh"
    end

    (1..N).each do |i|
        config.vm.define "node-#{i}" do |node|
            node.vm.box = IMAGE_NAME
            node.vm.network "private_network", ip: "192.168.50.#{i + 10}"
            node.vm.hostname = "node-#{i}"
            node.vm.provision "shell", inline: "apt install python -y"
            node.vm.provision "shell", inline: "apt update"
            node.vm.provision "ansible" do |ansible|
                ansible.playbook = "kubernetes-setup/node-playbook.yml"
                ansible.extra_vars = {
                    node_ip: "192.168.50.#{i + 10}",
                }
            end
            node.vm.provider "virtualbox" do |vb|
                vb.name = "node-#{i}"
                vb.customize ["modifyvm", :id, "--groups", "/k8s-nodes"]
            end
            node.vm.provision "shell", path: "scripts/setup_ssh.sh"
            node.vm.provision "shell", inline: "source <(kubectl completion bash)"
            node.vm.provision "shell", inline: "echo \"source <(kubectl completion bash)\" >> ~/.bashrc"
        end
    end
end