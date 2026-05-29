# Commit e Repo Standard — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Reorganizar a estrutura de pastas do repositório e preencher o README seguindo o padrão definido na spec.

**Architecture:** Três mudanças independentes em sequência: (1) remover pasta `doc/` duplicada, (2) renomear `image/` para `images/` rastreando via git, (3) escrever o README completo. Cada mudança vira um commit próprio com a convenção definida.

**Tech Stack:** git, markdown

---

## Arquivos Impactados

| Ação | Caminho |
|---|---|
| Remover | `doc/Atividade3_TECA2_20261_rev.pdf` + `doc/` |
| Renomear (git mv) | `image/` → `images/` |
| Escrever | `README.md` |

---

### Task 1: Remover pasta `doc/` duplicada

O PDF já existe em `docs/`. A pasta `doc/` é um duplicado sem utilidade.

**Files:**
- Remove: `doc/Atividade3_TECA2_20261_rev.pdf`
- Remove: `doc/` (diretório)

- [ ] **Step 1: Verificar que o PDF existe em `docs/`**

```bash
ls docs/
```

Esperado: `Atividade3_TECA2_20261_rev.pdf` aparece na listagem.

- [ ] **Step 2: Remover a pasta `doc/` pelo git**

```bash
git rm -r doc/
```

Esperado:
```
rm 'doc/Atividade3_TECA2_20261_rev.pdf'
```

- [ ] **Step 3: Commitar**

```bash
git commit -m "chore(geral): remove pasta doc/ duplicada"
```

---

### Task 2: Renomear `image/` para `images/`

Padronizar o nome da pasta de imagens para plural, mantendo o histórico git.

**Files:**
- Rename: `image/` → `images/`

- [ ] **Step 1: Renomear via git mv**

```bash
git mv image images
```

- [ ] **Step 2: Verificar staging**

```bash
git status
```

Esperado: os 4 arquivos aparecem como `renamed: image/... -> images/...`

- [ ] **Step 3: Commitar**

```bash
git commit -m "chore(geral): renomeia image/ para images/"
```

---

### Task 3: Escrever o README

Preencher o `README.md` vazio com as quatro seções definidas na spec.

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Escrever o conteúdo completo do README**

Substituir o conteúdo de `README.md` por:

```markdown
# Atividade 3 — Descritores de Imagem

> Visão Computacional (TECA2) · UFG · 2026/1

## Sobre a Atividade

Esta atividade aborda a extração e comparação de descritores de imagem clássicos,
como SIFT, HOG e outros, aplicados a conjuntos de imagens de referência.
O objetivo é compreender as características de cada descritor e avaliar seu
desempenho em tarefas de correspondência e reconhecimento.

O enunciado completo está em [`docs/Atividade3_TECA2_20261_rev.pdf`](docs/Atividade3_TECA2_20261_rev.pdf).

## Ambiente

### Pré-requisitos

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)

### Instalação

```bash
uv sync
```

### Executar os notebooks

```bash
uv run jupyter notebook
```

## Estrutura do Repositório

| Caminho | Descrição |
|---|---|
| `Tarefa1.ipynb` | Notebook da Tarefa 1 |
| `docs/` | PDF da atividade e documentação interna |
| `images/` | Imagens utilizadas nos notebooks |
| `pyproject.toml` | Dependências e configuração do projeto |

## Resultados

### Tarefa 1

> _Preencher após conclusão da tarefa._

### Tarefa 2

> _Preencher após conclusão da tarefa._
```

- [ ] **Step 2: Verificar renderização (opcional)**

Abrir o `README.md` no editor e conferir se a formatação markdown está correta.

- [ ] **Step 3: Commitar**

```bash
git add README.md
git commit -m "docs(geral): preenche README com estrutura inicial do projeto"
```

---

## Verificação Final

- [ ] `doc/` não existe mais no repositório
- [ ] `images/` existe com os 4 arquivos originais de `image/`
- [ ] `README.md` tem as 4 seções: Sobre, Ambiente, Estrutura, Resultados
- [ ] `git log --oneline` mostra 3 novos commits com a convenção definida
