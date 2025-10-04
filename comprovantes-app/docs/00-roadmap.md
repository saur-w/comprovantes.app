# 00 – Roadmap do Projeto ComprovantesApp

## 🎯 Objetivo
Desenvolver um aplicativo Android em Kotlin (Jetpack Compose) que permita armazenar e compartilhar comprovantes de pagamento entre dois usuários (ex: casal).

## 📅 Fases Principais
| Etapa | Descrição | Status |
|-------|------------|--------|
| M1 | Estrutura inicial (Room, Hilt, Compose) | ✅ Concluída |
| M2 | Firebase Auth + criação de Household | ⏳ Em andamento |
| M3 | Sincronização Firestore + Storage | 🔜 Próxima |
| M4 | Notificações (FCM) | ⏳ Planejada |
| M5 | Publicação Play Store | ⏳ Planejada |

## 📦 Stack Técnica
- **Kotlin + Jetpack Compose**
- **Room** (banco local)
- **Hilt** (injeção de dependência)
- **Firebase** (Auth, Firestore, Storage, Messaging)
- **WorkManager** (tarefas em segundo plano)

## 📈 Metas de aprendizado
- Entender arquitetura MVVM + Repository
- Implementar sincronização local/remota
- Aplicar práticas de segurança (Firestore Rules)
- Publicar o app na Play Store
