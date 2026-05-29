---
name: commit-e-repo-standard
description: Padrão de commits e estrutura de repositório para o projeto acadêmico de Visão Computacional (Atividade 3 - TECA2/UFG)
metadata:
  type: project
---

# Design: Padrão de Commits e Estrutura de Repositório

**Projeto:** Atividade 3 — Descritores de Imagem  
**Disciplina:** TECA2 — UFG  
**Data:** 2026-05-29

---

## Convenção de Commits

### Formato

```
type(scope): descrição em minúsculas
```

### Tipos permitidos

| Tipo       | Quando usar                                                          |
|------------|----------------------------------------------------------------------|
| `feat`     | Nova implementação (novo descritor, novo método)                     |
| `fix`      | Correção de bug ou resultado incorreto                               |
| `notebook` | Reorganização de células, limpeza de outputs, formatação do `.ipynb` |
| `docs`     | Alterações no README, comentários                                    |
| `chore`    | Dependências, configuração, `.gitignore`                             |

### Escopos

- `(tarefa1)`, `(tarefa2)`, ... — para mudanças específicas de uma tarefa
- `(geral)` — para mudanças que afetam o repositório como um todo

### Exemplos

```
feat(tarefa1): implementa extrator SIFT
fix(tarefa1): corrige normalização do histograma HOG
notebook(tarefa1): limpa outputs e reorganiza células
docs(geral): preenche README com resultados da tarefa1
chore(geral): adiciona opencv-contrib às dependências
```

---

## Estrutura de Pastas

```
descriptors/
├── Tarefa1.ipynb          ← notebooks na raiz
├── Tarefa2.ipynb          ← (futuro)
├── docs/                  ← PDF da atividade + specs internas
│   ├── Atividade3_TECA2_20261_rev.pdf
│   └── superpowers/
│       └── specs/
├── images/                ← imagens utilizadas nos notebooks
├── README.md
├── pyproject.toml
└── uv.lock
```

### Migrações necessárias

- Unificar `doc/` e `docs/` → tudo em `docs/`
- Renomear `image/` → `images/`

---

## Estrutura do README

```markdown
# Atividade 3 — Descritores de Imagem
> Disciplina · Professor · Período

## Sobre a Atividade
[Descrição do que é pedido na atividade]

## Ambiente

### Pré-requisitos
- Python 3.12+
- [uv](https://docs.astral.sh/uv/)

### Instalação
\`\`\`bash
uv sync
\`\`\`

### Executar os notebooks
\`\`\`bash
uv run jupyter notebook
\`\`\`

## Estrutura do Repositório

| Caminho | Descrição |
|---|---|
| `Tarefa1.ipynb` | Notebook da tarefa 1 |
| `docs/` | PDF da atividade e documentação |
| `images/` | Imagens utilizadas nos notebooks |

## Resultados

### Tarefa 1
[Resumo dos resultados obtidos]

### Tarefa 2
[Resumo dos resultados obtidos]
```

---

## Decisões de Design

- **Escopo obrigatório:** deixa claro a qual tarefa cada commit pertence, facilitando rastreabilidade
- **Notebooks na raiz:** estrutura simples e direta, sem subpastas por tarefa — adequado para a quantidade de tarefas esperada
- **`images/` único:** imagens compartilhadas entre tarefas ficam em uma só pasta para evitar duplicação
- **README com Resultados:** seção por tarefa permite atualização incremental conforme o trabalho avança
