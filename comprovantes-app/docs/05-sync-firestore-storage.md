# 05 – Sincronização (Firestore + Storage + WorkManager)

## 🎯 Objetivo
Sincronizar os comprovantes salvos no **Room** com o **Firebase**:
- Upload da **imagem** no **Storage**
- Gravação/atualização do **documento** no **Firestore**
- Controle de estado **offline-first** com `synced = true/false`
- Reexecução automática com **WorkManager** em caso de falha

> Pré-requisitos: Etapas 02 (esqueleto), 03 (Room) e 04 (Auth + Household).

---

## 🧭 Fluxo de Alto Nível

1) Usuário cria um comprovante local (`synced = false`).  
2) Disparamos uma **tarefa** (WorkManager) para sincronizar.  
3) No worker:
   - Se houver imagem local **sem URL remota**, faz upload para o **Storage**.
   - Monta um **payload** com os campos e a `imageUrl` (se houver).
   - Faz **upsert** no Firestore: `/households/{householdId}/receipts/{receiptId}`.
   - Marca o registro local como `synced = true` e salva `imageRemoteUrl`.
4) Em caso de erro → `Result.retry()` para tentar novamente.

---

## 🧱 Estrutura envolvida

- `ReceiptEntity` (Room) — tem `imageLocalUri`, `imageRemoteUrl`, `synced`
- `ReceiptRepositoryImpl` — possui `syncPending(householdId)`
- `FirebaseDataSource` — `uploadImageAndGetUrl()` e `upsertReceipt()`
- `SyncWorker` — orquestra a sincronização em background

---

## 🔌 Disparando a sincronização ao salvar

Ao inserir localmente um novo `Receipt`, dispare um `OneTimeWorkRequest`:

```kotlin
val work = OneTimeWorkRequestBuilder<SyncWorker>()
    .setInputData(workDataOf(SyncWorker.KEY_HOUSEHOLD_ID to householdId))
    .setConstraints(
        Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .build()
    )
    .build()
```
---
##☁️ Upload da imagem no Storage

No FirebaseDataSource:
```
suspend fun uploadImageAndGetUrl(householdId: String, localUri: Uri): String {
    val fileName = "${System.currentTimeMillis()}_${localUri.lastPathSegment ?: "receipt.jpg"}"
    val ref = storage.reference.child("households/$householdId/receipts/$fileName")
    ref.putFile(localUri).await()
    return ref.downloadUrl.await().toString()
}
```
---
--
Boas práticas:
Compactar imagem antes do upload (opcional, ver seção “Extras”).
Nomear por receiptId se preferir estabilidade: .../receipts/{receiptId}.jpg.
--
---
##🗄️ Upsert no Firestore

Ainda no FirebaseDataSource:
```
suspend fun upsertReceipt(householdId: String, receiptId: String, data: Map<String, Any?>) {
    firestore.collection("households")
        .document(householdId)
        .collection("receipts")
        .document(receiptId)
        .set(data, SetOptions.merge())
        .await()
}
```
---
--
Campos recomendados do documento:
--
---
```
{
  "id": "string",
  "householdId": "string",
  "description": "string",
  "amountCents": 12345,
  "currency": "BRL",
  "paymentDateMillis": 1730700000000,
  "dueDateMillis": 1731300000000,
  "paidByUserId": "uid",
  "imageUrl": "https://...",
  "createdAtMillis": 1730700000000,
  "updatedAtMillis": 1730701000000
}
```
--
Dica: Manter updatedAtMillis ajuda a resolver conflitos (última gravação vence).
--

### 🔁 Implementação do syncPending(householdId)

No ReceiptRepositoryImpl:
```
override suspend fun syncPending(householdId: String) {
    val unsynced = db.receiptDao().unsynced()
    for (e in unsynced) {
        try {
            var remoteUrl = e.imageRemoteUrl

            // 1) Upload da imagem se necessário
            if (!e.imageLocalUri.isNullOrBlank() && e.imageRemoteUrl.isNullOrBlank()) {
                remoteUrl = remote.uploadImageAndGetUrl(householdId, Uri.parse(e.imageLocalUri))
            }

            // 2) Upsert no Firestore
            val payload = mapOf(
                "id" to e.id,
                "householdId" to e.householdId,
                "description" to e.description,
                "amountCents" to e.amountCents,
                "currency" to e.currency,
                "paymentDateMillis" to e.paymentDateMillis,
                "dueDateMillis" to e.dueDateMillis,
                "paidByUserId" to e.paidByUserId,
                "imageUrl" to remoteUrl,
                "createdAtMillis" to e.createdAtMillis,
                "updatedAtMillis" to System.currentTimeMillis()
            )
            remote.upsertReceipt(householdId, e.id, payload)

            // 3) Marca como sincronizado
            markSynced(e.id, remoteUrl)
        } catch (t: Throwable) {
            // Propositalmente não falha o laço todo; deixa WorkManager tentar novamente
            t.printStackTrace()
        }
    }
}
```

## 🧵 SyncWorker (relembrando)
```
@HiltWorker
class SyncWorker @AssistedInject constructor(
    @Assisted appContext: Context,
    @Assisted params: WorkerParameters,
    private val repository: ReceiptRepository
) : CoroutineWorker(appContext, params) {

    override suspend fun doWork(): Result = try {
        val householdId = inputData.getString(KEY_HOUSEHOLD_ID) ?: return Result.failure()
        repository.syncPending(householdId)
        Result.success()
    } catch (t: Throwable) {
        t.printStackTrace()
        Result.retry()
    }

    companion object {
        const val KEY_HOUSEHOLD_ID = "household_id"
    }
}
```

## 🔐 Segurança (regras e validação)

Firestore Rules: permitir acesso apenas para membros do household (veja Etapa 04).
Storage Rules: mesma lógica para o caminho households/{householdId}/....
Validação de campos: no app, valide amountCents >= 0, datas coerentes, etc.
--
Sugestão: adicione validação no Repository antes do upsert.
--

## 📶 Estratégias Offline & Conflitos

Offline-first: Salve sempre no Room; sincronize depois.
Conflitos: use updatedAtMillis e estratégia Last-Write-Wins (simples para este projeto).
Reprocessamento: mantenha unsynced() e Worker periódico como “rede de segurança”.
--

## 🧪 Testes manuais

Modo avião: adicionar 2–3 comprovantes e reabrir o app → devem persistir no Room.
Voltar conexão: WorkManager roda; conferir no Firestore/Storage.
Dois usuários: inserir no dispositivo A e verificar que B recebeu (próxima Etapa 06).
Falha proposital: desligar internet durante upload → verificar Result.retry().

## 🧰 Extras (opcionais mas úteis)
Compactar imagem antes do upload
Reduz custo e acelera sync:

```
suspend fun compressToCache(context: Context, uri: Uri, maxWidth: Int = 1280): Uri {
    // 1) Ler bitmap via ContentResolver
    // 2) Redimensionar mantendo proporção
    // 3) Salvar JPEG 80% em arquivo temporário e retornar Uri
    // (implementar conforme necessidade do projeto)
    return uri
}
```

Use no repositório antes de uploadImageAndGetUrl.
Guardar householdId no DataStore
Para não depender de variável em memória:
Ao autenticar, salvar householdId em DataStore.
No Worker, se não vier por input, ler do DataStore.
Índices no Firestore
Se buscar por mês/intervalo, crie índices (Console → Firestore → Indexes).

## ✅ Resultado esperado
Comprovantes criados offline aparecem na lista.
Ao reconectar, imagem vai para o Storage e dados para o Firestore.
Registro local é marcado synced = true.
Em falhas, o WorkManager tenta novamente de forma automática.

## ▶️ Próxima etapa

## Etapa 06 – Notificações (Listener e FCM)
Alertar o parceiro quando novos comprovantes forem adicionados (notificação local + push).

WorkManager.getInstance(context).enqueue(work)
