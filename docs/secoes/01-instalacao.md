# Ansible — Instalação, Pré-requisitos e Chaves SSH
**Responsável:** Ellen Nunes
## 1. Sobre esta etapa

Esta etapa apresenta a preparação do ambiente para utilização do **Ansible**, abordando:

* Instalação do Ansible;
* Execução do Ansible no Linux e no WSL;
* Verificação dos pré-requisitos;
* Configuração do OpenSSH;
* Geração de chaves SSH;
* Configuração da autenticação por chave;
* Teste da conexão SSH;
* Teste do Ansible utilizando SSH.

O objetivo é preparar uma máquina de controle capaz de utilizar o Ansible para se comunicar com uma máquina gerenciada por SSH.

---

# 2. Conceitos básicos

## 2.1 O que é o Ansible?

O Ansible é uma ferramenta de automação utilizada para configurar e administrar computadores, servidores e outros recursos de infraestrutura.

Ele permite executar tarefas de forma automatizada, como:

* Instalar pacotes;
* Configurar serviços;
* Criar e modificar arquivos;
* Gerenciar usuários;
* Executar comandos;
* Automatizar configurações de servidores.

O Ansible utiliza uma arquitetura baseada em um **control node**, que é a máquina onde o Ansible está instalado, e máquinas gerenciadas, nas quais as tarefas são executadas.

Neste exemplo, o ambiente utilizado foi:

```text
Windows
   │
   ▼
WSL 2
   │
   ▼
Ubuntu 26.04 LTS
   │
   ├── Ansible
   ├── Python
   └── OpenSSH
```

---

# 3. Ambiente utilizado

Para esta prática foi utilizado:

| Componente             | Versão            |
| ---------------------- | ----------------- |
| Sistema operacional    | Ubuntu 26.04 LTS  |
| Ambiente               | WSL 2             |
| Python                 | 3.14.4            |
| Ansible Core           | 2.20.1            |
| OpenSSH                | 10.2p1            |
| Método de autenticação | Chave SSH Ed25519 |

---

# 4. Utilizando o Ubuntu pelo WSL

No Windows, o Ansible pode ser utilizado por meio do **Windows Subsystem for Linux (WSL)**.

O WSL permite executar um ambiente Linux diretamente no Windows.

Para abrir o WSL pelo PowerShell:

```powershell
wsl
```

Após entrar no Ubuntu, o terminal apresenta um prompt semelhante a:

```bash
ellen@ELLENOVO:~$
```

A partir desse ponto, os comandos utilizados neste guia são executados no terminal Linux.

---

# 5. Verificação dos pré-requisitos

Antes da instalação do Ansible, é importante verificar se os principais componentes necessários estão disponíveis.

## 5.1 Verificar a distribuição Linux

```bash
lsb_release -a
```

Exemplo de resultado:

```text
Distributor ID: Ubuntu
Description:    Ubuntu 26.04 LTS
Release:        26.04
Codename:       resolute
```

---

## 5.2 Verificar o Python

O Ansible depende do Python para executar suas operações.

Verifique a versão instalada:

```bash
python3 --version
```

Resultado utilizado nesta prática:

```text
Python 3.14.4
```

---

## 5.3 Verificar o OpenSSH

O SSH é utilizado para comunicação entre o Ansible e os hosts gerenciados.

Verifique a versão do cliente SSH:

```bash
ssh -V
```

Resultado utilizado nesta prática:

```text
OpenSSH_10.2p1 Ubuntu-2ubuntu3, OpenSSL 3.5.5
```

---

# 6. Instalação do Ansible

Com os pré-requisitos disponíveis, atualize a lista de pacotes do Ubuntu:

```bash
sudo apt update
```

Em seguida, instale o Ansible:

```bash
sudo apt install ansible -y
```

## 6.1 Verificar a instalação

Após a instalação, execute:

```bash
ansible --version
```

Resultado obtido nesta prática:

```text
ansible [core 2.20.1]
```

Também é possível verificar a versão do Python utilizada pelo Ansible no mesmo resultado.

Se o comando `ansible --version` retornar as informações da instalação, o Ansible está disponível no ambiente.

---

# 7. Configuração do SSH

O SSH (**Secure Shell**) é um protocolo utilizado para estabelecer conexões seguras com sistemas remotos.

Na arquitetura do Ansible, ele é utilizado para permitir que o **control node** se comunique com os hosts gerenciados.

Nesta prática foi utilizado o OpenSSH.

Existem dois componentes importantes:

* **Cliente SSH:** utilizado para iniciar uma conexão;
* **Servidor SSH:** recebe as conexões.

---

# 8. Instalação do servidor OpenSSH

Primeiro, foi verificado se o serviço SSH estava disponível:

```bash
sudo service ssh status
```

Caso o servidor SSH não esteja instalado, instale-o com:

```bash
sudo apt install openssh-server -y
```

Depois, inicie o serviço:

```bash
sudo service ssh start
```

Verifique novamente:

```bash
sudo service ssh status
```

Um resultado válido deve apresentar:

```text
Active: active (running)
```

Isso indica que o servidor SSH está em execução.

Também é possível observar que o servidor está aguardando conexões na porta padrão:

```text
22
```

---

# 9. Geração das chaves SSH

Para utilizar autenticação por chave, é necessário gerar um par de chaves.

Foi utilizado o algoritmo **Ed25519**, que pode ser gerado com:

```bash
ssh-keygen -t ed25519
```

Quando o terminal perguntar onde salvar a chave, pressione `Enter` para utilizar o caminho padrão:

```text
/home/ellen/.ssh/id_ed25519
```

A chave privada foi criada em:

```text
~/.ssh/id_ed25519
```

E a chave pública em:

```text
~/.ssh/id_ed25519.pub
```

---

# 10. Chave privada e chave pública

A geração cria duas chaves:

```text
~/.ssh/
├── id_ed25519
└── id_ed25519.pub
```

### Chave privada

```text
id_ed25519
```

A chave privada deve permanecer protegida na máquina do usuário e **nunca deve ser compartilhada**.

### Chave pública

```text
id_ed25519.pub
```

A chave pública pode ser instalada no servidor para permitir a autenticação do usuário.

A segurança do método está baseada no par de chaves: a chave privada permanece protegida enquanto a chave pública pode ser disponibilizada no servidor.

---

# 11. Verificando as chaves

Para verificar os arquivos criados:

```bash
ls -la ~/.ssh
```

Exemplo:

```text
-rw------- 1 ellen ellen ... id_ed25519
-rw-r--r-- 1 ellen ellen ... id_ed25519.pub
```

A chave privada possui permissões restritas.

Para visualizar a chave pública:

```bash
cat ~/.ssh/id_ed25519.pub
```

> **Atenção:** não utilize `cat ~/.ssh/id_ed25519`, pois esse comando exibiria a chave privada.

---

# 12. Configuração da chave autorizada

Para permitir que a chave pública seja utilizada na autenticação SSH, ela pode ser adicionada ao arquivo `authorized_keys`.

Primeiro, crie o diretório SSH caso ele ainda não exista:

```bash
mkdir -p ~/.ssh
```

Adicione a chave pública:

```bash
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
```

Configure as permissões:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

O arquivo resultante fica em:

```text
~/.ssh/authorized_keys
```

Esse arquivo contém as chaves públicas autorizadas a realizar login.

---

# 13. Testando a conexão SSH

Com o servidor SSH em execução, é possível testar uma conexão com o próprio host utilizando `localhost`:

```bash
ssh localhost
```

Na primeira conexão, pode ser necessário confirmar a identidade do host:

```text
Are you sure you want to continue connecting?
```

Digite:

```text
yes
```

Depois, a autenticação pode solicitar a senha do usuário.

Se a conexão for realizada corretamente, será exibido um terminal semelhante a:

```text
ellen@ELLENOVO:~$
```

Para sair da conexão:

```bash
exit
```

---

# 14. Testando a autenticação por chave

Após adicionar a chave pública ao arquivo `authorized_keys`, execute novamente:

```bash
ssh localhost
```

Se a configuração estiver correta, a conexão será estabelecida sem solicitar a senha do usuário.

Isso demonstra que a autenticação está sendo realizada utilizando o par de chaves SSH.

O fluxo é:

```text
Chave privada
      │
      │ autenticação
      ▼
Servidor SSH
      │
      │ verifica
      ▼
authorized_keys
      │
      ▼
Acesso autorizado
```

---

# 15. Testando o Ansible

Após configurar o SSH, podemos verificar se o Ansible consegue utilizar essa conexão.

Execute:

```bash
ansible localhost -m ping -c ssh
```

Onde:

* `localhost` → host que será administrado;
* `-m ping` → utiliza o módulo `ping` do Ansible;
* `-c ssh` → utiliza SSH como método de conexão.

O resultado esperado é semelhante a:

```text
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## O que significa `SUCCESS`?

O resultado `SUCCESS` indica que o Ansible conseguiu estabelecer a conexão com o host e executar o módulo solicitado.

O retorno:

```text
"ping": "pong"
```

confirma que o módulo `ping` foi executado corretamente.

O campo:

```text
"changed": false
```

indica que o teste não realizou nenhuma alteração no sistema.

---

# 16. Fluxo completo da prática

A preparação realizada nesta etapa pode ser representada da seguinte forma:

```text
Windows
   │
   ▼
WSL 2
   │
   ▼
Ubuntu
   │
   ├── Python
   │
   ├── OpenSSH
   │
   └── Ansible
          │
          ▼
      Chave SSH
       /      \
Privada       Pública
   │             │
   │             ▼
   │       authorized_keys
   │             │
   └──────► SSH Server
                 │
                 ▼
              localhost
                 │
                 ▼
          Ansible -m ping
                 │
                 ▼
              SUCCESS
```

---

# 17. Comandos utilizados

Resumo dos principais comandos utilizados nesta prática:

### Verificar o sistema

```bash
lsb_release -a
```

### Verificar Python

```bash
python3 --version
```

### Verificar SSH

```bash
ssh -V
```

### Instalar Ansible

```bash
sudo apt update
sudo apt install ansible -y
```

### Verificar Ansible

```bash
ansible --version
```

### Instalar servidor SSH

```bash
sudo apt install openssh-server -y
```

### Iniciar SSH

```bash
sudo service ssh start
```

### Verificar SSH

```bash
sudo service ssh status
```

### Criar chave SSH

```bash
ssh-keygen -t ed25519
```

### Verificar chaves

```bash
ls -la ~/.ssh
```

### Adicionar chave pública

```bash
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
```

### Configurar permissões

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### Testar SSH

```bash
ssh localhost
```

### Testar Ansible

```bash
ansible localhost -m ping -c ssh
```

---

# 18. Segurança

Alguns cuidados são fundamentais ao trabalhar com chaves SSH:

* Nunca compartilhar a chave privada;
* Nunca publicar `id_ed25519` no GitHub;
* Não adicionar chaves privadas ao repositório;
* Manter permissões restritas para arquivos privados;
* Utilizar autenticação por chave sempre que possível;
* Proteger chaves privadas com uma passphrase em ambientes reais.

**Atenção:** a chave pública pode ser compartilhada, mas a chave privada deve permanecer protegida.

---

## Imagens da prática

![Imagem 1](../images/src/1.png)

![Imagem 2](../images/src/2.png)

![Imagem 3](../images/src/3.png)

![Imagem 4](../images/src/4.png)

![Imagem 5](../images/src/5.png)


# 19. Referências

* Documentação oficial do Ansible:
  https://docs.ansible.com/

* Guia oficial de instalação do Ansible:
  https://docs.ansible.com/projects/ansible/latest/installation_guide/intro_installation.html

* Documentação oficial do OpenSSH:
  https://www.openssh.com/

* Documentação oficial do Ubuntu:
  https://documentation.ubuntu.com/

* Documentação oficial do WSL:
  https://learn.microsoft.com/windows/wsl/

---

Resultado final:

```text
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

A partir dessa configuração, o ambiente está preparado para as próximas etapas do projeto, incluindo inventários, playbooks e demais recursos do Ansible.

