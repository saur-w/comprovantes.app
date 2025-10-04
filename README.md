# 📘 Estrutura de Documentação e Organização do Projeto

## 📂 Estrutura Recomendada de Repositório

```
comprovantes-app/
├─ app/                     # código Android Studio
├─ docs/                    # documentação por etapas
│  ├─ 00-roadmap.md
│  ├─ 01-guia-setup.md
│  ├─ 02-esqueleto-projeto.md
│  ├─ 03-room-banco-local.md
│  ├─ 04-auth-firebase.md
│  ├─ 05-sync-firestore-storage.md
│  ├─ 06-notificacoes.md
│  ├─ 07-publicacao-playstore.md
│  └─ anexos/               # imagens e diagramas
├─ .github/
│  ├─ ISSUE_TEMPLATE/
│  │  ├─ tarefa.md
│  │  └─ bug.md
│  └─ workflows/            # automações (opcional)
└─ README.md                # índice principal do projeto
```

---

## 🧭 Estrutura do README.md

### **Exemplo de índice principal:**

```md
# App de Comprovantes Compartilhados 📲

Aplicativo Android desenvolvido em Kotlin (Jetpack Compose) para armazenamento e compartilhamento de comprovantes de pagamento familiares.

## 📅 Etapas de Desenvolvimento
- [00 – Roadmap](docs/00-roadmap.md)
- [01 – Setup do Ambiente e Projeto](docs/01-guia-setup.md)
- [02 – Esqueleto do Projeto (Room + Hilt + Compose)](docs/02-esqueleto-projeto.md)
- [03 – Banco Local (Room) em detalhes](docs/03-room-banco-local.md)
- [04 – Autenticação Firebase + Household](docs/04-auth-firebase.md)
- [05 – Sincronização (Firestore/Storage/WorkManager)](docs/05-sync-firestore-storage.md)
- [06 – Notificações (listener e FCM)](docs/06-notificacoes.md)
- [07 – Publicação (Play Store)](docs/07-publicacao-playstore.md)

## ✅ Progresso das Etapas
- [x] Setup do ambiente e Firebase
- [x] Estrutura inicial (Room, Repository, ViewModels)
- [ ] Firebase Auth e criação de Household
- [ ] Sincronização Firestore/Storage
- [ ] Notificações
- [ ] Publicação

## 🛠️ Stack Técnica
- Kotlin + Jetpack Compose
- Hilt (DI)
- Room (Banco local)
- Firebase (Auth, Firestore, Storage, Messaging)
- WorkManager (tarefas em segundo plano)

## 📚 Licença
MIT License
```

---

## 🗒️ Modelo de Documento para Cada Etapa (`docs/NN-*.md`)

```md
# NN – Título da Etapa

## 🎯 Objetivo
Descrever o que será construído nesta etapa e qual o resultado esperado.

## 🧩 Pré-requisitos
- Etapa anterior concluída
- Dependências instaladas

## 🚀 Passo a Passo
1. Explicação detalhada de cada ação
2. Trechos de código exemplificando cada ponto
3. Resultado esperado

## 💻 Código Principal
Blocos de código principais + explicação.

## ✅ Checklist de Conclusão
- [ ] Compila sem erros
- [ ] Executa com sucesso
- [ ] Comportamento esperado verificado

## 🧠 Exercícios (opcional)
Exemplos de desafios práticos para reforço do aprendizado.
```

---

## 🗂️ Templates de Issue (`.github/ISSUE_TEMPLATE`)

### **tarefa.md**

```md
---
name: Tarefa
description: Implementação técnica
---

**Descrição:**
Breve resumo do que será feito.

**Critérios de Aceitação:**
- [ ] Requisito 1
- [ ] Requisito 2

**Notas Técnicas:**
Classes envolvidas, dependências, links.

**Etapa:**
Exemplo: M2-LocalDB
```

### **bug.md**

```md
---
name: Bug
description: Erro identificado no app
---

**Como reproduzir:**
Passos para reproduzir o erro.

**Resultado atual:**
...

**Resultado esperado:**
...

**Evidências:**
Logs, screenshots, vídeos.

**Ambiente:**
Dispositivo / versão / data.
```

---

## 🧱 Organização de Milestones e Labels

**Milestones:**

* `M1 – Base do Projeto`
* `M2 – Banco Local`
* `M3 – Autenticação Firebase`
* `M4 – Sincronização`
* `M5 – Notificações`
* `M6 – Publicação`

**Labels sugeridas:**

* `etapa:setup`
* `etapa:db`
* `etapa:auth`
* `etapa:sync`
* `UI`
* `bug`
* `doc`
* `prioridade:alta`

---

## 🗺️ Organização Visual (GitHub Projects)

Crie um **quadro Kanban** com colunas:

* `Backlog`
* `Em andamento`
* `Em revisão`
* `Concluído`

Cada issue é movida conforme o progresso da etapa.

---

## 📌 Próximos Passos

1. Criar repositório GitHub `comprovantes-app`
2. Adicionar estrutura de pastas e README inicial
3. Copiar este modelo de organização
4. Subir o conteúdo do código (já gerado nas etapas anteriores)
5. Criar milestones + issues correspondentes

Assim o projeto ficará **documentado, rastreável e didático** — perfeito para ensino e colaboração futura.
