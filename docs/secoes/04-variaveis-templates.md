# Variáveis, Facts e Templates Jinja2

**Responsável:** Miqueias Eduardo

**Arquivo de exemplo:** `exemplos/04-template-config.yml`

## Pré-requisitos

Antes de utilizar variáveis, facts e templates Jinja2 é necessário:

- Ter o Python instalado na máquina.
- Ter o Ansible instalado na máquina.
- Possuir um inventário configurado.
- Ter pelo menos um host definido no inventário.
- Possuir um playbook para executar as tarefas.
- Ter acesso ao host através de SSH.

## Passo a passo

### 1. Utilizando variáveis

Variáveis permitem armazenar valores reutilizáveis dentro dos playbooks. Isso facilita a manutenção do código e evita repetição de informações.

Exemplo:

```yaml
---
- hosts: all

  vars:
    usuario: admin

  tasks:
    - name: Exibir usuário
      debug:
        msg: "Usuário: {{ usuario }}"
```

Resultado esperado:

```text
ok: [servidor01] => {
    "msg": "Usuário: admin"
}
```

### 2. Utilizando Facts

Facts são informações coletadas automaticamente pelo Ansible sobre os hosts presentes no inventário. Esses dados podem ser utilizados durante a execução dos playbooks para adaptar tarefas de acordo com o ambiente.

Alguns exemplos de facts disponíveis são:

- Hostname da máquina.
- Sistema operacional.
- Família do sistema operacional.
- Memória disponível.
- Endereço IP.

Exemplo:

```yaml
- name: Exibir hostname
  debug:
    msg: "Hostname: {{ ansible_hostname }}"
```

Resultado esperado:

```text
ok: [servidor01] => {
    "msg": "Hostname: servidor01"
}
```

### 3. Utilizando Templates Jinja2

Templates Jinja2 permitem gerar arquivos dinâmicos utilizando variáveis e facts. Em vez de criar vários arquivos manualmente, é possível criar um modelo e deixar o Ansible preencher os valores automaticamente durante a execução.

Exemplo de template:

```jinja2
Servidor: {{ ansible_hostname }}
Usuario: {{ usuario }}
```

Arquivo gerado após a execução:

```text
Servidor: servidor01
Usuario: admin
```

Essa abordagem é amplamente utilizada para gerar arquivos de configuração de aplicações, servidores web, bancos de dados e outros serviços.

## Conceitos principais

| Conceito | Descrição |
|-----------|------------|
| Variáveis | Permitem armazenar e reutilizar valores dentro dos playbooks. |
| Facts | Informações coletadas automaticamente pelo Ansible sobre os hosts gerenciados. |
| Templates Jinja2 | Modelos de arquivos que utilizam variáveis e facts para gerar conteúdo dinâmico. |

## Referências

- https://docs.ansible.com/
- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html
- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html
- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_templating.html
