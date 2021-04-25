


# Chapter 1: Introdução ao Ansible

[student@workstation ~]$ sudo yum install ansible
[student@workstation ~]$ ansible --version
[student@workstation ~]$ ansible -m setup localhost | grep ansible_python_version

# Chapter 2: Implantação do Ansible

```
[usa]
washington[1:2].example.com

[canada]
ontario[01:02].example.com

[north-america:children]
canada
usa

---

[user@controlnode ~]$ ansible washington1.example.com --list-hosts
  hosts (1):
    washington1.example.com

[user@controlnode ~]$ ansible canada --list-hosts
  hosts (2):
    ontario01.example.com
    ontario02.example.com

[user@controlnode ~]$ ansible localhost --list-hosts
```

### Configuração do Ansible

- Uso do /etc/ansible/ansible.cfg (MENOR PRIORIDADE)
- Uso do ~/.ansible.cfg no home do usuário
- Uso do ./ansible.cfg no diretório atual da execução 
- Uso da variável de ambiente ANSIBLE_CONFIG (MAIOR PRIORIDADE)

- verificar
```
[user@controlnode ~]$ ansible --version
[user@controlnode ~]$ ansible servers --list-hosts -v
```

- exemplo
```
[defaults]
inventory = ./inventory
remote_user = user
ask_pass = false

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false
```

### Configurações de conexão 

```
[user@controlnode ~]$ ssh-copy-id root@web1.example.com

ou

- name: Public key is deployed to managed hosts for Ansible
  hosts: all

  tasks:
    - name: Ensure key is in root's ~/.ssh/authorized_hosts
      authorized_key:
        user: root
        state: present
        key: '{{ item }}'
      with_file:
        - ~/.ssh/id_rsa.pub
```

### Conexões não SSH

```
[user@controlnode ~]$ ansible localhost --list-hosts
[WARNING]: provided hosts list is empty, only localhost is available

  hosts (1):
    localhost
```

- ignora a definição remote_user e executa comandos diretamente no sistema local. 
-  Se o escalonamento de privilégios estiver sendo usado, ele executa o sudo da conta de usuário que executou o comando do Ansible, não o remote_user.
-  Se você quiser garantir que você se conecta ao localhost usando o SSH como outros hosts gerenciados, uma abordagem é listá-lo em seu inventário. Mas isso o inclui nos grupos all e ungrouped, o que talvez não seja recomendável. 
- crie um subdiretório host_vars , crie um arquivo denominado localhost, contendo a linha ansible_connection: smart. Isso garante que o protocolo de conexão (SSH) smart seja usado em vez do local para localhost. 
- colocar todos esses hosts gerenciados no grupo windows e, depois, criar um arquivo denominado group_vars/windows contendo as seguintes linhas:
```
ansible_connection: winrm
ansible_port: 5986
```

### Comentários de arquivos de configuração

-  Há dois caracteres de comentário permitidos pelos arquivos de configuração do Ansible: a cerquilha, ou jogo da velha (#), e o ponto-e-vírgula (;).

- O caractere de jogo da velha no início de uma linha faz com que toda a linha seja interpretada como um comentário. Ele não deve estar na mesma linha que uma diretiva.

- O caractere de ponto-e-vírgula faz com que tudo que esteja à direita dele na linha seja interpretado como comentário. Ele pode estar na mesma linha que uma diretiva, desde que essa diretiva esteja à sua esquerda

### Execução de comandos ad hoc

```
[user@controlnode ~]$ ansible host-pattern -m module [-a 'module arguments'] [-i inventory]
[user@controlnode ~]$ ansible all -m ping
[user@controlnode ~]$ ansible -m user -a 'name=newbie uid=4000 state=present' servera.lab.example.com
```

### O comando ansible-doc 

- O comando ansible-doc -l lista todos os módulos que estão instalados no sistema

```
[user@controlnode ~]$ ansible-doc ping
```

### Comandos arbitrários

```
[user@controlnode ~]$ ansible mymanagedhosts -m command -a /usr/bin/hostname
[user@controlnode ~]$ ansible mymanagedhosts -m command -a /usr/bin/hostname -o (LINHA ÚNICA)
```
### Comandos arbitrários módulo shell

- as variáveis do ambiente de shell estão disponíveis, e as operações de shell, como redirecionamento e pipes, também estão disponíveis para uso

```
[user@controlnode ~]$ ansible localhost -m command -a set
localhost | FAILED | rc=2 >>
[Errno 2] No such file or directory
[user@controlnode ~]$ ansible localhost -m shell -a set
localhost | CHANGED | rc=0 >>
BASH=/bin/sh
BASHOPTS=cmdhist:extquote:force_fignore:hostcomplete:interact
ive_comments:progcomp:promptvars:sourcepath
BASH_ALIASES=()
...output omitted...
```

### Comandos arbitrários módulo raw

- Os módulos command e shell precisam ter uma instalação do Python em funcionamento no host gerenciado. Um terceiro módulo, raw, pode executar comandos diretamente no shell remoto, ignorando o subsistema do módulo. Isso é útil ao gerenciar sistemas que não podem ter o Python instalado (por exemplo, um roteador de rede). Também pode ser usado para instalar o Python em um host. 

### Configuração de conexões de comandos ad hoc

```
inventory 	        -i
remote_user 	    -u
become 	            --become, -b
become_method 	    --become-method
become_user 	    --become-user
become_ask_pass 	--ask-become-pass, -K 

[user@controlnode ~]$ ansible --help
```

### Exercício

```
[student@workstation deploy-adhoc]$ ansible localhost -m command -a 'id'
[student@workstation deploy-adhoc]$ ansible localhost -m command -a 'id' -u devops
[student@workstation deploy-adhoc]$ ansible localhost -m copy -a 'content="Managed by Ansible\n" dest=/etc/motd' -u devops (FALHA)
[student@workstation deploy-adhoc]$ ansible localhost -m copy -a 'content="Managed by Ansible\n" dest=/etc/motd' -u devops --become
[student@workstation deploy-adhoc]$ ansible all -m copy -a 'content="Managed by Ansible\n" dest=/etc/motd' -u devops --become
[student@workstation deploy-adhoc]$ ansible all -m command -a 'cat /etc/motd' -u devops
```

### Laboratório Aberto: Implantação do Ansible

```
[student@workstation ~]$ yum install ansible
[student@workstation ~]$ yum list installed ansible
[student@workstation ~]$ ansible --version
[student@workstation ~]$ vi ansible.cfg

[defaults]
remote_user = devops
inventory = inventory

[privilege_escalation]
become = False
become_method = sudo
become_user = root
become_ask_pass = False

[student@workstation deploy-review]$ ansible all -m command -a 'id'
[student@workstation deploy-review]$ ansible all -m copy -a 'content="This server is managed by Ansible.\n" dest=/etc/motd' --become
[student@workstation deploy-review]$ ansible all -m command -a 'cat /etc/motd'
```

# Chapter 3: Implementação de playbooks



```
f2727208@Workstation:~$ echo "autocmd FileType yaml setlocal ai ts=2 sw=2 et" >> $HOME/.vimrc
```
- Exemplo playbook

```
---
  name: just an example
  hosts: webservers
  tasks:
    - name: web server is enabled
      service:
        name: httpd
        enabled: true

    - name: NTP server is enabled
      service:
        name: chronyd
        enabled: true

```

- Checar sintaxe 
```
[student@workstation ~]$ ansible-playbook --syntax-check webserver.yml
```

- Dry run

```
[student@workstation ~]$ ansible-playbook -C webserver.yml
```

- Run playbooks run

```
[student@workstation ~]$ ansible-playbook webserver.yml
```

- Multiple

```
---
# This is a simple playbook with two plays

- name: first play
  hosts: web.example.com
  tasks:
    - name: first task
      yum:
        name: httpd
        status: present

    - name: second task
      service:
        name: httpd
        enabled: true

- name: second play
  hosts: database.example.com
  tasks:
    - name: first task
      service:
        name: mariadb
        enabled: true
```

- Remote Users and Privilege Escalation in Plays

```
---
- name: /etc/hosts is up to date
  hosts: datacenter-west
  remote_user: automation
  become: yes
  become_method: sudo
  cecome_user: root

  tasks:
    - name: server.example.com in /etc/hosts
      lineinfile:
        path: /etc/hosts
        line: '192.0.2.42 server.example.com server'
        state: present
```

- Finding Modules for Tasks

```
[student@workstation modules]$ ansible-doc -l
[student@workstation modules]$ ansible-doc yum
[student@workstation ~]$ ansible-doc -s yum   (EXEMPLO)
```

- YAML Strings
```
this is a string

'this is another string'

"this is yet another a string"
```


- YAML Strings new lines
```
include_newlines: |
        Example Company
        123 Main Street
        Atlanta, GA 30303
```


- YAML Strings single line
```
fold_newlines: >
        This is an example
        of a long string,
        that will become
        a single sentence once folded.
```


- YAML Dictionaries
```
 You have seen collections of key-value pairs written as an indented block, as follows:

  name: svcrole
  svcservice: httpd
  svcport: 80

Dictionaries can also be written in an inline block format enclosed in curly braces, as follows:

  {name: svcrole, svcservice: httpd, svcport: 80}
  
```

- YAML Lists
```
 You have also seen lists written with the normal single-dash syntax:

  hosts:
    - servera
    - serverb
    - serverc

Lists also have an inline format enclosed in square braces, as follows:

hosts: [servera, serverb, serverc]
```

- YAML Dictionaries
```
```

- Obsolete key=value Playbook Shorthand

```
- shorthand form (EVITAR)

tasks:
    - name: shorthand form
      service: name=httpd enabled=true state=started

- Normally you would write the same task as follows: 

  tasks:
    - name: normal form
      service:
        name: httpd
        enabled: true
        state: started
```

- Lab: Implementing Playbooks



# Chapter 4: Gerenciamento de variáveis e fatos

- Scopo de variáveis
  -  Global scope: Variables set from the command line or Ansible configuration (MAIOR PRECEDêNCIA)
  -  Play scope: Variables set in the play and related structures 
  -  Host scope: Variables set on host groups and individual hosts by the inventory, fact gathering, or registered tasks (MENOR PRECEDÊNCIA)


- Variables in playbooks
```
- hosts: all
  vars:
    user: joe
    home: /home/joe

or 

- hosts: all
  vars_files:
    - vars/users.yml

The playbook variables are then defined in that file or those files in YAML format:

user: joe
home: /home/joe
```

- Using variables in a play

```
vars:
  user: joe

tasks:
  # This line will read: Creates the user joe
  - name: Creates the user {{ user }}
    user:
      # This line will create the user named Joe
      name: "{{ user }}"

When a variable is used as the first element to start a value, quotes are mandatory. 

yum:
     name: {{ service }}
            ^ here

Should be written as:

yum:
     name: "{{ service }}"
          
```

- Host Variables and Group Variables

```
[servers]
demo1.example.com
demo2.example.com

[servers:vars]
user=joe
```

- Using Directories to Populate Host and Group Variables (MELHOR)

```
[admin@station project]$ cat ~/project/inventory
[datacenter1]
demo1.example.com
demo2.example.com

[datacenter2]
demo3.example.com
demo4.example.com

[datacenters:children]
datacenter1
datacenter2

[admin@station project]$ cat ~/project/group_vars/datacenters
package: httpd

[admin@station project]$ cat ~/project/group_vars/datacenter1
package: httpd

[admin@station project]$ cat ~/project/group_vars/datacenter2
package: apache

[admin@station project]$ cat ~/project/host_vars/demo1.example.com
package: httpd
[admin@station project]$ cat ~/project/host_vars/demo2.example.com
package: apache
[admin@station project]$ cat ~/project/host_vars/demo3.example.com
package: mariadb-server
[admin@station project]$ cat ~/project/host_vars/demo4.example.com
package: mysql-server

```

- Overriding Variables from the Command Line

```
[user@demo ~]$ ansible-playbook main.yml -e "package=apache"
```

- Using Arrays as Variables

```
users:
  bjones:
    first_name: Bob
    last_name: Jones
    home_dir: /users/bjones
  acook:
    first_name: Anne
    last_name: Cook
    home_dir: /users/acook
    
You can then use the following variables to access user data: 

# Returns 'Bob'
users.bjones.first_name

# Returns '/users/acook'
users.acook.home_dir

 Because the variable is defined as a Python dictionary, an alternative syntax is available.

# Returns 'Bob'
users['bjones']['first_name']

# Returns '/users/acook'
users['acook']['home_dir']
```

- Capturing Command Output with Registered Variables

```
---
- name: Installs a package and prints the result
  hosts: all
  tasks:
    - name: Install the package
      yum:
        name: httpd
        state: installed
      register: install_result

    - debug: var=install_result
```


- Ansible vault

```

```






# Chapter 5: Implementação de controle de tarefas

```

```

# Chapter 6: Implantação de arquivos em hosts gerenciados

```

```

# Chapter 7: Gerenciamento de projetos grandes

```

```

# Chapter 8: Simplificação de playbooks com funções

```

```

# Chapter 9: Solução de problemas do Ansible

```

```

# Chapter 10: Automatização de tarefas de administração do Linux

```

```

# Chapter 11: Análise abrangente: Automation with Ansible

```

```

# Appendix A: Tópicos complementares

# Section A.1: Investigação das opções de configuração do Ansible

# Appendix B: Licenciamento do Ansible Lightbulb

# Section B.1: Licença do Ansible Lightbulb