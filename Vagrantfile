# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  
  # Configurações globais padrão do VirtualBox
  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.linked_clone = true
  end

  # VM 1: Teleconsulta (Nó Master - Control Plane)
  config.vm.define "teleconsulta" do |teleconsulta|
    teleconsulta.vm.hostname = "teleconsulta"
    teleconsulta.vm.network "private_network", ip: "192.168.56.30"
    
    teleconsulta.vm.provider "virtualbox" do |vb|
      vb.memory = 2048 # Aumentado para 2GB para aguentar o Chaos Mesh
      vb.cpus = 1
    end
  end

  # VM 2: Telemetria (Nó Worker - Broker MQTT)
  config.vm.define "telemetria" do |telemetria|
    telemetria.vm.hostname = "telemetria"
    telemetria.vm.network "private_network", ip: "192.168.56.40"
    
    telemetria.vm.provider "virtualbox" do |vb|
      vb.memory = 768  # Mantido enxuto
      vb.cpus = 1
    end
  end

  # VM 3: Video (Nó Worker - Tráfego WebRTC)
  config.vm.define "video" do |video|
    video.vm.hostname = "video"
    video.vm.network "private_network", ip: "192.168.56.50"
    
    video.vm.provider "virtualbox" do |vb|
      vb.memory = 768 # Reduzido ligeiramente para equilibrar o host, agnhost é leve
      vb.cpus = 1
    end
  end
  # --- PROVISIONADOR GLOBAL ---
  # Colocado fora dos blocos individuais, ele só roda depois que TODAS as VMs acima estiverem 100% ativas.
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "conf-geral.yml"
    ansible.inventory_path = "hosts.ini"
    ansible.limit = "all"    
  end
  config.vm.provision "ansible" do |ansible|
    ansible.inventory_path = "hosts.ini"
    ansible.playbook = "kube-up.yml"
  end
end