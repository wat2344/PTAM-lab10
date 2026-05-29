# PTAM-lab10

Данная лабораторная работа посвящена изучению процесса создания и конфигурирования виртуальной среды разработки с использованием Vagrant.

## Tutorial

```bash
export GITHUB_USERNAME=wat2344
export PACKAGE_MANAGER="sudo apt"
cd ${GITHUB_USERNAME}/workspace
${PACKAGE_MANAGER} install vagrant -y
vagrant version
```
### Инициализация Vagrant-окружения
```bash
mkdir lab10 && cd lab10
vagrant init bento/ubuntu-19.10
less Vagrantfile
vagrant init -f -m bento/ubuntu-19.10
mkdir shared
```
### Редактирование Vagrantfile
```bash
cat > Vagrantfile <<EOF
\$script = <<-SCRIPT
sudo apt install docker.io -y
sudo docker pull fastide/ubuntu:19.04
sudo docker create -ti --name fastide fastide/ubuntu:19.04 bash
sudo docker cp fastide:/home/developer /home/
sudo useradd developer
sudo usermod -aG sudo developer
echo "developer:developer" | sudo chpasswd
sudo chown -R developer /home/developer
SCRIPT
EOF
```
```bash
cat >> Vagrantfile <<EOF

Vagrant.configure("2") do |config|

  config.vagrant.plugins = ["vagrant-vbguest"]
EOF
```
```bash
cat >> Vagrantfile <<EOF

  config.vm.box = "bento/ubuntu-19.10"
  config.vm.network "public_network"
  config.vm.synced_folder('shared', '/vagrant', type: 'rsync')

  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
    vb.memory = "2048"
  end

  config.vm.provision "shell", inline: \$script, privileged: true

  config.ssh.extra_args = "-tt"

end
EOF
```
```bash
vagrant validate
```
### Запуск и управление виртуальной машиной
```bash
vagrant status
vagrant up --provider virtualbox
vagrant port
vagrant status
vagrant ssh
```
### Работа со снимками (snapshots)
```bash
vagrant snapshot list
vagrant snapshot push
vagrant snapshot list
vagrant halt
vagrant snapshot pop
```
### Провайдер VMware ESXi
```bash
cat >> Vagrantfile <<EOF

  config.vm.provider :vmware_esxi do |esxi|
    esxi.esxi_hostname = '<exsi_hostname>'
    esxi.esxi_username = 'root'
    esxi.esxi_password = 'prompt:'
    esxi.esxi_hostport = 22
    esxi.guest_name = '${GITHUB_USERNAME}'
    esxi.guest_username = 'vagrant'
    esxi.guest_memsize = '2048'
    esxi.guest_numvcpus = '2'
    esxi.guest_disk_type = 'thin'
  end
EOF

vagrant plugin install vagrant-vmware-esxi
vagrant plugin list
# vagrant up --provider=vmware_esxi (требуется наличие ESXi)
```
## Вывод
В ходе лабораторной работы были освоены базовые возможности Vagrant: инициализация окружения, настройка Vagrantfile, запуск виртуальной машины с использованием провайдера VirtualBox, синхронизация папок, выполнение provisioning-скриптов (установка Docker, создание пользователя), а также управление снимками состояния. Vagrant позволяет создавать воспроизводимые и изолированные среды разработки.
