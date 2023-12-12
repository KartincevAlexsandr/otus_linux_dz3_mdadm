# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
    :otuslinux => {
        :box_name => "centos/7",
        :ip_addr => '192.168.56.101',
        :disks => {
            :sata1 => {
                :dfile => './sata1.vdi',
                :size => 250,
                :port => 1
            },
            :sata2 => {
                :dfile => './sata2.vdi',
                :size => 250,
                :port => 2
            },
            :sata3 => {
                :dfile => './sata3.vdi',
                :size => 250,
                :port => 3
            },
            :sata4 => {
                :dfile => './sata4.vdi',
                :size => 250,
                :port => 4
            },
            :sata5 => {
                :dfile => './sata5.vdi',
                :size => 250,
                :port => 5
            }
        }
    },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s
        box.vm.box_version = "2004.01"
        
        box.vm.network "private_network", ip: boxconfig[:ip_addr]

        box.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "1024"]
            needsController = false
            boxconfig[:disks].each do |dname, dconf|
                    unless File.exist?(dconf[:dfile])
                            vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                            needsController =  true
                    end
            end
            if needsController == true
                vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                boxconfig[:disks].each do |dname, dconf|
                    vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                end
            end
        end
        box.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh
            cp ~vagrant/.ssh/auth* ~root/.ssh
            yum install -y mdadm smartmontools hdparm gdisk
            mdadm --zero-superblock --force /dev/sdb /dev/sdc /dev/sdd /dev/sde /dev/sdf    
            mdadm --create --verbose /dev/md0 -l 5 -n 5 /dev/sdb /dev/sdc /dev/sdd /dev/sde /dev/sdf 
            mdadm --detail --scan --verbose
            mkdir /etc/mdadm
            echo "DEVICE partitions" > /etc/mdadm/mdadm.conf 
            mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >>  /etc/mdadm/mdadm.conf
            parted -s /dev/md0 mklabel gpt
            parted /dev/md0 mkpart primary ext4 0% 20%
            parted /dev/md0 mkpart primary ext4 20% 40%
            parted /dev/md0 mkpart primary ext4 40% 60%
            parted /dev/md0 mkpart primary ext4 60% 80%
            parted /dev/md0 mkpart primary ext4 80% 100%
            mkfs.ext4 /dev/md0p1
            mkfs.ext4 /dev/md0p2
            mkfs.ext4 /dev/md0p3
            mkfs.ext4 /dev/md0p4
            mkfs.ext4 /dev/md0p5
            mkdir -p /raid/part1
            mkdir -p /raid/part2
            mkdir -p /raid/part3
            mkdir -p /raid/part4
            mkdir -p /raid/part5
            mount /dev/md0p1 /raid/part1
            mount /dev/md0p2 /raid/part2
            mount /dev/md0p3 /raid/part3
            mount /dev/md0p4 /raid/part4
            mount /dev/md0p5 /raid/part5
            echo "/dev/md0p1 /raid/part1 ext4 defaults 0 1" >> /etc/fstab
            echo "/dev/md0p2 /raid/part2 ext4 defaults 0 1" >> /etc/fstab
            echo "/dev/md0p3 /raid/part3 ext4 defaults 0 1" >> /etc/fstab
            echo "/dev/md0p4 /raid/part4 ext4 defaults 0 1" >> /etc/fstab
            echo "/dev/md0p5 /raid/part5 ext4 defaults 0 1" >> /etc/fstab
            

        SHELL
      end
  end
end