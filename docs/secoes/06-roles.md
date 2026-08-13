# Roles e Ansible Galaxy

**Responsável:** Jhonny Emanoel  
**Arquivo de exemplo:** `exemplos/06-roles/`

## Pré-requisitos

Antes de executar esta etapa, é necessário ter:

* Ansible instalado e funcionando.
* Acesso ao repositório do projeto.
* Inventário configurado em `inventario/hosts.ini`.
* Para execução remota: acesso SSH a um host Linux definido no inventário.
* Para execução local: uma máquina Linux com Ansible instalado.
* Conhecimento básico de `playbooks`, `plays`, `tasks` e `modules`.

## Passo a passo

### 1. Criar a estrutura da Role

As Roles organizam tarefas, arquivos, templates e variáveis em uma estrutura padronizada, facilitando a reutilização e manutenção do código.

A Role `webserver` utilizada neste exemplo já está incluída no repositório. Para criar uma nova Role do zero, o comando utilizado seria:

```bash
ansible-galaxy role init webserver
```

Resultado esperado:

```text
- Role webserver was created successfully
```

A estrutura criada será semelhante a:

```text
webserver/
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
├── tests/
│   ├── inventory
│   └── test.yml
└── vars/
    └── main.yml
```

### 2. Configurar o inventário

O arquivo `inventario/hosts.ini` define em quais máquinas a Role será executada.

Para executar localmente, pode ser utilizado o grupo `local` já existente no inventário:

```ini
[local]
localhost ansible_connection=local
```

Nesse caso, o `playbook.yml` deve utilizar:

```yaml
hosts: local
```

Para executar em uma máquina Linux remota, adicione o endereço IP e o usuário ao grupo `webservers`:

```ini
[webservers]
web01 ansible_host=192.168.56.10 ansible_user=usuario
```

Caso seja necessário especificar uma chave SSH:

```ini
[webservers]
web01 ansible_host=192.168.56.10 ansible_user=usuario ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

Nesse caso, o `playbook.yml` deve utilizar:

```yaml
hosts: webservers
```

Não coloque senhas ou chaves privadas diretamente no arquivo `hosts.ini`. A autenticação deve ser configurada previamente no ambiente.

### 3. Definir as tarefas da Role

As tarefas principais da Role ficam em `tasks/main.yml`. Nesse arquivo são utilizados módulos do Ansible para realizar a configuração desejada.

Exemplo:

```yaml
---
- name: Instalar nginx
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: true

- name: Iniciar nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```

A Role pode ser utilizada por um playbook através da propriedade `roles`:

```yaml
---
- name: Configurar servidor web
  hosts: webservers
  become: true

  roles:
    - webserver
```

Nesse caso, o Ansible executará automaticamente as tarefas definidas em `webserver/tasks/main.yml`.

### 4. Executar a Role

Após clonar o repositório, entre na pasta raiz do projeto:

```bash
cd grupo4-ansible-guia
```

Primeiro, verifique se o host definido no inventário está acessível:

```bash
ansible -i inventario/hosts.ini webservers -m ansible.builtin.ping
```

Resultado esperado:

```text
web01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Antes da execução, valide a sintaxe do playbook:

```bash
ansible-playbook --syntax-check -i inventario/hosts.ini exemplos/06-roles/playbook.yml
```

Resultado esperado:

```text
playbook: exemplos/06-roles/playbook.yml
```

Por fim, execute o playbook:

```bash
ansible-playbook -i inventario/hosts.ini exemplos/06-roles/playbook.yml
```

Como a Role utiliza `become: true`, o usuário utilizado pelo Ansible precisa possuir privilégios administrativos na máquina de destino.

Para execução local utilizando o grupo `local`, o comando é:

```bash
ansible -i inventario/hosts.ini local -m ansible.builtin.ping
```

E depois:

```bash
ansible-playbook -i inventario/hosts.ini exemplos/06-roles/playbook.yml
```

### 5. Utilizar o Ansible Galaxy

O Ansible Galaxy permite encontrar e instalar Roles prontas para serem utilizadas em projetos Ansible.

Para pesquisar Roles disponíveis:

```bash
ansible-galaxy search nginx
```

Para instalar uma Role:

```bash
ansible-galaxy role install nome-da-role
```

Também é possível listar as Roles instaladas:

```bash
ansible-galaxy role list
```

As Roles instaladas podem então ser utilizadas nos playbooks da mesma forma que uma Role criada localmente.

## Comandos principais

| Comando | O que faz |
|---|---|
| `ansible-galaxy role init webserver` | Cria a estrutura inicial de uma Role. |
| `ansible-galaxy search nginx` | Pesquisa Roles relacionadas ao termo informado. |
| `ansible-galaxy role install nome-da-role` | Instala uma Role do Ansible Galaxy. |
| `ansible-galaxy role list` | Lista as Roles instaladas no ambiente. |
| `ansible -i inventario/hosts.ini webservers -m ansible.builtin.ping` | Testa a comunicação com os hosts do grupo `webservers`. |
| `ansible-playbook -i inventario/hosts.ini exemplos/06-roles/playbook.yml` | Executa o playbook que utiliza a Role. |
| `ansible-playbook --syntax-check -i inventario/hosts.ini exemplos/06-roles/playbook.yml` | Verifica a sintaxe do playbook sem executá-lo. |

## Erros comuns

| Erro | Causa | Solução |
|---|---|---|
| `ERROR! the role 'webserver' was not found` | O Ansible não encontrou a Role no caminho esperado. | Verifique a estrutura de diretórios e se a Role está localizada em `exemplos/06-roles/webserver/`. |
| `ansible-galaxy: command not found` | O Ansible não está instalado ou não está disponível no PATH. | Instale o Ansible e verifique sua instalação com `ansible --version`. |
| Falha de conexão SSH | O host não está acessível ou as credenciais SSH estão incorretas. | Verifique o `inventario/hosts.ini`, a chave SSH e a conectividade com o host. |
| `skipping: no hosts matched` | O valor definido em `hosts:` não corresponde a nenhum grupo ou host do inventário. | Confira se `hosts:` corresponde ao grupo utilizado no `hosts.ini`, como `local` ou `webservers`. |
| Erro de sintaxe no playbook | YAML ou estrutura do playbook está incorreta. | Execute `ansible-playbook --syntax-check` antes de executar o playbook. |
| Permissão negada durante a configuração | A tarefa exige privilégios administrativos. | Utilize `become: true` no playbook e certifique-se de que o usuário possui privilégios administrativos. |

## Referências

* [Documentação oficial — Ansible Roles](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)
* [Documentação oficial — Ansible Galaxy](https://docs.ansible.com/ansible/latest/galaxy/user_guide.html)
* [Documentação oficial — ansible-galaxy](https://docs.ansible.com/ansible/latest/cli/ansible-galaxy.html)