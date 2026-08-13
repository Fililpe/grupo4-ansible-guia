# Inventário e Comandos Ad-hoc
**Responsável:** André Costa
**Arquivo de exemplo:** `exemplos/02-inventario-adhoc.yml`

O inventário informa ao Ansible quais máquinas serão gerenciadas, como elas são
organizadas em grupos e quais dados devem ser usados na conexão. Neste guia será
utilizado um inventário estático no formato INI e o próprio computador como
máquina de teste.

## Pré-requisitos

- Ansible instalado na máquina de controle
- Terminal aberto na raiz deste repositório
- Arquivo `inventario/hosts.ini` disponível
- Python 3 instalado

Confirme a instalação:

```bash
ansible --version
```

Caso ainda não instalado:

```bash
sudo apt install ansible
```

## 1. Entender o inventário do projeto

Abra o arquivo `inventario/hosts.ini`:

```ini
[local]
localhost ansible_connection=local

[webservers]
# web01 ansible_host=192.168.56.10 ansible_user=vagrant

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

Elementos desse inventário:

| Elemento | Função |
| --- | --- |
| `[local]` | Cria um grupo chamado `local` |
| `localhost` | Nome do host que será administrado |
| `ansible_connection=local` | Executa as tarefas no próprio computador, sem SSH |
| `[webservers]` | Cria um grupo para futuros servidores web |
| `web01` | Apelido que poderia identificar um servidor remoto |
| `ansible_host` | Endereço IP ou nome DNS utilizado na conexão |
| `ansible_user` | Usuário utilizado para acessar o host remoto |
| `[all:vars]` | Define variáveis aplicadas a todos os hosts |

A linha de `web01` começa com `#`, por isso é apenas um exemplo e não será
carregada pelo Ansible.

## 2. Visualizar os grupos e hosts

Execute:

```bash
ansible-inventory -i inventario/hosts.ini --graph
```

Saída esperada, de forma resumida:

```text
@all:
  |--@ungrouped:
  |--@local:
  |  |--localhost
  |--@webservers:
```

O grupo `all` é criado automaticamente e representa todos os hosts. O grupo
`ungrouped` reúne hosts que não pertencem a nenhum grupo declarado pelo usuário.

Para visualizar todas as informações e variáveis em JSON, utilize:

```bash
ansible-inventory -i inventario/hosts.ini --list
```

## 3. Testar a comunicação com um comando ad-hoc

Um comando ad-hoc executa uma ação rápida sem a necessidade de criar um
playbook. A estrutura básica é:

```text
ansible PADRÃO_DE_HOSTS -i INVENTÁRIO -m MÓDULO -a "ARGUMENTOS"
```

Teste o grupo `local` com o módulo `ping`:

```bash
ansible local -i inventario/hosts.ini -m ansible.builtin.ping
```

Saída esperada:

```text
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

O `ping` do Ansible não é o mesmo comando de rede baseado em ICMP. Ele verifica
se o Ansible consegue executar um módulo no host e receber uma resposta válida.

## 4. Executar outros comandos ad-hoc

Descobrir o nome da máquina:

```bash
ansible local -i inventario/hosts.ini -m ansible.builtin.command -a "hostname"
```

Coletar informações do sistema operacional, memória, rede e hardware:

```bash
ansible local -i inventario/hosts.ini -m ansible.builtin.setup
```

Exibir somente a distribuição do sistema operacional:

```bash
ansible local -i inventario/hosts.ini -m ansible.builtin.setup -a "filter=ansible_distribution*"
```

## 5. Executar o exemplo em playbook

O arquivo `exemplos/02-inventario-adhoc.yml` transforma as verificações rápidas
em tarefas documentadas e repetíveis.

Primeiro, valide a sintaxe:

```bash
ansible-playbook --syntax-check -i inventario/hosts.ini exemplos/02-inventario-adhoc.yml
```

Depois, execute:

```bash
ansible-playbook -i inventario/hosts.ini exemplos/02-inventario-adhoc.yml
```

O resumo final deve apresentar `failed=0`. Como as tarefas apenas consultam
informações, o resultado também deverá apresentar `changed=0`.

## 6. Como adicionar um servidor remoto

Um servidor Linux acessível por SSH poderia ser declarado assim:

```ini
[webservers]
web01 ansible_host=192.168.56.10 ansible_user=usuario
```

Nesse exemplo:

- `web01` é o nome usado dentro do Ansible;
- `192.168.56.10` é o endereço real da máquina;
- `usuario` é a conta usada na conexão SSH.

Não coloque senhas diretamente no inventário. Utilize autenticação por chave SSH
e, quando for necessário guardar informações sensíveis, utilize o Ansible Vault.

## Padrões de hosts

O primeiro argumento depois de `ansible` determina quais hosts receberão a ação:

| Padrão | Alvo |
| --- | --- |
| `all` | Todos os hosts do inventário |
| `local` | Todos os hosts do grupo `local` |
| `localhost` | Somente o host chamado `localhost` |
| `webservers` | Todos os hosts do grupo `webservers` |
| `webservers:&producao` | Hosts presentes nos dois grupos |
| `all:!webservers` | Todos os hosts, exceto os servidores web |

## Comandos principais

| Comando | O que faz |
| --- | --- |
| `ansible-inventory -i inventario/hosts.ini --graph` | Exibe a organização do inventário |
| `ansible-inventory -i inventario/hosts.ini --list` | Lista hosts e variáveis em JSON |
| `ansible local -i inventario/hosts.ini --list-hosts` | Mostra os hosts selecionados sem executar tarefas |
| `ansible local -i inventario/hosts.ini -m ansible.builtin.ping` | Testa a comunicação do Ansible |
| `ansible local -i inventario/hosts.ini -m ansible.builtin.command -a "hostname"` | Executa uma ação rápida no grupo `local` |
| `ansible local -i inventario/hosts.ini -m ansible.builtin.setup` | Coleta informações dos hosts |

## Erros comuns

| Erro | Causa provável | Solução |
| --- | --- | --- |
| `command not found: ansible` | Ansible não instalado ou fora do `PATH` | Instalar o Ansible e abrir novamente o terminal |
| `Unable to parse ... as an inventory source` | Caminho ou sintaxe do inventário incorretos | Conferir o caminho e executar `ansible-inventory --graph` |
| `No hosts matched` | Grupo ou host informado não existe ou está comentado | Conferir o padrão de hosts e o arquivo `hosts.ini` |
| `UNREACHABLE!` | Falha de rede, SSH, usuário ou chave | Testar o acesso SSH e revisar as variáveis de conexão |
| `The interpreter ... was not found` | Caminho do Python incorreto no host | Verificar com `which python3` e ajustar `ansible_python_interpreter` |
| `Permission denied` | Usuário sem acesso ou chave SSH inválida | Corrigir usuário, chave e permissões de conexão |

## Referências

- [Como criar um inventário](https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_inventory.html)
- [Introdução aos comandos ad-hoc](https://docs.ansible.com/projects/ansible/latest/command_guide/intro_adhoc.html)
- [Padrões para selecionar hosts e grupos](https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_patterns.html)
- [Módulos do Ansible](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/index.html)
