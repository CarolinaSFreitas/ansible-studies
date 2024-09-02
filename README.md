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

# Anotações - Ansible for the Absolute Beginner - KodeKloud:

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

# Guia Rápido sobre Configuração e Inventário no Ansible

## Arquivo de Configuração do Ansible

Ao instalar o Ansible, um arquivo de configuração padrão é criado:

```plaintext
/etc/ansible/ansible.cfg
```

Esse arquivo contém duas principais seções:

- `[defaults]` (seção padrão no topo)
- Outras seções como `[inventory]`, `[privileges_escalation]`, `[paramiko_connection]`, `[ssh_connection]`, `[persistent_connection]` e `[colors]`

### Seção `[defaults]`

Contém opções principais, como:

```plaintext
inventory = /etc/ansible/hosts
log_path = /var/log/ansible.log
library = /usr/share/my_modules/
roles_path = /etc/ansible/roles
action_plugins = /usr/share/ansible/plugins/action
gathering = implicit
timeout = 10
forks = 5
```

### Seção `[inventory]`

Define plugins para o inventário, como:

```plaintext
enable_plugins = host_list, virtualbox, yaml, constructed
```

## Arquivos de Inventário

Os arquivos de inventário especificam os sistemas-alvo e podem estar em formatos diferentes, como INI ou YAML.

### Exemplo de Arquivo INI

```plaintext
[web]
server1.company.com ansible_connection=ssh ansible_user=root
server4.company.com ansible_connection=winrm

[db]
ansibleserver2.company.com ansible_connection=winrm ansible_user=admin

[mail]
server3.company.com ansible_connection=ssh ansible_ssh_pass=P@#

[localhost]
ansible_connection=localhost
```

### Exemplo de Arquivo YAML

```yaml
web:
  hosts:
    server1.company.com:
      ansible_connection: ssh
      ansible_user: root
    server4.company.com:
      ansible_connection: winrm

db:
  hosts:
    ansibleserver2.company.com:
      ansible_connection: winrm
      ansible_user: admin

mail:
  hosts:
    server3.company.com:
      ansible_connection: ssh
      ansible_ssh_pass: P@#

localhost:
  hosts:
    localhost:
      ansible_connection: localhost
```

## Agrupamento e Relações Pai e Filho

### Agrupamento

Facilita a atualização de servidores por categoria, como departamento ou localização.

### Relações Pai e Filho

Permite definir configurações específicas para grupos e subgrupos de servidores.

```plaintext
[web]
server1
server2

[web:children]
server3
```

## Variáveis

Variáveis são usadas para armazenar informações dinâmicas.

### Tipos de Variáveis

- **String Variables**: Sequências de caracteres.
- **Number Variables**: Números inteiros e de ponto flutuante.
- **Boolean Variables**: Valores verdadeiro ou falso.
- **List Variables**: Sequências ordenadas de valores.
- **Dictionary Variables**: Pares chave-valor.

### Definindo e Usando Variáveis

- **Playbooks**: Variáveis podem ser definidas diretamente em playbooks ou arquivos separados.
- **Variáveis de Registro**: Usadas para armazenar resultados de comandos e tarefas.

```yaml
- name: Example Task
  command: /bin/echo "Hello World"
  register: result
```

### Escopo de Variáveis

- **Host Scope**: Variáveis definidas para um host específico.
- **Play Scope**: Variáveis definidas apenas para uma play específica.
- **Global Scope**: Variáveis acessíveis em qualquer lugar do playbook, como via CLI.

## Variáveis Mágicas

São variáveis especiais que fornecem informações sobre o ambiente ou hosts.

- **`hostvars`**: Acessa variáveis de outros hosts.
- **`groups`**: Retorna todos os hosts de um grupo.
- **`group_names`**: Retorna os nomes dos grupos aos quais o host pertence.
- **`inventory_hostname`**: Nome do host conforme definido no arquivo de inventário.

### Exemplo de Uso

```yaml
- name: Print DNS of web2
  debug:
    msg: "{{ hostvars['web2']['ansible_host'] }}"
```

Para mais informações, consulte a [documentação oficial do Ansible](https://docs.ansible.com/ansible/latest/index.html).

# Entendendo `ansible_facts` no Ansible

## O que são `ansible_facts`?

`ansible_facts` são um conjunto de informações coletadas automaticamente pelo Ansible sobre o sistema gerenciado. Essas informações incluem detalhes sobre a configuração do sistema, hardware, rede, e muito mais.

## Como os `ansible_facts` São Coletados?

Durante a execução de um playbook, o Ansible coleta essas informações como parte do processo de **gathering facts**. O gathering facts é a tarefa que reúne dados sobre o sistema antes de executar outras tarefas.

## Exemplo de Uso

### Playbook com Gathering Facts Ativado

````
- name: Print hello message
  hosts: all
  tasks:
    - debug:
        msg: "{{ ansible_facts }}"
````

### Saída Exemplo:
````
ok: [web1] => {
  "ansible_facts": {
    "all_ipv4_addresses": ["172.20.1.100"],
    "architecture": "x86_64",
    "date_time": {"date": "2019-09-07"},
    "distribution": "Ubuntu",
    "fqdn": "web1",
    "hostname": "web1",
    "interfaces": ["lo", "eth0"],
    "machine": "x86_64",
    "memfree_mb": 72,
    "memory_mb": {"real": {"free": 72, "total": 985, "used": 913}},
    "memtotal_mb": 985,
    "processor": ["0", "GenuineIntel", "Intel(R) Core(TM) i9-9980HK CPU @ 2.40GHz"],
    "product_name": "VirtualBox",
    "product_serial": "0",
    "product_uuid": "18A31B5D-FAC9-445F-9B6F-95B4B587F485"
  }
}
````
### Desativando Gathering Facts
Você pode desativar o gathering facts se não precisar dessas informações, usando:

````
- name: Print hello message
  hosts: all
  gather_facts: no
  tasks:
    - debug:
        msg: "{{ ansible_facts }}"
````
      
Saída Exemplo com Gathering Facts Desativado:

````
ok: [web1] => {
  "ansible_facts": {}
}
````
Configuração de Gathering Facts
A configuração do comportamento de gathering facts pode ser ajustada no arquivo de configuração do Ansible (ansible.cfg):

````
[defaults]
gathering = implicit
````

implicit: Fatos são coletados por padrão.
explicit: Fatos não são coletados por padrão; deve-se definir gather_facts: True para coletá-los.
Conclusão
ansible_facts fornecem uma visão detalhada do sistema gerenciado e podem ser muito úteis para criar playbooks dinâmicos e adaptáveis. A coleta automática de fatos ajuda a personalizar e otimizar suas configurações e tarefas.

Para mais detalhes, consulte a documentação oficial do Ansible sobre Gathering Facts.
