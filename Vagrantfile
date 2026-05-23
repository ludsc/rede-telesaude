# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  #configuração de todas as máquinas
  config.vm.box = "debian/bookworm64"
  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = 512
    vb.linked_clone = true
  end
#conf da do Node teleconsulta
  config.vm.define "teleconsulta" do |teleconsulta|
    teleconsulta.vm.hostname = "teleconsulta.node"
    teleconsulta.vm.network "private_network", ip: "192.168.56.30" 
  end
#conf da máquina telemetria de sinais vitais 
  config.vm.define "telemetria" do |telemetria|
    telemetria.vm.hostname = "telemetria.node"
    telemetria.vm.network "private_network", ip: "192.168.56.40" 
  end
#conf da máquina  transmissao de video
  config.vm.define "video" do |video|
    video.vm.hostname = "video.node"
    video.vm.network "private_network", ip: "192.168.56.50" 
  end
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "conf-geral.yml"
  end
end