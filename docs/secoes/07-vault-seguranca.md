# Ansible Vault e Hardening

**Responsável:** Tales Cavalcanti  
**Arquivo de exemplo:** `exemplos/07-vault-hardening.yml`

Esta seção mostra, de forma prática, como proteger informações sensíveis com **Ansible Vault** e como usar o Ansible para aplicar medidas de **hardening**. O exemplo foi construído para ser seguro em laboratório: ele cria uma configuração de SSH de demonstração em `/tmp` e **não altera o SSH real da máquina**.

---

## 1. Pré-requisitos

Antes de começar, confirme que:

- o Ansible está instalado;
- o repositório foi clonado;
- você está na raiz do projeto;
- o inventário `inventario/hosts.ini` contém o grupo `local`;
- o comando abaixo consegue alcançar o host local.

```bash
ansible -i inventario/hosts.ini local -m ping
```

Resultado esperado:

```text
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Também confira se o comando do Vault está disponível:

```bash
ansible-vault --help
```

---

# Parte A — Ansible Vault

## 2. O problema: não salvar secrets em texto puro

Playbooks e arquivos de variáveis podem precisar de dados sensíveis, por exemplo:

- senhas de banco de dados;
- tokens de API;
- credenciais de aplicações;
- senhas de usuários;
- chaves e outros valores privados.

Esses valores **não devem ficar em texto puro dentro do Git**. O Ansible Vault permite criptografar arquivos ou valores individuais e descriptografá-los somente durante a execução quando a senha correta do Vault é fornecida.

> Importante: a senha usada para abrir o Vault também é um segredo. Não coloque essa senha dentro do repositório.

---

## 3. Criando um arquivo de secrets para o laboratório

Para não adicionar um terceiro arquivo ao repositório, vamos criar o arquivo de secrets temporariamente em `/tmp`.

Crie o arquivo:

```bash
cat > /tmp/vault-demo.yml <<'EOF'
vault_demo_secret: "SenhaDeDemonstracao123!"
EOF
```

Nesse momento o conteúdo ainda está em texto puro. Confira:

```bash
cat /tmp/vault-demo.yml
```

Saída:

```yaml
vault_demo_secret: "SenhaDeDemonstracao123!"
```

---

## 4. Criptografando o arquivo com `ansible-vault encrypt`

Execute:

```bash
ansible-vault encrypt /tmp/vault-demo.yml
```

O Ansible pedirá uma senha:

```text
New Vault password:
Confirm New Vault password:
Encryption successful
```

Depois disso, visualize o arquivo diretamente:

```bash
cat /tmp/vault-demo.yml
```

O conteúdo não estará mais legível e começará de forma semelhante a:

```text
$ANSIBLE_VAULT;1.1;AES256
...
```

Agora o segredo está criptografado.

---

## 5. Visualizando um arquivo criptografado

Para ler o conteúdo sem descriptografar permanentemente o arquivo:

```bash
ansible-vault view /tmp/vault-demo.yml
```

Informe a senha do Vault.

Resultado esperado:

```yaml
vault_demo_secret: "SenhaDeDemonstracao123!"
```

O arquivo continua criptografado no disco.

---

## 6. Editando um arquivo protegido

Use:

```bash
ansible-vault edit /tmp/vault-demo.yml
```

O Ansible solicita a senha, abre o conteúdo em um editor e criptografa novamente quando o arquivo é salvo.

Esse método é melhor do que executar `decrypt`, editar em texto puro e depois lembrar de criptografar novamente.

---

## 7. Executando o playbook usando o arquivo criptografado

O playbook desta seção aceita a variável `vault_demo_secret` através de um arquivo externo.

Execute:

```bash
ansible-playbook -i inventario/hosts.ini exemplos/07-vault-hardening.yml \
  -e @/tmp/vault-demo.yml \
  --ask-vault-pass
```

O parâmetro `--ask-vault-pass` faz o Ansible solicitar a senha necessária para descriptografar o arquivo durante a execução.

O segredo é usado por uma task com:

```yaml
no_log: true
```

Isso evita que o valor sensível seja exibido normalmente na saída da task.

O playbook grava o segredo apenas no laboratório em:

```text
/tmp/ansible-vault-hardening/app.secret
```

com permissão:

```text
0600
```

Ou seja, somente o proprietário do arquivo possui leitura e escrita.

Para conferir a permissão sem mostrar o segredo:

```bash
ls -l /tmp/ansible-vault-hardening/app.secret
```

---

## 8. Criptografando somente uma variável

Nem sempre é necessário criptografar um arquivo inteiro. O `encrypt_string` permite gerar um valor YAML criptografado.

Exemplo:

```bash
ansible-vault encrypt_string --ask-vault-pass \
  --name 'db_password' \
  'SenhaBanco123!'
```

A saída poderá ser copiada para um arquivo YAML como uma variável protegida com `!vault`.

Isso é útil quando um arquivo possui várias configurações públicas e somente uma ou duas precisam ser secretas.

---

## 9. Trocando a senha do Vault

Para alterar a senha usada para proteger um arquivo:

```bash
ansible-vault rekey /tmp/vault-demo.yml
```

O comando solicita a senha atual e depois uma nova senha.

Esse recurso é útil quando existe rotação de credenciais ou quando alguém que conhecia a senha deixa a equipe.

---

## 10. Descriptografando permanentemente

É possível remover a criptografia com:

```bash
ansible-vault decrypt /tmp/vault-demo.yml
```

Use esse comando com cuidado. Depois dele, o secret volta a ficar em texto puro no disco.

Para este laboratório, se você usar `decrypt`, criptografe novamente depois:

```bash
ansible-vault encrypt /tmp/vault-demo.yml
```

---

# Parte B — Hardening com Ansible

## 11. O que o exemplo de hardening faz

O arquivo `exemplos/07-vault-hardening.yml` cria um pequeno laboratório de segurança em:

```text
/tmp/ansible-vault-hardening/
```

Ele demonstra práticas que também podem ser usadas em servidores reais:

1. cria um diretório com permissão `0700`;
2. cria um arquivo de configuração com permissão `0600`;
3. define opções mais restritivas de SSH;
4. impede que secrets apareçam normalmente nos logs usando `no_log: true`;
5. valida se o arquivo foi criado com a permissão esperada;
6. usa um handler para validar a configuração quando o arquivo muda.

O exemplo **não modifica `/etc/ssh/sshd_config`**, evitando que alguém se bloqueie para fora da própria máquina durante a aula.

---

## 12. Executando somente o hardening de demonstração

Não é obrigatório carregar o Vault para testar o restante do playbook.

Execute:

```bash
ansible-playbook -i inventario/hosts.ini exemplos/07-vault-hardening.yml
```

Ao terminar, visualize a configuração criada:

```bash
cat /tmp/ansible-vault-hardening/sshd_config
```

Conteúdo esperado:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
X11Forwarding no
PermitEmptyPasswords no
```

Confira as permissões:

```bash
ls -ld /tmp/ansible-vault-hardening
ls -l /tmp/ansible-vault-hardening/sshd_config
```

O diretório deve estar com `0700` e o arquivo com `0600`.

---

## 13. Entendendo as opções de hardening usadas

### `PermitRootLogin no`

Impede login SSH direto como `root`. Em um ambiente real, é preferível entrar com um usuário administrativo e elevar privilégio somente quando necessário.

### `PasswordAuthentication no`

Desabilita autenticação por senha no SSH. Em servidores configurados corretamente, o acesso pode ser feito com chaves SSH.

> Antes de aplicar isso em um servidor real, confirme que o acesso por chave SSH funciona. Caso contrário, você pode perder o acesso remoto.

### `PubkeyAuthentication yes`

Mantém habilitada a autenticação por chave pública.

### `MaxAuthTries 3`

Reduz a quantidade de tentativas de autenticação permitidas por conexão.

### `X11Forwarding no`

Desabilita o encaminhamento X11 quando ele não é necessário.

### `PermitEmptyPasswords no`

Impede autenticação SSH de contas com senha vazia.

---

## 14. Privilégio mínimo com `become`

Para alterar configurações reais de sistema, normalmente são necessários privilégios administrativos.

No Ansible isso pode ser feito com:

```yaml
become: true
```

Exemplo conceitual:

```yaml
- name: Alterar um arquivo protegido do sistema
  ansible.builtin.file:
    path: /etc/exemplo.conf
    mode: "0600"
  become: true
```

O ideal é elevar privilégio apenas onde ele é necessário, em vez de executar tudo como `root` sem necessidade.

Se o `sudo` exigir senha, o playbook pode ser executado com:

```bash
ansible-playbook -i inventario/hosts.ini playbook.yml --ask-become-pass
```

---

## 15. Testando antes de modificar um servidor

Antes de aplicar mudanças reais, utilize o modo de verificação quando os módulos utilizados oferecerem suporte:

```bash
ansible-playbook -i inventario/hosts.ini exemplos/07-vault-hardening.yml --check
```

Para visualizar diferenças em arquivos:

```bash
ansible-playbook -i inventario/hosts.ini exemplos/07-vault-hardening.yml --check --diff
```

Esse tipo de validação reduz o risco de aplicar uma configuração incorreta diretamente em produção.

---

## 16. Principais comandos do Ansible Vault

| Comando | O que faz |
|---|---|
| `ansible-vault create arquivo.yml` | Cria um novo arquivo já criptografado |
| `ansible-vault encrypt arquivo.yml` | Criptografa um arquivo existente |
| `ansible-vault decrypt arquivo.yml` | Remove a criptografia do arquivo |
| `ansible-vault view arquivo.yml` | Exibe o conteúdo sem descriptografar permanentemente |
| `ansible-vault edit arquivo.yml` | Edita o conteúdo mantendo o arquivo protegido |
| `ansible-vault rekey arquivo.yml` | Troca a senha/chave usada pelo Vault |
| `ansible-vault encrypt_string ...` | Criptografa somente um valor/variável |
| `ansible-playbook ... --ask-vault-pass` | Executa um playbook solicitando a senha do Vault |
| `ansible-playbook ... --ask-become-pass` | Solicita a senha para elevação de privilégio |

---

## 17. Boas práticas

- nunca commitar senhas em texto puro;
- nunca commitar a senha que desbloqueia o Vault;
- usar `no_log: true` em tasks que manipulam secrets;
- aplicar permissões restritivas como `0600` em arquivos sensíveis;
- usar `become` somente quando necessário;
- testar mudanças com `--check` e `--diff` antes de aplicá-las em produção;
- manter acesso por chave SSH testado antes de desabilitar login por senha;
- revisar playbooks de hardening para evitar bloqueio de acesso remoto;
- rotacionar senhas do Vault quando necessário;
- separar secrets das configurações públicas sempre que possível.

---

## 18. Erros comuns

| Erro | Causa provável | Solução |
|---|---|---|
| `Attempting to decrypt but no vault secrets found` | O playbook recebeu um arquivo criptografado, mas nenhuma senha do Vault foi fornecida | Execute com `--ask-vault-pass` |
| `Decryption failed` | Senha do Vault incorreta | Informe a senha correta ou confirme qual Vault foi usado |
| `vault_demo_secret is undefined` | O arquivo `/tmp/vault-demo.yml` não foi carregado | Use `-e @/tmp/vault-demo.yml --ask-vault-pass` |
| `Permission denied` | Uma task tentou alterar arquivo que exige privilégio administrativo | Use `become: true` e, se necessário, `--ask-become-pass` |
| SSH deixa de aceitar senha após hardening real | `PasswordAuthentication no` foi aplicado sem testar chave SSH | Teste autenticação por chave antes de desabilitar senha |
| Secret aparece em saída de task | A task sensível não está protegida | Adicione `no_log: true` à task que manipula o segredo |

---

## 19. Limpando o laboratório

Quando terminar os testes, remova os arquivos temporários:

```bash
rm -rf /tmp/ansible-vault-hardening
rm -f /tmp/vault-demo.yml
```

---

## 20. Como explicar o playbook em sala

Uma sequência simples para apresentação é:

1. **`hosts: local`** — executa no host `local` definido no inventário;
2. **`vars`** — define os caminhos usados no laboratório;
3. **`ansible.builtin.file`** — cria o diretório com permissão restrita;
4. **`ansible.builtin.copy`** — gera a configuração SSH segura de demonstração;
5. **`vault_demo_secret`** — variável que chega através do arquivo criptografado;
6. **`no_log: true`** — evita mostrar o secret normalmente na saída;
7. **`when`** — só executa a task do secret se a variável existir;
8. **`stat` + `assert`** — verifica se o arquivo existe e se está em `0600`;
9. **handler** — executa a validação quando a configuração muda;
10. **idempotência** — ao executar novamente sem alterar nada, as tasks de `file` e `copy` tendem a não realizar mudanças desnecessárias.

Uma demonstração rápida pode ser feita assim:

```bash
ansible-playbook -i inventario/hosts.ini exemplos/07-vault-hardening.yml
```

Depois:

```bash
ansible-playbook -i inventario/hosts.ini exemplos/07-vault-hardening.yml \
  -e @/tmp/vault-demo.yml \
  --ask-vault-pass
```

A diferença é que, na segunda execução, o playbook recebe um secret criptografado pelo Vault.

---

## Referências

- [Ansible Vault Guide](https://docs.ansible.com/projects/ansible/latest/vault_guide/index.html)
- [ansible-vault CLI](https://docs.ansible.com/projects/ansible/latest/cli/ansible-vault.html)
- [Privilege escalation: become](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_privilege_escalation.html)
- [Check mode e Diff mode](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_checkmode.html)
- [Ansible Playbook Guide](https://docs.ansible.com/ansible/latest/playbook_guide/)
