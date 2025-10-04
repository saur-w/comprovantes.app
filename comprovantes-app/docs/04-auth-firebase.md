# 04 – Autenticação Firebase + Household

## 🎯 Objetivo
Habilitar **login/registro com e-mail e senha** usando Firebase Authentication e criar a estrutura de **household** (conta compartilhada) no Firestore para que duas pessoas vejam os mesmos comprovantes.

---

## 🧩 O que vamos fazer
1. Ativar **Email/Password** no Firebase Auth.  
2. Implementar **registro** e **login**.  
3. Criar documento `/users/{uid}` no Firestore ao registrar.  
4. Criar (ou vincular) um **household** com `members: [uid1, uid2]`.  
5. Armazenar o `householdId` para filtrar os comprovantes.  
6. Regras básicas de segurança para Firestore/Storage.

> **Pré-requisito:** Projeto já conectado ao Firebase e com Hilt configurado (Etapas 01 e 02).

---

## ⚙️ Ativar método de login no Console
No **Firebase Console → Authentication → Sign-in method**  
- Ative **Email/Password**.

---

## 🗂️ Estrutura de dados (Firestore)
/users/{uid}
{
uid: string,
email: string,
displayName: string,
households: [householdId]
}
/households/{householdId}
{
members: [uid1, uid2],
createdAtMillis: number
}

/households/{householdId}/receipts/{receiptId}
{ ... (mesma estrutura da etapa Room, com imageUrl opcional) }


---

## 🧱 Regras de segurança (rascunho)
Ajuste conforme sua necessidade (coloque em **Firestore Rules**):

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{db}/documents {
    function isMember(householdId) {
      return request.auth != null
        && request.auth.uid in get(/databases/$(db)/documents/households/$(householdId)).data.members;
    }

    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }

    match /households/{householdId} {
      allow read, write: if isMember(householdId);

      match /receipts/{receiptId} {
        allow read, write: if isMember(householdId);
      }
    }
  }
}

## E em Storage Rules (imagens de comprovantes), algo como:
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /households/{householdId}/{allPaths=**} {
      allow read, write: if request.auth != null
        && request.auth.uid in firestore.get(/databases/(default)/documents/households/$(householdId)).data.members;
    }
  }
}

# Importante: As regras acima são um ponto de partida. Teste com 2 usuários para garantir que apenas membros do household conseguem ler/escrever.


## 🧩 Camada de Autenticação (Data Source)

Crie um arquivo data/remote/AuthDataSource.kt:

package br.com.comprovantes.data.remote

import com.google.firebase.auth.FirebaseAuth
import com.google.firebase.firestore.FirebaseFirestore
import kotlinx.coroutines.tasks.await

class AuthDataSource(
    private val auth: FirebaseAuth,
    private val firestore: FirebaseFirestore
) {
    fun currentUserId(): String? = auth.currentUser?.uid

    suspend fun register(email: String, password: String, displayName: String): String {
        val result = auth.createUserWithEmailAndPassword(email, password).await()
        val uid = result.user?.uid ?: throw IllegalStateException("UID inválido")

        // Cria /users/{uid}
        firestore.collection("users").document(uid).set(
            mapOf(
                "uid" to uid,
                "email" to email,
                "displayName" to displayName,
                "households" to emptyList<String>()
            )
        ).await()

        return uid
    }

    suspend fun login(email: String, password: String): String {
        val result = auth.signInWithEmailAndPassword(email, password).await()
        return result.user?.uid ?: throw IllegalStateException("UID inválido")
    }

    fun logout() {
        auth.signOut()
    }
}

## 🧰 DI (Hilt) – Provedores
No seu di/AppModule.kt, registre os data sources:
@Provides @Singleton
fun provideAuthDataSource(
    auth: FirebaseAuth,
    firestore: FirebaseFirestore
) = AuthDataSource(auth, firestore)

@Provides @Singleton
fun provideHouseholdDataSource(
    firestore: FirebaseFirestore
) = HouseholdDataSource(firestore)

##🧠 ViewModel de Autenticação

Crie ui/viewmodel/AuthViewModel.kt:

package br.com.comprovantes.ui.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import br.com.comprovantes.data.remote.AuthDataSource
import br.com.comprovantes.data.remote.HouseholdDataSource
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch
import javax.inject.Inject

@HiltViewModel
class AuthViewModel @Inject constructor(
    private val auth: AuthDataSource,
    private val households: HouseholdDataSource
) : ViewModel() {

    data class UiState(
        val loading: Boolean = false,
        val error: String? = null,
        val uid: String? = null,
        val householdIds: List<String> = emptyList()
    )

    private val _state = MutableStateFlow(UiState())
    val state = _state.asStateFlow()

    fun login(email: String, password: String) {
        viewModelScope.launch {
            _state.value = _state.value.copy(loading = true, error = null)
            try {
                val uid = auth.login(email, password)
                val hhs = households.getUserHouseholds(uid)
                _state.value = UiState(loading = false, uid = uid, householdIds = hhs)
            } catch (t: Throwable) {
                _state.value = UiState(error = t.message)
            }
        }
    }

    fun register(email: String, password: String, displayName: String) {
        viewModelScope.launch {
            _state.value = _state.value.copy(loading = true, error = null)
            try {
                val uid = auth.register(email, password, displayName)
                // por padrão: cria household novo para o usuário
                val hh = households.createHousehold(uid)
                _state.value = UiState(loading = false, uid = uid, householdIds = listOf(hh))
            } catch (t: Throwable) {
                _state.value = UiState(error = t.message)
            }
        }
    }

    fun joinHousehold(householdId: String) {
        val uid = _state.value.uid ?: return
        viewModelScope.launch {
            _state.value = _state.value.copy(loading = true, error = null)
            try {
                households.joinHousehold(uid, householdId)
                val hhs = households.getUserHouseholds(uid)
                _state.value = _state.value.copy(loading = false, householdIds = hhs)
            } catch (t: Throwable) {
                _state.value = _state.value.copy(loading = false, error = t.message)
            }
        }
    }

    fun logout() = auth.logout()
}

##🎨 Tela simples de Login/Registro (Compose)

Crie ui/screens/AuthScreen.kt (simples e didática):
package br.com.comprovantes.ui.screens

import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.hilt.navigation.compose.hiltViewModel
import br.com.comprovantes.ui.viewmodel.AuthViewModel

@Composable
fun AuthScreen(
    onAuthenticated: (householdId: String) -> Unit,
    vm: AuthViewModel = hiltViewModel()
) {
    val state by vm.state.collectAsState()

    var email by remember { mutableStateOf("") }
    var pass by remember { mutableStateOf("") }
    var name by remember { mutableStateOf("") }
    var joinCode by remember { mutableStateOf("") } // householdId

    Column(Modifier.padding(16.dp)) {
        Text("Autenticação", style = MaterialTheme.typography.headlineSmall)
        Spacer(Modifier.height(12.dp))

        OutlinedTextField(value = email, onValueChange = { email = it }, label = { Text("E-mail") }, modifier = Modifier.fillMaxWidth())
        Spacer(Modifier.height(8.dp))
        OutlinedTextField(value = pass, onValueChange = { pass = it }, label = { Text("Senha") }, modifier = Modifier.fillMaxWidth())

        Spacer(Modifier.height(12.dp))
        Button(onClick = { vm.login(email, pass) }, enabled = !state.loading) {
            Text("Entrar")
        }

        Divider(Modifier.padding(vertical = 16.dp))

        Text("Novo por aqui?")
        OutlinedTextField(value = name, onValueChange = { name = it }, label = { Text("Seu nome") }, modifier = Modifier.fillMaxWidth())
        Spacer(Modifier.height(8.dp))
        Button(onClick = { vm.register(email, pass, name) }, enabled = !state.loading) {
            Text("Criar conta + Household")
        }

        Spacer(Modifier.height(16.dp))
        Text("Ou entrar em um Household existente (código):")
        OutlinedTextField(value = joinCode, onValueChange = { joinCode = it }, label = { Text("householdId") }, modifier = Modifier.fillMaxWidth())
        Spacer(Modifier.height(8.dp))
        Button(onClick = { vm.joinHousehold(joinCode) }, enabled = !state.loading && joinCode.isNotBlank()) {
            Text("Entrar no Household")
        }

        if (state.error != null) {
            Spacer(Modifier.height(12.dp))
            Text("Erro: ${state.error}", color = MaterialTheme.colorScheme.error)
        }

        // Quando autenticado e com household, navegue para a lista
        if (state.uid != null && state.householdIds.isNotEmpty()) {
            // Escolha simples: usar o primeiro household
            LaunchedEffect(state.householdIds) {
                onAuthenticated(state.householdIds.first())
            }
        }
    }
}
##Dica didática: Em projetos reais você pode ter uma tela separada para escolher qual household entrar (se houver mais de um). Aqui simplificamos.

#🔗 Integração com a Navegação

No NavGraph.kt, adicione uma rota de auth e passe o householdId para as telas:
// exemplo de ideia (adapte ao seu NavGraph existente)
composable("auth") {
    AuthScreen(onAuthenticated = { householdId ->
        // guarde o householdId em um ViewModel compartilhado ou no DataStore
        nav.navigate("list")
    })
}

#Onde guardar householdId:
- Simples: ViewModel de sessão (enquanto o app estiver aberto).
- Persistente: DataStore para reabrir o app já logado no mesmo household.

🧪 Testes manuais recomendados

Criar usuário A → verificar /users/{uidA} e household criado.

Criar usuário B → join no mesmo householdId → verificar members: [uidA, uidB].

Confirmar que A e B veem os mesmos comprovantes (na etapa 05).

Validar regras: usuário C (não membro) não consegue ler o household.

✅ Resultado esperado

Login e registro funcionando.

Documento /users/{uid} criado com households.

Household criado no registro e com join por código.

householdId disponível para filtrar e sincronizar comprovantes.

▶️ Próxima etapa

Etapa 05 – Sincronização (Firestore + Storage + WorkManager)
Enviar comprovantes locais para a nuvem e manter ambos os usuários sincronizados.


---
