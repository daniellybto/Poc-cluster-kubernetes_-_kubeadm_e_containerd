Vagrant.configure("2") do |config|
    # Pasta compartilhada para trocar dados entre VMs (ex: token do kubeadm)
    config.vm.synced_folder ".", "/vagrant", disabled: false
    config.vm.provision "shell", inline: <<-SHELL
        echo "Tornando o scrip executável..."
        chmod +x /vagrant/code/comum.sh
        /vagrant/code/comum.sh
    SHELL

    (1..1).each do |i|
        # Máquina Master
        config.vm.define "k8s-master" do |k8s|
            k8s.vm.box = "ubuntu/jammy64"
            k8s.vm.hostname = "k8s-master"
            k8s.vm.network "private_network", ip: "192.168.1.1#{i}"

            k8s.ssh.insert_key = false
            k8s.ssh.private_key_path = ['~/.vagrant.d/insecure_private_key', '~/.ssh/id_ed25519']
            k8s.vm.provision "file", source: "~/.ssh/id_ed25519.pub", destination: "~/.ssh/authorized_keys"

            k8s.vm.provider "virtualbox" do |vb|
                vb.gui = false
                vb.cpus = 2
                vb.memory = "2048"
            end

            # Provisionamento do master
            k8s.vm.provision "shell", inline: <<-SHELL

                # Início do Cluster K8S:
                # kubeadm init
                MASTER_PRIVATE_IP='192.168.1.11'
                NODENAME=$(hostname -s)
                POD_CIDR="192.168.1.10/16"

                sudo kubeadm init --apiserver-advertise-address="$MASTER_PRIVATE_IP" --apiserver-cert-extra-sans="$MASTER_PRIVATE_IP" --pod-network-cidr="$POD_CIDR" --node-name "$NODENAME" --ignore-preflight-errors Swap


                # Criação do diretório .kube
                sudo mkdir -p $HOME/.kube
                sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
                sudo chown $(id -u):$(id -g) $HOME/.kube/config

                # Aplicar a instalação do Calico:
                kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

                # Gerando o comando de "Join" no cluster:
                kubeadm token create --print-join-command > /vagrant/code/join.sh


                echo "Master inicializado e pronto usar...."
            SHELL

        end
    end

    # Nós do cluster
    (1..2).each do |i|
        config.vm.define "k8s-node0#{i}" do |k8s|
            k8s.vm.box = "ubuntu/jammy64"
            k8s.vm.hostname = "k8s-node0#{i}"
            k8s.vm.network "private_network", ip: "192.168.1.2#{i}"

            k8s.ssh.insert_key = false
            k8s.ssh.private_key_path = ['~/.vagrant.d/insecure_private_key', '~/.ssh/id_ed25519']
            k8s.vm.provision "file", source: "~/.ssh/id_ed25519.pub", destination: "~/.ssh/authorized_keys"

            k8s.vm.provider "virtualbox" do |vb|
                vb.gui = false
                vb.cpus = 1
                vb.memory = "2048"
            end


            # Provisionamento do nó
            k8s.vm.provision "shell", inline: <<-SHELL
                # Executar o comando de Join para juntar os nodes no Control-Plane
                chmod +x /vagrant/code/join.sh
                /vagrant/code/join.sh

                # Alterar configurações do kubelet para pegar o IP do node, e não o IP padrão
                ## arquivo de configuração do kubelet: /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
                sudo su
                sed -i 's@Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"@Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml --node-ip=192.168.1.2#{i}"@g' /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
                systemctl daemon-reload
                systemctl restart kubelet

                echo "Nó k8s-node0#{i} inicializado e pronto para usar...."
            SHELL


        end
    end
end
