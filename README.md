# Projeto Infraestrutura de Rede e Orquestração para Telemedicina (Telesaúde)

Este projeto apresenta o design, implementação e validação de uma infraestrutura de rede resiliente e orquestrada para suportar serviços críticos de **Telesaúde**. O ecossistema simula um ambiente hospitalar interconectado através de um cluster Kubernetes leve (**K3s**) distribuído em máquinas virtuais automatizadas com **Vagrant** e provisionadas via **Ansible**.

---

## 🎯 Objetivo do Projeto

O objetivo principal é prover uma rede de comunicação dedicada, isolada e de alta performance para garantir a operação contínua e segura de três aplicações pilares da saúde digital:

1. **Teleconsulta Clínica (`Nó: teleconsulta`):** Canal de comunicação HTTP/WebSocket para suporte a consultas médicas remotas e interações em tempo real.
2. **Telemetria de Sinais Vitais (`Nó: telemetria`):** Broker MQTT (`Eclipse Mosquitto`) responsável pelo recebimento contínuo de streams de dados críticos de IoT médica (ex: batimentos cardíacos, oximetria de leitos de UTI).
3. **Vídeo Cirúrgico (`Nó: video`):** Canal de streaming de mídia WebRTC de alta prioridade, simulando a transmissão de procedimentos cirúrgicos via pacotes UDP de baixa latência.

---

## 🏗️ Arquitetura da Infraestrutura

A topologia foi projetada utilizando uma abordagem de **Tráfego Direcionado**, onde regras de afinidade de nós (`nodeName`) garantem que cada microsserviço seja instanciado estritamente em seu respectivo nó de computação, simulando o isolamento de hardware de uma rede hospitalar real.

* **Hipervisor:** VirtualBox (Rede Host-Only: `192.168.56.0/24`)
* **Sistema Operacional Base:** Debian 12 (Bookworm)
* **Orquestrador:** K3s (Kubernetes v1.30+ com Containerd integrado)
* **Gerenciamento de Configuração:** ConfigMaps nativos para injeção de parâmetros do Broker MQTT.

| Serviço | Função Principal | Protocolo Base | Porta NodePort | IP de Destino da VM |
| :--- | :--- | :--- | :--- | :--- |
| **Teleconsulta** | Web / WebSockets | TCP | `30030` | `192.168.56.30` |
| **Telemetria** | Broker IoT Médico | TCP | `31883` | `192.168.56.40` |
| **Vídeo Cirúrgico** | Stream WebRTC | UDP | `31000` | `192.168.56.50` |

---

## 🧪 Validação Funcional e Análise de Tráfego

Para homologar a integridade da malha de dados, foram realizados testes práticos de conectividade a partir da máquina hospedeira (*Host*) direcionados aos endpoints do cluster.

### Captura de Processos via Wireshark
Toda a atividade de rede foi monitorada e capturada em tempo real utilizando o **Wireshark** atrelado à interface de rede virtual **`vboxnet0`** (gateway da rede privada das VMs). link para download: https://drive.google.com/drive/folders/1iD_iMlsnsY0lkMM9cdZg7zXW85n5DQdv?usp=sharing

* **Análise TCP (Telemetria):** O tráfego de dados simulados de monitoramento clínico foi validado através de payloads JSON transmitidos com sucesso sobre a camada de transporte estável do MQTT na porta `31883`.
* **Análise UDP (Vídeo):** Pacotes de dados brutos foram injetados na porta `31000` para validar a abertura de sockets e o comportamento de vazão e tempos de chegada (*delta time*) característicos de fluxos WebRTC.

Tanto o roteamento interno do plugin de rede (CNI Flannel) quanto os encaminhamentos de portas (*NodePorts*) responderam de forma nominal, garantindo o isolamento e entrega dos fluxos de dados das três aplicações.

---

## ⚠️ Limitações do Escopo e Diagnóstico de Hardware

Embora o planejamento inicial incluísse a execução de cenários de engenharia de caos e injeção de falhas de rede (como latência artificial e perda induzida de pacotes via *Chaos Mesh*), **os testes de estresse e degradação controlada de rede não puderam ser concluídos devido a limitações estritas de hardware da máquina hospedeira**.

### Diagnóstico Técnico do Ambiente Local:
* A máquina física possuía restrição de memória RAM (8 GB totais), o que exigiu um dimensionamento enxuto do cluster (alocando no máximo 2 GB para o nó Master e 768 MB para os Workers).
* A API do Kubernetes (`kube-apiserver`) e o runtime `containerd` operaram no limite do hardware disponível ao sustentar os serviços de saúde em paralelo com as ferramentas do plano de controle.
* A carga gerada pela inicialização dos esquemas complexos de validação de dados (*Custom Resource Definitions - CRDs*) e chamadas internas de validação por Webhooks do framework de testes provocou esgotamento de memória (*Out of Memory - OOM*) e quedas por *Timeout* na comunicação TLS do cluster.

**Conclusão:** Diante do teto operacional de hardware, priorizou-se garantir a **estabilidade, alta disponibilidade e acoplamento correto das três aplicações de telessaúde** com status `Running`, resguardando a integridade das capturas de pacotes realizadas no Wireshark.

---
## 🤖 Uso de Inteligência Artificial (IA) como Copiloto

Este projeto foi desenvolvido utilizando Inteligência Artificial (IA) de forma estratégica como uma ferramenta de engenharia assistida. A IA atuou como um parceiro de desenvolvimento (*pair programmer*) e consultor de infraestrutura, otimizando o ciclo de entrega e o diagnóstico de falhas de hardware e software.

### 1. Áreas de Atuação da IA no Projeto

* **Geração de Código (IaC e Manifestos):** Auxiliou na estruturação inicial e sintaxe das tarefas do Ansible Playbook, na definição da topologia multi-vm do `Vagrantfile` e no acoplamento correto dos recursos do Kubernetes (Deployments, Services e ConfigMaps).
* **Revisão e Refatoração:** Atuou ativamente na correção de bugs críticos de execução (como o travamento de inicialização do Broker MQTT Mosquitto), sugerindo a substituição de argumentos em linha de comando por injeção nativa de arquivos de configuração via `ConfigMap` e volumes.
* **Documentação:** Estruturação lógica e redação técnica deste arquivo `README.md`, garantindo clareza na apresentação da arquitetura e dos resultados para avaliadores técnicos.
* **Análise e Diagnóstico de Falhas:** Suporte analítico em tempo real para interpretar erros do sistema (como estouros de memória por falta de RAM no nó Master, falhas de handshake TLS do Kubernetes e problemas de concorrência com Webhooks de validação do Chaos Mesh).
* **Planejamento de Testes:** Formulação teórica de cenários de Engenharia de Caos utilizando arquivos de configuração `NetworkChaos` para prever os impactos de degradação de rede no tráfego UDP e TCP.

### 2. Orientação e Contexto Fornecidos à IA

A IA não foi utilizada para substituição de tomada de decisão técnica, mas sim orientada através de metodologias claras de engenharia de prompt:

* **Injeção de Contexto Real (Logs do Terminal):** A ferramenta foi alimentada continuamente com saídas reais de erro geradas pelo terminal das máquinas virtuais (`kubectl get pods`, logs de estouro de timeout e erros do Helm), permitindo diagnósticos precisos sobre o estado real do cluster.
* **Cenários de Restrição de Hardware:** A IA foi informada desde o início sobre as limitações físicas do ambiente (notebook com 8 GB de RAM), o que a orientou a sugerir soluções sob medida para o cenário (como redução de réplicas de gerenciadores, deleção de webhooks pesados e redimensionamento inteligente de memória entre os nós Master e Workers no Vagrant).
* **Definição de Requisitos de Negócio:** Foram fornecidos à IA os parâmetros exatos do problema (separação de tráfego de telessaúde por afinidade de nós, protocolo MQTT para telemetria e UDP para vídeo cirúrgico), garantindo que todo o código e cenários gerados estivessem estritamente alinhados com o escopo do projeto hospitalar.

## 🛠️ Como Executar o Projeto

1. Certifique-se de ter o **Vagrant**, **VirtualBox** e o **Ansible** instalados no seu host Linux.
2. Clone este repositório e acesse a pasta raiz:
   ```bash
   git clone https://github.com/ludsc/rede-telesaude.git
   cd rede-telesaude
   vagrant up
   vagrant ssh teleconsulta
   kubectl get nodes -o wide
   kubectl get pods -n saude -o wide
