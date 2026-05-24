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
      vb.memory = 2048 
      vb.cpus = 1 #algo
    end
  end

  # VM 2: Telemetria (Nó Worker - Broker MQTT)
  config.vm.define "telemetria" do |telemetria|
    telemetria.vm.hostname = "telemetria"
    telemetria.vm.network "private_network", ip: "192.168.56.40"
    
    telemetria.vm.provider "virtualbox" do |vb|
      vb.memory = 768  
      vb.cpus = 1
    end
  end

  # VM 3: Video (Nó Worker - Tráfego WebRTC)
  config.vm.define "video" do |video|
    video.vm.hostname = "video"
    video.vm.network "private_network", ip: "192.168.56.50"
    
    video.vm.provider "virtualbox" do |vb|
      vb.memory = 768 
      vb.cpus = 1
    end

# --- PROVISIONADORES SEQUENCIAIS NOMEADOS ---
    
    # Passo A: Configuração de pacotes comuns do SO
    video.vm.provision "config_geral", type: "ansible" do |ansible|
      ansible.playbook = "conf-geral.yml"
      ansible.inventory_path = "hosts.ini"
      ansible.limit = "all"    
    end

    # Passo B: Instalação e configuração do K3s (Só roda se o de cima passar)
    video.vm.provision "instalacao_kube", type: "ansible" do |ansible|
      ansible.inventory_path = "hosts.ini"
      ansible.playbook = "kube-up.yml"
      ansible.limit = "all" # Garantir que o limite englobe o inventário
    end
  end
end