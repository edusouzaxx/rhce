


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
[student@demo ~]$ cat secret.yml
my_secret: "yJJvPqhsiusmmPPZdnjndkdnYNDjdj782meUZcw"

[student@demo ~]$ ansible-vault create secret.yml
New Vault password: redhat
Confirm New Vault password: redhat

[student@demo ~]$ ansible-vault view secret.yml

[student@demo ~]$ ansible-vault edit secret.yml

[student@demo ~]$ ansible-vault encrypt secret1.yml (ARQUIVO EXISTENTE)

[student@demo ~]$ ansible-vault rekey secret.yml (TROCAR SENHA)
```

- Ansible-vault em playbooks
```
[student@demo ~]$ ansible-playbook site.yml
ERROR: A vault password must be specified to decrypt vars/api_key.yml

[student@demo ~]$ ansible-playbook --vault-id @prompt site.yml
Vault password (default): redhat

[student@demo ~]$ ansible-playbook --ask-vault-pass site.yml
Vault password: redhat
```


- Estrutura com vault

```
.
├── ansible.cfg
├── group_vars
│   └── webservers
│       └── vars
├── host_vars
│   └── demo.example.com
│       ├── vars
│       └── vault
├── inventory
└── playbook.yml

 Then the administrator can use ansible-vault to encrypt the vault file, while leaving the vars file as plain text. 

```

-  Sensitive playbook variables can be placed in a separate file which is encrypted with Ansible Vault and which is included in the playbook through a vars_files directive.


- Exercício


```
[student@workstation data-secret]$ ansible-vault edit secret.yml
Vault password: redhat

username: ansibleuser1
pwhash: $6$jf...uxhP1

---
- name: create user accounts for all our servers
  hosts: devservers
  become: True
  remote_user: devops
  vars_files:
    - secret.yml
  tasks:
    - name: Creating user from secret.yml
      user:
        name: "{{ username }}"
        password: "{{ pwhash }}"

[student@workstation data-secret]$ ansible-playbook --syntax-check --ask-vault-pass create_users.yml
Vault password (default): redhat

or

[student@workstation data-secret]$ ansible-playbook --syntax-check --vault-id @prompt create_users.yml
Vault password (default): redhat

- Utilizando arquivo

[student@workstation data-secret]$ echo 'redhat' > vault-pass
[student@workstation data-secret]$ chmod 0600 vault-pass
[student@workstation data-secret]$ ansible-playbook --vault-password-file=vault-pass create_users.yml
[student@workstation data-secret]$ ssh -o PreferredAuthentications=password ansibleuser1@servera.lab.example.com
[ansibleuser1@servera ~]$ exit
```

- Managing Facts

Ansible facts are variables that are automatically discovered by Ansible on a managed host.

```
[user@demo ~]$ cat facts.yml
---
- name: Fact dump
  hosts: all
  tasks:
    - name: Print all facts
      debug:
        var: ansible_facts

[user@demo ~]$ ansible-playbook facts.yml
```

-  Remember that when a variable's value is a hash/dictionary, there are two syntaxes that can be used to retrieve the value. To take two examples from the preceding table:

    ansible_facts['default_ipv4']['address'] can also be written ansible_facts.default_ipv4.address

    ansible_facts['dns']['nameservers'] can also be written ansible_facts.dns.nameservers 

```
---
- hosts: all
  tasks:
  - name: Prints various Ansible facts
    debug:
      msg: >
        The default IPv4 address of {{ ansible_facts.fqdn }}
        is {{ ansible_facts.default_ipv4.address }}
```

- Ansible Facts Injected as Variables

```
[user@demo ~]$ ansible demo1.example.com -m setup (ad-hoc que retorna facts)
```


- Turning off Fact Gathering
```
---
- name: This play gathers no facts automatically
  hosts: large_farm
  gather_facts: no
```

- Creating Custom Facts

 Custom facts allow administrators to define certain values for managed hosts, which plays might use to populate configuration files or conditionally run tasks. By default, the setup module loads custom facts from files and scripts in each managed host's /etc/ansible/facts.d directory. The name of each file or script must end in .fact in order to be used. Dynamic custom fact scripts must output JSON-formatted facts and must be executable. 
```
[user@demo1 ~]$ cat /etc/ansible/facts.d/host.fact

[packages]
web_package = httpd
db_package = mariadb-server

[users]
user1 = joe
user2 = jane

[user@demo ~]$ ansible demo1.example.com -m setup (ad-hoc que retorna facts)
```

- Custom facts can be used the same way as default facts in playbooks:

```
[user@demo ~]$ cat playbook.yml
---
- hosts: all
  tasks:
  - name: Prints various Ansible facts
    debug:
      msg: >
           The package to install on {{ ansible_facts['fqdn'] }}
           is {{ ansible_facts['ansible_local']['custom']['packages']['web_package'] }}

[user@demo ~]$ ansible-playbook playbook.yml
PLAY ***********************************************************************

TASK [Gathering Facts] *****************************************************
ok: [demo1.example.com]

TASK [Prints various Ansible facts] ****************************************
ok: [demo1.example.com] => {
    "msg": "The package to install on demo1.example.com  is httpd"
}

PLAY RECAP *****************************************************************
demo1.example.com    : ok=2    changed=0    unreachable=0    failed=0
```

- Using Magic Variables
 Some variables are not facts or configured through the setup module, but are also automatically set by Ansible. These magic variables can also be useful to get information specific to a particular managed host.

Four of the most useful are:

hostvars

    Contains the variables for managed hosts, and can be used to get the values for another managed host's variables. It does not include the managed host's facts if they have not yet been gathered for that host. 
group_names

    Lists all groups the current managed host is in. 
groups

    Lists all groups and hosts in the inventory. 
inventory_hostname

    Contains the host name for the current managed host as configured in the inventory. This may be different from the host name reported by facts for various reasons. 


[user@demo ~]$ ansible localhost -m debug -a 'var=hostvars["localhost"]'
```
localhost | SUCCESS => {
    "hostvars[\"localhost\"]": {
        "ansible_check_mode": false,
        "ansible_connection": "local",
        "ansible_diff_mode": false,
        "ansible_facts": {},
        "ansible_forks": 5,
        "ansible_inventory_sources": [
            "/home/student/demo/inventory"
        ],
        "ansible_playbook_python": "/usr/bin/python2",
        "ansible_python_interpreter": "/usr/bin/python2",
        "ansible_verbosity": 0,
        "ansible_version": {
            "full": "2.7.0",
            "major": 2,
            "minor": 7,
            "revision": 0,
            "string": "2.7.0"
        },
        "group_names": [],
        "groups": {
            "all": [
                "serverb.lab.example.com"
            ],
            "ungrouped": [],
            "webservers": [
                "serverb.lab.example.com"
            ]
        },
        "inventory_hostname": "localhost",
        "inventory_hostname_short": "localhost",
        "omit": "__omit_place_holder__18d132963728b2cbf7143dd49dc4bf5745fe5ec3",
        "playbook_dir": "/home/student/demo"
    }
}
```

### Guided Exercise: Managing Facts

```
[student@workstation data-facts]$ ansible webserver -m setup

vi /home/student/data-facts/custom.fact (TEM QUE SER .fact  E NÃO .facts)

[general]
package = httpd
service = httpd
state = started
enabled = true

vi setup_facts.yml
---
- name: Install remote facts
  hosts: webserver
  vars:
    remote_dir: /etc/ansible/facts.d
    facts_file: custom.fact
  tasks:
    - name: Create the remote directory
      file:
        state: directory
        recurse: yes
        path: "{{ remote_dir }}"
    - name: Install the new facts
      copy:
        src: "{{ facts_file }}"
        dest: "{{ remote_dir }}"


vi install-server.yml
---
- name: Install Apache and starts the service
  hosts: webserver

  tasks:
    - name: Install the required package
      yum:
        name: "{{ ansible_facts['ansible_local']['custom']['general']['package'] }}"
        state: latest

    - name: Start the service
      service:
        name: "{{ ansible_facts['ansible_local']['custom']['general']['service'] }}"
        state: "{{ ansible_facts['ansible_local']['custom']['general']['state'] }}"
        enabled: "{{ ansible_facts['ansible_local']['custom']['general']['enabled'] }}"

[student@workstation data-facts]$ ansible servera.lab.example.com -m command -a 'systemctl status httpd'
Unit httpd.service could not be found.non-zero return code

[student@workstation data-facts]$ ansible-playbook --syntax-check install-server.yml
[student@workstation data-facts]$ ansible-playbook install-server.yml
[student@workstation data-facts]$ ansible servera.lab.example.com -m command -a 'systemctl status httpd'


```

### Lab: Managing Variables and Facts


```

```

# Chapter 5: Implementação de controle de tarefas

## Writing Loops and Conditional Tasks

### Simple loop

```
- name: Postfix and Dovecot are running
  service:
    name: "{{ item }}"
    state: started
  loop:
    - postfix
    - dovecot

```

### Simple loop variable

```
vars:
  mail_services:
    - postfix
    - dovecot

tasks:
  - name: Postfix and Dovecot are running
    service:
      name: "{{ item }}"
      state: started
    loop: "{{ mail_services }}"
```

### Loops over a List of Hashes or Dictionaries

```
- name: Users exist and are in the correct groups
  user:
    name: "{{ item.name }}"
    state: present
    groups: "{{ item.groups }}"
  loop:
    - name: jane
      groups: wheel
    - name: joe
      groups: root
```
### Earlier-Style Loop Keywords (with_items,with_file,with_sequence)

```
  vars:
    data:
      - user0
      - user1
      - user2
  tasks:
    - name: "with_items"
      debug:
        msg: "{{ item }}"
      with_items: "{{ data }}"
```

### Using Register Variables with Loops
```
---
- name: Loop Register Test
  gather_facts: no
  hosts: localhost
  tasks:
    - name: Looping Echo Task
      shell: "echo This is my item: {{ item }}"
      loop:
        - one
        - two
      register: echo_results1

    - name: Show echo_results variable
      debug:
        var: echo_results2

```
### Using Register Variables with Loops (FILTRANDO A SAÍDA)

```
---
- name: Loop Register Test
  gather_facts: no
  hosts: localhost
  tasks:
    - name: Looping Echo Task
      shell: "echo This is my item: {{ item }}"
      loop:
        - one
        - two
      register: echo_results

    - name: Show stdout from the previous task.
      debug:
        msg: "STDOUT from previous task: {{ item.stdout }}"
      loop: "{{ echo_results['results'] }}"
```

## Running Tasks Conditionally

```
---
- name: Simple Boolean Task Demo
  hosts: all
  vars:
    run_my_task: true

  tasks:
    - name: httpd package is installed
      yum:
        name: httpd
      when: run_my_task
```
### If the my_service variable is not defined, then the task is skipped without an error. 

```
---
- name: Test Variable is Defined Demo
  hosts: all
  vars:
    my_service: httpd

  tasks:
    - name: "{{ my_service }} package is installed"
      yum:
        name: "{{ my_service }}"
      when: my_service is defined
```
### Using ansible_distribution variable 

```
---
- name: Demonstrate the "in" keyword
  hosts: all
  gather_facts: yes
  vars:
    supported_distros:
      - RedHat
      - Fedora
  tasks:
    - name: Install httpd using yum, where supported
      yum:
        name: http
        state: present
      when: ansible_distribution in supported_distros
```

### Testing Multiple Conditions

```
when: ansible_distribution == "RedHat" or ansible_distribution == "Fedora"

when: ansible_distribution_version == "7.5" and ansible_kernel == "3.10.0-327.el7.x86_64"

- list (AND operation)

when:
  - ansible_distribution_version == "7.5"
  - ansible_kernel == "3.10.0-327.el7.x86_64"

when: >
    ( ansible_distribution == "RedHat" and
      ansible_distribution_major_version == "7" )
    or
    ( ansible_distribution == "Fedora" and
    ansible_distribution_major_version == "28" )
```

### Combining Loops and Conditional Tasks 

```
- name: install mariadb-server if enough space on root
  yum:
    name: mariadb-server
    state: latest
  loop: "{{ ansible_mounts }}"
  when: item.mount == "/" and item.size_available > 300000000
```

### ### Combining Loops and Conditional Tasks and register variables

```
---
- name: Restart HTTPD if Postfix is Running
  hosts: all
  tasks:
    - name: Get Postfix server status
      command: /usr/bin/systemctl is-active postfix 1
      ignore_errors: yes2
      register: result3

    - name: Restart Apache HTTPD based on Postfix status
      service:
        name: httpd
        state: restarted
      when: result.rc == 0
```

### Guided Exercise: Writing Loops and Conditional Tasks

```
---
- name: Ensure MariaDB is installed and running
  hosts: database_dev
  vars:
    mariadb_packages:
      - mariadb-server
      - python3-PyMySQL
  tasks:

    - name: Install MariaDB
      yum:
        name: "{{ item }}"
        state: latest
      loop: "{{ mariadb_packages }}"

    - name: Start MariaDB service
      service:
        name: mariadb
        state: started
        enabled: true

---
- name: Ensure MariaDB is installed and running
  hosts: database_prod
  vars:
    mariadb_packages:
      - mariadb-server
      - python3-PyMySQL
  tasks:

    - name: Install MariaDB
      yum:
        name: "{{ item }}"
        state: latest
      loop: "{{ mariadb_packages }}"
      when: ansible_distribution == "RedHat" 

    - name: Start MariaDB service
      service:
        name: mariadb
        state: started
        enabled: true


ansible database_prod -m command  -a 'cat /etc/redhat-release' -u devops --become
```

### Ansible Handlers

- Handlers are tasks that respond to a notification triggered by other tasks. 
- Normally, handlers are used to reboot hosts and restart services. 
- Handlers can be considered as inactive tasks that only get triggered when explicitly invoked using a notify statement. 

```
tasks:
  - name: copy demo.example.conf configuration template
    template:
      src: /var/lib/templates/demo.example.conf.template
      dest: /etc/httpd/conf.d/demo.example.conf
    notify:
      - restart apache

handlers:
  - name: restart apache
    service:
      name: httpd
      state: restarted
```
### Ansible multiple Handlers

```
tasks:
  - name: copy demo.example.conf configuration template
    template:
      src: /var/lib/templates/demo.example.conf.template
      dest: /etc/httpd/conf.d/demo.example.conf
    notify:
      - restart mysql
      - restart apache

handlers:
  - name: restart mysql
    service:
      name: mariadb
      state: restarted

  - name: restart apache
    service:
      name: httpd
      state: restarted
```

- Handlers always run in the order specified by the handlers section of the play. 
- Handlers normally run after all other tasks in the play complete.
- If two handlers are incorrectly given the same name, only one will run.
- Even if more than one task notifies a handler, the handler only runs once.
- If a task that includes a notify statement does not report a changed result (for example, a package is already installed and the task reports ok), the handler is not notified. 


### Handlers exercise

```
---
- name: MariaDB server is installed
  hosts: databases
  vars:
    db_packages:
      - mariadb-server
      - python3-PyMySQL
    db_service: mariadb
    resources_url: http://materials.example.com/labs/control-handlers
    config_file_url: "{{ resources_url }}/my.cnf.standard"
    config_file_dst: /etc/my.cnf
  tasks:

    - name: "Install {{ db_packages }}"
      yum:
        name: "{{ db_packages }}"
        state: latest
      notify: set db password
  
    - name: Start DB service
      service:
        name: "{{ db_service }}
        state: started
        enabled: true
    
    - name: "The {{ config_file_dst }} is installed"
      get_url:
        url: "{{ config_file_url }}"
        dest: "{{ config_file_dst }}"
        owner: mysql
        group: mysql
        force: yes
      notify: restart db service

  handlers:

    - name: restart db service
      service:
        name: "{{ db_service }}"
        state: restarted

    - name: set db password
      mysql_user:
        name: root
        password: redhat

```

## Handling Task Failure
### Ignoring Task Failure

```
- name: Latest version of notapkg is installed
  yum:
    name: notapkg
    state: latest
  ignore_errors: yes
```

### Forcing Execution of Handlers after Task Failure

```
---
- hosts: all
  force_handlers: yes
  tasks:
    - name: a task which always notifies its handler
      command: /bin/true
      notify: restart the database

    - name: a task which fails because the package doesn't exist
      yum:
        name: notapkg
        state: latest

  handlers:
    - name: restart the database
      service:
        name: mariadb
        state: restarted
```

### Specifying Task Failure Conditions

```
tasks:
  - name: Run user creation script
    shell: /usr/local/bin/create_users.sh
    register: command_result
    failed_when: "'Password missing' in command_result.stdout"
```
### Specifying Task Failure Conditions - fail module

```
tasks:
  - name: Run user creation script
    shell: /usr/local/bin/create_users.sh
    register: command_result
    ignore_errors: yes

  - name: Report script failure
    fail:
      msg: "The password is missing in the output"
    when: "'Password missing' in command_result.stdout"
```

### Specifying When a Task Reports “Changed” Results

- Allways report ok or failed

```
  - name: get Kerberos credentials as "admin"
    shell: echo "{{ krb_admin_pass }}" | kinit -f admin
    changed_when: false
```
-  changed based on the output of the module

```
tasks:
  - shell:
      cmd: /usr/local/bin/upgrade-database
    register: command_result
    changed_when: "'Success' in command_result.stdout"
    notify:
      - restart_database

handlers:
  - name: restart_database
     service:
       name: mariadb
       state: restarted
```
### Ansible Blocks and Error Handling

-  block: Defines the main tasks to run. 
-  rescue: Defines the tasks to run if the tasks defined in the block clause fail. 
-  always: Defines the tasks that will always run independently of the success or failure of tasks defined in the block and rescue clauses. 

```
- name: block example
  hosts: all
  tasks:
    - name: installing and configuring Yum versionlock plugin 
      block:
      - name: package needed by yum
        yum:
          name: yum-plugin-versionlock
          state: present
      - name: lock version of tzdata
        lineinfile:
          dest: /etc/yum/pluginconf.d/versionlock.list
          line: tzdata-2016j-1
          state: present
      when: ansible_distribution == "RedHat"
```
 
 - rescue and always statements

```
  tasks:
    - name: Upgrade DB
      block:
        - name: upgrade the database
          shell:
            cmd: /usr/local/lib/upgrade-database
      rescue:
        - name: revert the database upgrade
          shell:
            cmd: /usr/local/lib/revert-database
      always:
        - name: always restart the database
          service:
            name: mariadb
            state: restarted
```

-  The when condition on a block clause also applies to its rescue and always clauses if present. 
### Guided Exercise: Handling Task Failure

- TEM QUE TER O NAME ENTRE tasks E block

```
---
- name: Controle de erros
  hosts: databases
  become: true
  vars:
    web_package: http
    db_package: mariadb-server
    db_service: mariadb
  tasks:
    - name: Install server

      block:
        - name: Install web package (nome errado)
          yum:
            name: "{{ web_package }}"
            state: latest
#          ignore_errors: yes
#          failed_when: web_package == "httpd"
    
      rescue:
        - name: Install database
          yum:
            name: "{{ db_package }}"
            state: latest

      always:
        - name: Start database
          service:
            name: "{{ db_service }}"
            state: started

```

## Lab: Implementing Task Control

```

```

# Chapter 6: Implantação de arquivos em hosts gerenciados

### Ensuring a File Exists on Managed Hosts
```
- name: Touch a file and set permissions
  file:
    path: /path/to/file
    owner: user1
    group: group1
    mode: 0640
    state: touch
```

### Modifying File Attributes
```
$ ls -Z samba_file
-rw-r--r-- owner group unconfined_u:object_r:user_home_t:s0 samba_file

- name: SELinux type is set to samba_share_t
  file:
    path: /path/to/samba_file
    setype: samba_share_t

$ ls -Z samba_file
-rw-r--r--  owner group unconfined_u:object_r:samba_share_t:s0 samba_file
```
### Making SELinux File Context Changes Persistent
-  The file module acts like chcon when setting file contexts. Changes made with that module could be unexpectedly undone by running restorecon. After using file to set the context, you can use sefcontext from the collection of System modules to update the SELinux policy like semanage fcontext. 
```
- name: SELinux type is persistently set to samba_share_t
  sefcontext:
    target: /path/to/samba_file
    setype: samba_share_t
    state: present
  
$ ls -Z samba_file
-rw-r--r--  owner group unconfined_u:object_r:samba_share_t:s0 samba_file
```

### Copying and Editing Files on Managed Hosts
```
- copy
- fetch
- lineinfile
- blockinfile
- file (absent para remover)
```

### Retrieving the Status of a File on Managed Hosts
```
- name: Verify the checksum of a file
  stat:
    path: /path/to/file
    checksum_algorithm: md5
  register: result

- debug
    msg: "The checksum of the file is {{ result.stat.checksum }}"
```
### Retrieving all atributes
```
- name: Examine all stat output of /etc/passwd
  hosts: localhost

  tasks:
    - name: stat /etc/passwd
      stat:
        path: /etc/passwd
      register: results

    - name: Display stat results
      debug:
        var: results
```
### Syncronize files (rsync)
```
- name: synchronize local file to remote files
  synchronize:
    src: file
    dest: /path/to/file
```
### Exercicio
```
---
- name: Modifying and Copying files 
  hosts: servers
  become: true
  remote_user: root
  tasks:
    - name: Fetch files from hosts
      fetch:
        src: /var/log/secure
        dest: secure-backups
        flat: no

---
- name: Modifying and Copying files 
  hosts: all
  become: true
  remote_user: root
  tasks:
    - name: Copy files to hosts
      copy:
        src: /home/student/file-manage/files/users.txt 
        dest: /home/devops/users.txt 
        owner: devops
        group: devops
        mode:  u+rw,g-wx,o-rwx
        setype: samba_share_t

[student@workstation file-manage]$ ansible all -m command -a 'ls -Z' -u devops

---
- name: Using the file module to ensure SELinux file context
  hosts: all
  remote_user: root
  tasks:
    - name:  SELinux file context is set to defaults
      file:
        path: /home/devops/users.txt
        seuser: _default
        serole: _default
        setype: _default
        selevel: _default

---
- name: Add text to an existing file
  hosts: all
  remote_user: devops
  tasks:
    - name: Add a single line of text to a file
      lineinfile:
        path: /home/devops/users.txt
        line: This line was added by the lineinfile module.
        state: present


---
- name: Add block of text to a file
  hosts: all
  remote_user: devops
  tasks:
    - name: Add a block of text to an existing file
      blockinfile:
        path: /home/devops/users.txt
        block: |
          This block of text consists of two lines.
          They have been added by the blockinfile module.
        state: present

---
- name: Use the file module to remove a file
  hosts: all
  remote_user: devops
  tasks:
    - name: Remove a file from managed hosts
      file:
        path: /home/devops/users.txt
        state: absent
```

### Deploying Custom Files with Jinja2 Templates
```
arquivo_exemplo.txt
{# /etc/hosts line #}  ESSE NÃO VAI APARECER
# {{ ansible_managed }} ESSE VAI APARECER
{{ ansible_facts['default_ipv4']['address'] }}    {{ ansible_facts['hostname'] }} (VIA ANSIBLE FACTS)
Port {{ ssh_port }} (DEFINIDA EM UMA VARIÁVEL)

tasks:
  - name: template render
    template:
      src: /tmp/arquivo_exemplo.txt
      dest: /tmp/dest-config-file.txt

```

###
```

```

###
```

```

###
```

```

###
```

```

###
```

```

# Chapter 7: Gerenciamento de projetos grandes


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
