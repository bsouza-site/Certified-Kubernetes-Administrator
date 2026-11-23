# Instalação do Kubeadm.

## Infraestrutura do cluster Kubernetes.  
Iremos preparar 3 VMs com sistema operacional Debian e/ou Ubuntu e nelas iremos instalar e configurar todos os componentes da estrutura do Cluster Kubernetes.  
<div align="center"><img width="845" height="403" alt="image" src="https://github.com/user-attachments/assets/b00014dd-b680-4dc6-aa39-2ea797d02869" /></div>

**Entendendo os componentes de uma instalação do cluster Kubernetes.**  
Antes de iniciar a instalação de um cluster do Kubernetes, devemos entender todos os seus componentes.  
<br>

**KUBEADM.**  
O **[kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)** é uma ferramenta oficial do Kubernetes criada para automatizar a infraestrutura de criação de um cluster. Pense nele como o "instalador manual por excelência": ele cuida de toda a parte chata e complexa de configurar os componentes principais (como o painel de controle, certificados de segurança e tokens de autenticação) para que o cluster siga as melhores práticas de segurança.  

Enquanto outras ferramentas criam clusters inteiros na nuvem com um clique (como o Terraform ou instâncias gerenciadas tipo EKS/GKE), o kubeadm é focado em configurar o software do Kubernetes em máquinas que você já preparou (sejam VMs locais, servidores físicos ou instâncias na nuvem).  

**O que o kubeadm faz na prática?**  
O fluxo de trabalho principal do kubeadm se resume a dois comandos mágicos:  

**1. kubeadm init**: executado na máquina que será o nó do painel de controle (Control Plane ou Master). Ele inicializa o banco de dados do cluster (etcd), gera os certificados SSL e inicia os componentes essenciais (API Server, Controller Manager, Scheduler).  

**2. kubeadm join**: executado nos nós de trabalho (workers). Esse comando usa um token gerado pelo init para conectar à máquina com segurança ao nó principal, expandindo a capacidade do seu cluster.  
<br>

**KUBELET.**  
Se o **[kubeadm](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)** é o instalador que monta o cluster, o kubelet é o "gerente local" ou o "capataz" de cada máquina (nó) dentro do Kubernetes.  

O kubelet é um agente que roda em todos os nós do cluster (tanto no Master/Control Plane quanto nos workers). A principal função dele é garantir que os containers que deveriam estar rodando naquela máquina específica estejam, de fato, vivos, saudáveis e operando conforme o esperado.  

**O que o kubelet faz na prática?**  
O funcionamento do kubelet é baseado em um ciclo constante de checagem. Ele não toma grandes decisões sozinho; ele apenas obedece a ordens que vêm de cima (do painel de controle).  

* **Recebe as ordens (PodSpec)**: o ponto central do Kubernetes (o API Server) envia para o kubelet uma lista de diretrizes chamada **PodSpec** (um arquivo YAML/JSON que descreve como o Pod deve ser). O kubelet olha e diz: "Ok, preciso rodar 2 containers da imagem X nessa minha máquina".  

* **Conversa com o Container Runtime**: o kubelet não sabe criar containers sozinho. Ele pega essa ordem e aciona o Container Runtime da máquina (como o containerd ou CRI-O) através de uma interface chamada CRI (Container Runtime Interface) para que ele baixe a imagem e suba os containers.  

* **Monitora a saúde (probes)**: ele fica vigiando os containers locais. Se você configurou um teste de vida (Liveness Probe) e o container travar, o kubelet percebe e reinicia aquele container imediatamente.  

* **Reporta o status de volta**: periodicamente, ele envia relatórios para o API Server dizendo: "olha, a máquina está com X% de memória livre e os containers que você me pediu estão rodando perfeitamente".  

**Uma analogia simples:**  
Pense no Kubernetes como uma grande rede de franquias de fast-food:  

* O Control Plane (Master) é a diretoria da empresa. Ela decide o cardápio e qual loja vai atender qual pedido.  

* O kubelet é o gerente local de uma filial específica. Ele recebe o pedido da diretoria, grita para os cozinheiros (Container Runtime) prepararem o sanduíche (os containers) e garante que a cozinha não esteja pegando fogo. Se um cozinheiro faltar ou um ingrediente estragar, ele avisa a diretoria.  

**O que ele NÃO faz?**  
* Ele não gerencia o cluster global: o kubelet só enxerga a própria máquina onde está instalado. Ele não sabe e não se importa com o que está acontecendo nos outros nós do vizinho.  

* Ele não faz o agendamento (**scheduling**): ele não decide onde um Pod vai rodar. Quem decide isso é o kube-scheduler no nó Master. O kubelet apenas aceita o Pod depois que o scheduler diz: "Esse vai para a sua máquina".  

Existem duas formas de visualizar, de forma básica, informações do kubelet.

* **Verificar a execução do serviço**: `systemctl status kubelet`  
* **Verificar os logs**: `journalctl -u kubelet`  



