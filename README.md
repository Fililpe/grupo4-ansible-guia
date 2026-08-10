# Guia Prático de Ansible

Guia passo a passo de utilização do Ansible, produzido pelo Grupo 4 da Trilha DevOps, Turma 5, Aponti FAP 2026.

## Sumário

1. [Instalação e Configuração Inicial](docs/secoes/01-instalacao.md)
2. [Inventário e Comandos Ad-hoc](docs/secoes/02-inventario.md)
3. [Anatomia de um Playbook](docs/secoes/03-playbooks.md)
4. [Variáveis, Facts e Templates Jinja2](docs/secoes/04-variaveis-templates.md)
5. [Loops, Condicionais, Handlers e Tags](docs/secoes/05-loops-handlers.md)
6. [Roles e Ansible Galaxy](docs/secoes/06-roles.md)
7. [Ansible Vault e Hardening](docs/secoes/07-vault-seguranca.md)

## Início rápido

```bash
git clone URL_DO_REPO
cd grupo4-ansible-guia
ansible --version
ansible -i inventario/hosts.ini local -m ping
```

## Estrutura do repositório

grupo4-ansible-guia/
├── README.md guia consolidado
├── CONTRIBUTING.md fluxo de trabalho do grupo
├── inventario/ inventário de exemplo
├── docs/
│ ├── divisao-tarefas.md
│ └── secoes/ uma seção por membro
└── exemplos/ playbooks de exemplo

## Divisão de tarefas

Ver [docs/divisao-tarefas.md](docs/divisao-tarefas.md).

## Referências

Consolidadas ao final de cada seção.
