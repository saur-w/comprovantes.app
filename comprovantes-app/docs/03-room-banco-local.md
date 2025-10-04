# 03 – Banco Local (Room)

## 🎯 Objetivo
Implementar o armazenamento local dos comprovantes de pagamento usando **Room**, permitindo o uso offline e sincronização posterior.

---

## 🧩 O que é o Room?
Room é a biblioteca do Android que facilita o uso de banco de dados **SQLite**, oferecendo:
- Segurança de tipos com **DAO (Data Access Object)**
- Consultas automáticas via **coroutines / Flow**
- Integração direta com ViewModels

---

## ⚙️ Estrutura do Banco

| Componente | Função |
|-------------|--------|
| `ReceiptEntity.kt` | Define o modelo salvo no banco |
| `ReceiptDao.kt` | Define as operações de CRUD |
| `AppDatabase.kt` | Cria e configura o banco local |

---

## 🧱 Exemplo de Entity
```kotlin
@Entity(tableName = "receipts")
data class ReceiptEntity(
    @PrimaryKey val id: String,
    val householdId: String,
    val description: String,
    val amountCents: Long,
    val paymentDateMillis: Long,
    val imageLocalUri: String?,
    val synced: Boolean
)
