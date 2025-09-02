# POC - Cluster kubernetes com Kubeadm e ContainerD - CNI = calico

Esse é um ambiente de estudos criado por meio da ferramenta Vagrant que faz a criação das VMs utilizando o Virtual Box.
Essa POC se trata de um cluster kubernetes criado com kubeadm e containerD como o container runtime.
<br>

Por padrão o arquivo de criação do ambiente cria um cluster com:
- 1 VM master/control-plane (k8s-master) - 2GB e 2CPUs
- 2 VMs nodes (k8s-node01 e k8s-node02)- 2GB e 1CPUs

Essas configurações podem ser alteradas no arquivo Vagrantfile, caso você precise alterar a quantidade de nodes ou master na POC.

> [!Pré-requisitos:]
> Para criar este ambiente vocẽ precisará ter o Virtual Box e o Vagrant instalados em seu S.O


## 🛠️ Criando o ambiente da POC:

Para criar o ambiente, tenha em mente os pré-requisitos acima.
Após baixar esse repositório rode o comando do vagrant para criar o ambiente (para rodar o comando abaixo é necessário que você esteja no mesmo diretório do Vagrantfile):

```bash
vagrant up
```

Após criado os ambientes, para acesar o SSH das VMs basta rodar o comando:

```bash
vagrant ssh NOME_DA_VM

# Exemplo
vagrant ssh k8s-master
```

## 💡 Comandos Vagrant úteis

Para parar/desligar (sem excluir) todas as VMs criadas pelo vagrant:

```bash
vagrant halt
```

Para excluir as VMs criadas pelo vagrant:

```bash
vagrant destroy
```
