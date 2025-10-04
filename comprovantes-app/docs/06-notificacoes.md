# 06 – Notificações (Listener e Firebase Cloud Messaging)

## 🎯 Objetivo
Adicionar **notificações em tempo real** para avisar o parceiro quando novos comprovantes forem adicionados, usando:
- **Listener** do Firestore (atualizações em tempo real)
- **Notificações locais** dentro do app
- (Opcional) **Firebase Cloud Messaging (FCM)** para push notifications quando o app estiver fechado.

---

## 🔍 Tipos de Notificação

| Tipo | Quando ocorre | Requer servidor? | Complexidade |
|------|----------------|------------------|---------------|
| Listener Firestore | Quando há mudanças na coleção `receipts` | ❌ Não | ⭐ Simples |
| FCM Push | Quando o app está fechado / em background | ✅ Sim (Cloud Function) | 🔥 Avançado |

---

## 🧩 Listener em tempo real (modo simples)

Adicione um **snapshotListener** no Firestore para detectar novos comprovantes adicionados ao `household`:

```kotlin
firestore.collection("households")
    .document(householdId)
    .collection("receipts")
    .addSnapshotListener { snapshot, _ ->
        snapshot?.documentChanges?.forEach { change ->
            if (change.type == DocumentChange.Type.ADDED) {
                val data = change.document.data
                val description = data["description"] as? String ?: "Comprovante adicionado"
                showLocalNotification("Novo comprovante", description)
            }
        }
    }

💡 Coloque esse listener dentro do ReceiptsListViewModel ou de um ListenerService que inicia ao abrir o app.

## 🔔 Função para exibir notificação local

Crie uma função auxiliar em NotificationUtils.kt:

fun Context.showLocalNotification(title: String, message: String) {
    val channelId = "receipts"
    val nm = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

    // Cria canal no Android 8.0+
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        val channel = NotificationChannel(channelId, "Comprovantes", NotificationManager.IMPORTANCE_DEFAULT)
        nm.createNotificationChannel(channel)
    }

    val notification = NotificationCompat.Builder(this, channelId)
        .setSmallIcon(R.drawable.ic_notification)
        .setContentTitle(title)
        .setContentText(message)
        .setPriority(NotificationCompat.PRIORITY_DEFAULT)
        .setAutoCancel(true)
        .build()

    nm.notify(Random.nextInt(), notification)
}
📱 Permissão no Android 13+

Adicione a permissão para notificações no AndroidManifest.xml:

<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />


E, no MainActivity, solicite dinamicamente (a partir do Android 13):

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
    requestPermissions(arrayOf(Manifest.permission.POST_NOTIFICATIONS), 0)
}

🚀 Firebase Cloud Messaging (modo avançado - opcional)

Se quiser notificar mesmo com o app fechado, use FCM + Cloud Functions.

Passo 1. Ativar o FCM no Console

Em Firebase Console → Cloud Messaging → Get Started

Passo 2. Registrar o token do usuário

No app (por exemplo, dentro do AuthViewModel):

FirebaseMessaging.getInstance().token.addOnSuccessListener { token ->
    firestore.collection("users").document(uid)
        .update("fcmToken", token)
}

Passo 3. Criar Cloud Function no Firebase

Crie o arquivo index.js dentro da pasta functions/ do projeto Firebase:

const functions = require("firebase-functions");
const admin = require("firebase-admin");
admin.initializeApp();

exports.notifyNewReceipt = functions.firestore
    .document("households/{householdId}/receipts/{receiptId}")
    .onCreate(async (snap, context) => {
        const data = snap.data();
        const householdId = context.params.householdId;

        // Busca todos os membros do household
        const hhDoc = await admin.firestore().doc(`households/${householdId}`).get();
        const members = hhDoc.data().members;

        // Pega tokens dos usuários
        const usersSnap = await admin.firestore()
            .collection("users")
            .where("uid", "in", members)
            .get();

        const tokens = [];
        usersSnap.forEach(doc => {
            const t = doc.data().fcmToken;
            if (t) tokens.push(t);
        });

        // Envia notificação
        const message = {
            notification: {
                title: "Novo comprovante adicionado!",
                body: `${data.description || "Um comprovante foi enviado."}`
            },
            tokens: tokens
        };

        return admin.messaging().sendMulticast(message);
    });

Passo 4. Deploy da Function

Execute:

firebase deploy --only functions


Agora, sempre que um comprovante for criado, o outro membro receberá uma notificação push.

🧪 Testes manuais
Cenário	Esperado
Usuário A adiciona comprovante	Usuário B recebe notificação local (listener)
Usuário B fecha o app	Recebe notificação FCM push (se configurado)
Household com 2 membros	Ambos veem notificações ao mesmo tempo
Household isolado	Nenhuma notificação enviada a terceiros
🧠 Dicas de UX

Evite excesso de notificações → notifique por grupo de comprovantes se possível.

Mostre uma tela de “Novos comprovantes” quando o usuário tocar na notificação.

Permita desligar notificações no menu Configurações.

✅ Resultado esperado

Listener ativo detecta novos comprovantes em tempo real.

Notificações locais funcionam no app aberto.

(Opcional) FCM envia push notifications quando o app estiver fechado.

▶️ Próxima etapa

Etapa 07 – Publicação na Play Store
Gerar build assinado, adicionar política de privacidade e subir para a loja.


---

Esse arquivo completa a **Etapa 06** e prepara o app para alertar o outro membro do casal sempre que novos comprovantes forem adicionados.  
