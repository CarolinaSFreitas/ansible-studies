# Ansible Studies
Repositório para minhas anotações de estudo do Ansible

[Ansible Documentation](https://docs.ansible.com/ansible/latest/index.html)

Ansible is an agentless automation tool that you install on a single host (referred to as the control node).

From the control node, Ansible can manage an entire fleet of machines and other devices (referred to as managed nodes) remotely with SSH, Powershell remoting, and numerous other transports, all from a simple command-line interface with no databases or daemons required.

Ansible oferece várias vantagens significativas para a automação de TI. É uma ferramenta agentless, o que significa que não requer a instalação de software adicional nos servidores gerenciados, simplificando a configuração e manutenção. Utiliza uma linguagem de configuração baseada em YAML, tornando seus playbooks fáceis de escrever e entender. Ansible também é altamente escalável e permite a automação de tarefas repetitivas, configuração de sistemas, implantação de aplicativos e orquestração de serviços complexos de maneira eficiente e idempotente, garantindo que as ações sejam aplicadas somente quando necessário. Além disso, sua flexibilidade e integração com diversas tecnologias e plataformas o tornam uma escolha popular para profissionais de DevOps e administradores de sistemas.

## Instalação:
CentOS/RH: 
1. ``yum install epel-release`` 
2. ``yum install ansible``

tip: "ansible" no terminal traz o menu de ajuda do Ansible

# Anotações:

## Componentes Essenciais do Ansible
No "/etc/ansible/" encontram-se os arquivos:
- ansible.cfg: configurações novas sobescrevem as configurações padrões
- hosts: arquivo de inventário, onde se declara quais máquinas serão controladas 
- roles: estrutura para organizar tarefas, handlers, arquivos, templates, variáveis e outras informações para modularizar e reutilizar configurações

## Criando grupo no "hosts" para armazenar servidores

tip: pode usar domínio, ip

Criando esse grupo o Ansible pode se logar nos Servidores:

````
[servidores-distros]               //nome do grupo

172.16.34.78
172.20.58.194

[servidores-distros:vars]      
ansible_user=root                //fora do laboratório, nao utilizar o user root, pode ser um chamado 'ansible'
ansible_password=minhasenha123   //fora do laboratório, nao utilizar a senha assim via ssh e sim uma chave
````

Como é um lab o exemplo acima, deve-se descomentar, dentro do 'ansible.cfg', a linha "host_key_checking" porque o Ansible sempre busca pelas chaves para acessar via SSH. 

## Checar conexão do Ansible com as máquinas

``ansible servidores-distros -m ping``

## Alterações em múltiplos servidores com Ansible - Via Playbook - 1
A capacidade de gerenciar e alterar configurações em múltiplos servidores simultaneamente é uma das principais vantagens do Ansible, tornando-o uma ferramenta poderosa para a automação de TI e gerenciamento de configurações.

#### Exemplo
Vamos supor que você queira instalar o Apache em 5 servidores. 

Inventário (inventory):
````
[servidores-distros]
server1.example.com
server2.example.com
server3.example.com
server4.example.com
server5.example.com
````

Playbook (playbook.yml):
````
- name: Install and configure Apache on multiple servers    //explicação da jogada
  hosts: servidores-distros     //nome do inventário de hosts
  become: yes           //linha para indicar que as tarefas vão ser executadas como root
  tasks:
    - name: Install Apache
       package:      //módulo "package" é p/ qqr distro, o ansible vê a distro e faz o ger. de pac. rodar (apt, yum)
        name: apache2
        state: present     
                    //"state" pode ser present, absent ou latest. p: instalado no servidor, a: não quer no     servidor, l: instalar a última versão disponível

    - name: Ensure Apache is running
      service:
        name: apache2
        state: started
````

nota: as tarefas num Playbook são chamadas de Play/Jogada

Comando para Executar o Playbook: ``ansible-playbook -i inventory playbook.yml`` 

**Vantagens**
Eficiência: Com um único comando, você pode aplicar a mesma configuração a vários servidores simultaneamente.
Consistência: Garante que todos os servidores estejam configurados de maneira idêntica, reduzindo o risco de erros de configuração.
Escalabilidade: Facilmente escalável para gerenciar dezenas, centenas ou milhares de servidores.
Automação: Reduz a necessidade de intervenções manuais repetitivas, economizando tempo e esforço.

tip:
Em distros diferentes, pode ser melhor separar os grupos de distribuições por tipo para manutenções ou criar inventários diferentes com SOs diferentes. 

## Alterações em múltiplos servidores com Ansible - 2

Atualizando o inventário 'servidores-distros' via CLI:
``ansible servidores-distros -a "apt-get update"``

## Informações dos hosts:
``ansible servidores-distros -a "uptime"``

## Ansible - EC2 

