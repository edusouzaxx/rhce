


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

```

# Chapter 4: Gerenciamento de variáveis e fatos

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