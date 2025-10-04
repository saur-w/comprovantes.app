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

## 💾 Exemplo de DAO
@Dao
interface ReceiptDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(receipt: ReceiptEntity)

    @Query("SELECT * FROM receipts WHERE householdId = :householdId ORDER BY paymentDateMillis DESC")
    fun receiptsByHousehold(householdId: String): Flow<List<ReceiptEntity>>
}

🧠 Boas práticas

Sempre usar Flow para listas observáveis.
Evitar acessos diretos ao banco na UI.
Testar com dados falsos antes da integração com Firebase.
✅ Resultado esperado
O app consegue:
Salvar comprovantes localmente
Exibir lista offline
Atualizar auomaticamente na UI (graças ao Flow)




