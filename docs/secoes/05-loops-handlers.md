# Loops, Handlers e Tags

**Responsável:** Lucas Emmanoel

**Arquivo de exemplo:** [exemplos/05-loops-handlers.yml](../../exemplos/05-loops-handlers.yml)

## Objetivo

Demonstrar o uso de loops para instalar pacotes, handlers para reiniciar serviços quando necessário, e tags para executar partes específicas do playbook.

## Pré-requisitos

- Ansible instalado na máquina de controle.
- Inventário disponível em [inventario/hosts.ini](inventario/hosts.ini).
- Acesso com permissão de sudo/`become` nos hosts alvo.

## Como executar

Executar o playbook de exemplo:

```bash
ansible-playbook -i inventario/hosts.ini exemplos/05-loops-handlers.yml
```

Executar somente tarefas marcadas com uma tag (ex.: `pacotes`):

```bash
ansible-playbook -i inventario/hosts.ini exemplos/05-loops-handlers.yml --tags pacotes
```

Executar e mostrar diferenças sem aplicar (modo dry-run):

```bash
ansible-playbook -i inventario/hosts.ini exemplos/05-loops-handlers.yml --check --diff
```

## Explicação dos blocos do playbook

1. Exemplo 1 — Loop (instalação de pacotes)

- Tarefa: `ansible.builtin.apt` com `loop` para instalar `nginx`, `git`, `curl` e `vim`.
- `update_cache: true` assegura que o cache do apt seja atualizado.
- Tags: `pacotes`, `loop` — útil para executar apenas esse grupo de tarefas.

2. Exemplo 2 — Handler (configuração do Nginx)

- Tarefa: `ansible.builtin.copy` sobrescreve `/etc/nginx/sites-available/default` com uma configuração mínima que retorna uma resposta 200.
- A tarefa usa `notify` para disparar o handler `Reiniciar Nginx` apenas quando houver mudança no arquivo.
- Tags: `nginx`, `configuracao`.

3. Exemplo 3 — Tag (arquivo de teste)

- Tarefa: `ansible.builtin.file` cria `/tmp/ansible-teste.txt` (estado `touch`).
- Tag: `teste` — permite executar essa tarefa isoladamente.

### Handlers

- `Reiniciar Nginx`: usa `ansible.builtin.service` para reiniciar o serviço `nginx` quando notificado.

Handlers são executados ao final do play por padrão, e somente se uma ou mais tarefas deram `changed` e notificaram o handler.

## Resultado esperado

- Os pacotes listados estão instalados (`nginx`, `git`, `curl`, `vim`).
- O arquivo `/etc/nginx/sites-available/default` é atualizado com a configuração do exemplo.
- Se a cópia do arquivo alterou algo, o handler reinicia o serviço `nginx`.
- O arquivo `/tmp/ansible-teste.txt` é criado.

## Erros comuns e soluções

- `apt` falhando em distribuições não Debian/Ubuntu: use o módulo apropriado (`dnf`, `yum`, `pacman`) ou rode o play em hosts Debian/Ubuntu.
- Permissão negada ao gravar em `/etc/nginx/...`: verifique `become: true` e credenciais de sudo.
- Handler não executado: verifique se a tarefa que notifica realmente mudou (`changed: true`); em modo `--check` handlers não serão acionados.

## Dicas

- Use `--tags` e `--skip-tags` para controlar execução.
- Teste com `--check --diff` antes de aplicar mudanças.
- Verifique logs do systemd (`journalctl -u nginx`) se o serviço não reiniciar corretamente.

## Referências

- Playbook de exemplo: [exemplos/05-loops-handlers.yml](../../exemplos/05-loops-handlers.yml)
- Documentação Ansible — Loops: https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_loops.html
- Documentação Ansible — Handlers: https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html
- Documentação Ansible — Tags: https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_tags.html

