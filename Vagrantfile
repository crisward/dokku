# -*- mode: ruby -*-
# vi: set ft=ruby :

BOX_NAME = ENV["BOX_NAME"] || "bento/ubuntu-14.04"
BOX_MEMORY = ENV["BOX_MEMORY"] || "4096"
DOKKU_DOMAIN = ENV["DOKKU_DOMAIN"] || "dokku.me"
DOKKU_IP = ENV["DOKKU_IP"] || "10.0.0.2"
FORWARDED_PORT = (ENV["FORWARDED_PORT"] || '8080').to_i
PREBUILT_STACK_URL = File.exist?("#{File.dirname(__FILE__)}/stack.tgz") ? 'file:///root/dokku/stack.tgz' : nil
PUBLIC_KEY_PATH = "#{Dir.home}/.ssh/id_rsa.pub"

make_cmd = "DEBIAN_FRONTEND=noninteractive make -e install"
if PREBUILT_STACK_URL
  make_cmd = "PREBUILT_STACK_URL='#{PREBUILT_STACK_URL}' #{make_cmd}"
end

Vagrant::configure("2") do |config|
  config.ssh.forward_agent = true

  config.vm.box = BOX_NAME

  #websites
  config.vm.synced_folder "/websites", "/websites-nfs",
  type: "nfs",
  mount_options:['actimeo=2']
  config.bindfs.bind_folder "/websites-nfs", "/websites",
  perms: "u=rwx:g=rwx:o=rwx",
  o:"nonempty",
  chown_ignore: true,
  chgrp_ignore: true

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    # Ubuntu's Raring 64-bit cloud image is set to a 32-bit Ubuntu OS type by
    # default in Virtualbox and thus will not boot. Manually override that.
    vb.customize ["modifyvm", :id, "--ostype", "Ubuntu_64"]
    vb.customize ["modifyvm", :id, "--memory", BOX_MEMORY]
  end

  config.vm.provider :vmware_fusion do |v, override|
    v.vmx["memsize"] = BOX_MEMORY
  end

  config.vm.define "empty", autostart: false

  config.vm.define "dokku", primary: true do |vm|
    vm.vm.synced_folder File.dirname(__FILE__), "/root/dokku"
    vm.vm.network :forwarded_port, guest: 80, host: FORWARDED_PORT
    vm.vm.hostname = "#{DOKKU_DOMAIN}"
    vm.vm.network :private_network, ip: DOKKU_IP
    vm.vm.network "public_network", bridge: 'en0: Ethernet',ip: "192.168.1.22"
    vm.vm.provision :shell, :inline => "export DEBIAN_FRONTEND=noninteractive && apt-get update > /dev/null && apt-get -qq -y install git > /dev/null && cd /root/dokku && #{make_cmd}"
    vm.vm.provision :shell, :inline => "cd /root/dokku && make dokku-installer"
    vm.vm.provision :shell do |s|
      s.inline = <<-EOT
        echo '"\e[5~": history-search-backward' > /root/.inputrc
        echo '"\e[6~": history-search-forward' >> /root/.inputrc
        echo 'set show-all-if-ambiguous on' >> /root/.inputrc
        echo 'set completion-ignore-case on' >> /root/.inputrc
      EOT
    end
  end


  config.vm.define "dokku-deb", autostart: false do |vm|
    vm.vm.synced_folder File.dirname(__FILE__), "/root/dokku"
    vm.vm.network :forwarded_port, guest: 80, host: FORWARDED_PORT
    vm.vm.hostname = "#{DOKKU_DOMAIN}"
    vm.vm.network :private_network, ip: DOKKU_IP
    vm.vm.network "public_network", bridge: 'en0: Ethernet',ip: "192.168.1.22"
    vm.vm.provision :shell, :inline => "cd /root/dokku && make install-from-deb"
  end

  config.vm.define "build", autostart: false do |vm|
    vm.vm.synced_folder File.dirname(__FILE__), "/root/dokku"
    vm.vm.network :forwarded_port, guest: 80, host: FORWARDED_PORT
    vm.vm.hostname = "#{DOKKU_DOMAIN}"
    vm.vm.network :private_network, ip: DOKKU_IP
    vm.vm.network "public_network", bridge: 'en0: Ethernet',ip: "192.168.1.22"
    vm.vm.provision :shell, :inline => "export DEBIAN_FRONTEND=noninteractive && apt-get update > /dev/null && apt-get -qq -y install git > /dev/null && cd /root/dokku && #{make_cmd}"
    vm.vm.provision :shell, :inline => "cd /root/dokku && make deb-all"
  end

  if Pathname.new(PUBLIC_KEY_PATH).exist?
    config.vm.provision :file, source: PUBLIC_KEY_PATH, destination: '/tmp/id_rsa.pub'
    config.vm.provision :shell, :inline => "rm -f /root/.ssh/authorized_keys && mkdir -p /root/.ssh && sudo cp /tmp/id_rsa.pub /root/.ssh/authorized_keys"
  end
end
