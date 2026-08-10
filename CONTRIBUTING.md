# Como contribuir neste repositório

Cada membro edita **somente 2 arquivos**. Isso evita conflito de merge.

## Passo 1: clonar

```bash
git clone URL_DO_REPO
cd grupo4-ansible-guia
```

## Passo 2: criar sua branch

Substitua `NN-assunto` pela sua branch da tabela em `docs/divisao-tarefas.md`.

```bash
git checkout -b feat/NN-assunto
```

## Passo 3: editar seus 2 arquivos

1. `docs/secoes/NN-assunto.md` - sua parte do guia prático
2. `exemplos/NN-nome.yml` - seu playbook de exemplo

**Não edite o `README.md`.** Ele é montado no final a partir das seções.

## Passo 4: commitar e enviar

```bash
git add docs/secoes/NN-assunto.md exemplos/NN-nome.yml
git commit -m "docs(NN): guia de <assunto> e playbook de exemplo"
git push -u origin feat/NN-assunto
```

## Passo 5: abrir Pull Request

No GitHub, abra um PR da sua branch para a `main`.

## Regras

- Mínimo de 2 commits na sua branch (requisito da atividade)
- O guia é **prático**: passo a passo, comandos, saída esperada. Conceito fica para os slides
- Seu playbook precisa rodar. Teste antes de commitar
- Prazo interno: terça-feira 11/08 até 20h
