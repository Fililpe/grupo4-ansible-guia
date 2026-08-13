# Anatomia de um Playbook

**Responsável:** Filipe Sousa
**Arquivo de exemplo:** `exemplos/03-playbook-nginx.yml`

## Pré-requisitos

- `ansible-core` instalado (`ansible --version`)
- `ansible-lint` instalado (`pip install ansible-lint`)
- Inventário `inventario/hosts.ini` com o grupo `[local]`
- Executar todos os comandos a partir da raiz do repositório

## Passo a passo

### 1. Entender a estrutura

Um playbook é uma lista de plays. Cada play define onde rodar (`hosts`) e o que fazer (`tasks`).

```yaml
- name: Instalar e configurar um servidor web nginx
  hosts: local
  become: true

  vars:
    pacote_web: nginx

  tasks:
    - name: Garantir que o nginx esteja instalado
      ansible.builtin.apt:
        name: "{{ pacote_web }}"
        state: present
```

Regra de indentação: o que está sob o nome do módulo pertence ao módulo. O que está no nível do nome do módulo pertence à task. Por isso `notify` e `register` ficam alinhados com `ansible.builtin.apt:`, não com `name:`.

### 2. Validar a sintaxe

```bash
ansible-playbook --syntax-check exemplos/03-playbook-nginx.yml
```

Resultado esperado:

```
playbook: exemplos/03-playbook-nginx.yml
```

### 3. Verificar boas práticas

```bash
ansible-lint exemplos/03-playbook-nginx.yml
```

Resultado esperado:

```
Passed: 0 failure(s), 0 warning(s)
Last profile that met the validation criteria was 'production'.
```

O lint roda perfis progressivos e para no primeiro que falha. Se o último perfil aprovado for `basic`, corrija os erros e rode de novo até chegar em `production`.

### 4. Executar

```bash
ansible-playbook -i inventario/hosts.ini exemplos/03-playbook-nginx.yml
```

Resultado esperado na primeira execução:

```
PLAY RECAP *****
localhost : ok=8  changed=2  unreachable=0  failed=0
```

### 5. Comprovar a idempotência

Execute o mesmo comando outra vez, sem alterar nada:

```
PLAY RECAP *****
localhost : ok=7  changed=0  unreachable=0  failed=0
```

Nenhuma task muda nada na segunda execução. O Ansible é declarativo: cada módulo verifica o estado atual antes de agir e só executa se houver diferença. Um playbook idempotente sempre produz `changed=0` na segunda rodada.

### 6. Usar handlers

Handler é uma task que só roda quando é notificada, e apenas uma vez ao final do play.

```yaml
tasks:
  - name: Publicar pagina inicial personalizada
    ansible.builtin.copy:
      dest: /var/www/html/index.html
      content: |
        <h1>Provisionado com Ansible</h1>
    notify: Recarregar nginx

handlers:
  - name: Recarregar nginx
    ansible.builtin.service:
      name: "{{ pacote_web }}"
      state: reloaded
```

Duas regras importantes:

1. O handler só dispara se a task retornar `changed`, nunca em `ok`
2. Ele roda no fim do play, não logo após a notificação

Para ver o disparo, altere o conteúdo do `content` e execute de novo. O `RUNNING HANDLER` aparece depois da última task.

## Comandos principais

| Comando                                                    | O que faz                     |
| ---------------------------------------------------------- | ----------------------------- |
| `ansible-playbook --syntax-check <playbook>`               | Valida a sintaxe sem executar |
| `ansible-lint <playbook>`                                  | Verifica boas práticas        |
| `ansible-playbook -i <inventario> <playbook> --check`      | Simula a execução (dry run)   |
| `ansible-playbook -i <inventario> <playbook>`              | Executa o playbook            |
| `ansible-playbook -i <inventario> <playbook> --list-tasks` | Lista as tasks sem executar   |

## Erros comuns

| Erro                                                     | Causa                                         | Solução                                                              |
| -------------------------------------------------------- | --------------------------------------------- | -------------------------------------------------------------------- |
| `the playbook could not be found`                        | Comando rodado de outro diretório             | Execute da raiz usando `exemplos/03-playbook-nginx.yml`              |
| `Unable to parse inventario/hosts.ini`                   | Caminho do inventário errado                  | Volte à raiz e use `-i inventario/hosts.ini`                         |
| `Could not match supplied host pattern: local`           | Inventário não carregado ou grupo inexistente | Confirme o grupo `[local]` com `cat inventario/hosts.ini`            |
| `ansible-lint: command not found`                        | Pacote separado do `ansible-core`             | `pip install ansible-lint --break-system-packages`                   |
| `yaml[truthy]`                                           | Uso de `yes` e `no` como booleanos            | Troque por `true` e `false`                                          |
| `Unsupported parameters for (apt) module: content, dest` | Módulo errado para a ação                     | Use `copy` para arquivos, `apt` para pacotes, `service` para daemons |
| `Unsupported parameters for (copy) module: notify`       | `notify` indentado como parâmetro do módulo   | Alinhe `notify` com o nome do módulo                                 |
| Handler não dispara                                      | Task retornou `ok`, não `changed`             | Comportamento esperado. Altere o arquivo para forçar o `changed`     |

## Referências

- [Ansible Playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html)
- [Handlers](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html)
- [Módulo copy](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html)
- [ansible-lint](https://ansible.readthedocs.io/projects/lint/)
