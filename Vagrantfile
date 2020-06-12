IMAGE_NAME = "centos/7"
MASTER_COUNT = 2
WORKER_COUNT = 2
LOAD_BALANCE_IP="192.168.50.50"
HOME_IP="192.168.50.100"


# global script
$script = <<SCRIPT
 
echo " ******** Install Tools ***********"

sudo yum update -y
# Install wget curl and sshpass
sudo yum install wget curl sshpass git python3 -y
 
echo " ********  SSH Config   ***********"

yes y |ssh-keygen -f /home/vagrant/.ssh/id_rsa -t rsa -N ''

# Exclude node* from host checking
cat > /home/vagrant/.ssh/config <<EOF
Host *
   StrictHostKeyChecking no
   UserKnownHostsFile=/dev/null
EOF

echo "Now config Master nodes"
for x in {11..#{10+MASTER_COUNT}}; do
  sshpass -p vagrant ssh-copy-id -o "StrictHostKeyChecking no" -i /home/vagrant/.ssh/id_rsa vagrant@192.168.50.${x}
  echo "* REGISTED for master vagrant@${x} *"
done

echo "Now config Worker nodes"

for x in {21..#{20+WORKER_COUNT}}; do
  sshpass -p vagrant ssh-copy-id -o "StrictHostKeyChecking no" -i /home/vagrant/.ssh/id_rsa vagrant@192.168.50.${x}
  echo "* REGISTED for worker vagrant@${x} *"
done

echo "Now config LB nodes"

sshpass -p vagrant ssh-copy-id -o "StrictHostKeyChecking no" -i /home/vagrant/.ssh/id_rsa vagrant@#{LOAD_BALANCE_IP}

chown vagrant:vagrant /home/vagrant/.ssh/id_rsa 
chown vagrant:vagrant /home/vagrant/.ssh/config
systemctl sshd reload
echo " ********   Script end  ***********"

SCRIPT

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
    end

    (1..MASTER_COUNT).each do |i|
        config.vm.define "master-#{i}" do |master|
            master.vm.box = IMAGE_NAME
            master.vm.network "private_network", ip: "192.168.50.#{i + 10}"
            master.vm.hostname = "master-#{i}"
        end
    end


    (1..WORKER_COUNT).each do |i|
        config.vm.define "node-#{i}" do |node|
            node.vm.box = IMAGE_NAME
            node.vm.network "private_network", ip: "192.168.50.#{i + 20}"
            node.vm.hostname = "node-#{i}"
        end
    end

    config.vm.define "k8s-haproxy" do |lb|
        lb.vm.box = IMAGE_NAME
        lb.vm.network "private_network", ip: LOAD_BALANCE_IP
        lb.vm.hostname = "k8s-lb"
    end

    config.vm.define "home" do |home|
        home.vm.box = IMAGE_NAME
        home.vm.network "private_network", ip: HOME_IP
        home.vm.hostname = "home"
        home.vm.provision "sshconf", type: "shell",  inline: $script
    end

    config.vm.provision :shell, inline: <<-SHELL
	  	sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
	  	systemctl restart sshd
	  SHELL

end
