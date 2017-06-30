# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"

  # 
  # MON_SETTINGS
  MON_MEMORY=1024
  N_MON_HOSTS=1
  
  #
  # OSD_SETTINGS
  OSD_MEMORY=1024
  N_OSD_HOSTS=3
  N_DISK_OSD=3

  ### MON HOSTS SETUP
  (1..N_MON_HOSTS).each do |vm_id|
    ip_sufix = 10 + vm_id
    config.vm.define "mon#{vm_id}" do |mon|
      mon.vm.hostname = "mon#{vm_id}"
      mon.vm.network "private_network", ip: "192.168.33.#{ip_sufix}"
      #mon.vm.network "public_network", ovs:true, network_name: "ovsnet", dev:"br0"

      # Provider Virtualbox
      mon.vm.provider :virtualbox do |vb|
        vb.customize ['modifyvm', :id, '--memory', "#{MON_MEMORY}"]
      end

      # Provider Libvirt
      mon.vm.provider :libvirt do |lv|
        lv.memory = MON_MEMORY
      end
    end
  end

  ### OSD HOSTS SETUP
  (1..N_OSD_HOSTS).each do |vm_id|
    ip_sufix = 20 + vm_id
    config.vm.define "osd#{vm_id}" do |osd|
      osd.vm.hostname = "osd#{vm_id}"
      osd.vm.network "private_network", ip: "192.168.33.#{ip_sufix}"
      #osd.vm.network "public_network", ovs:true, network_name: "ovsnet", dev:"br0"

      # Provider Virtualbox
      osd.vm.provider :virtualbox do |vb|
        # Memory
        vb.customize ['modifyvm', :id, '--memory', "#{OSD_MEMORY}"]

        # OSD Disks
        (1..N_DISK_OSD).each do |disk_id|
          vb.customize [
            'createhd',
            '--filename', "disk-osd-#{vm_id}-#{disk_id}",
            '--size', '2048'
          ] unless File.exist?("disk-osd-#{vm_id}-#{disk_id}.vdi")

          vb.customize [
            'storageattach', :id,
            '--storagectl', 'OSD Controller',
            '--port', 3 + disk_id,
            '--device', 0,
            '--type', 'hdd',
            '--medium', "disk-osd-#{vm_id}-#{disk_id}.vdi"
          ]
        end
      end

      # Provider Libvirt
      osd.vm.provider :libvirt do |lv|
        # Memory
        lv.memory = MON_MEMORY

        # OSD Disks
        driverletters = ('b'..'z').to_a
        (1..N_DISK_OSD).each do |disk_id|
	  lv.storage :file, :device => "vd#{driverletters[disk_id]}", :path => "disk-osd-#{vm_id}-#{disk_id}.disk", :size => '2G', :bus => "virtio"
        end
      end
    end
  end

  config.vm.provision "ansible" do |ansible|
	  ansible.sudo = true
	  ansible.playbook = "site.yml"
	  ansible.verbose = "v"
	  ansible.groups = {
            'mon' => (1..N_MON_HOSTS).map { |m| "mon#{m}"},
            'osd' => (1..N_OSD_HOSTS).map { |m| "osd#{m}"}
	  }
  end
end
