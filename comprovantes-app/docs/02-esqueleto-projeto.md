# 02 – Esqueleto do Projeto

## 🎯 Objetivo
Criar a base do app com arquitetura MVVM, Room, Hilt e Jetpack Compose.

## 🧱 Estrutura do projeto
br.com.comprovantes
├── data/
│   ├── local/
│   ├── remote/
│   └── repository/
├── domain/
│   └── model/
├── ui/
│   ├── screens/
│   └── viewmodel/
├── di/
└── work/

## 🧩 Principais arquivos
- `Receipt.kt` — modelo de dados (domain)
- `ReceiptDao.kt` — interface Room
- `AppDatabase.kt` — configuração Room
- `FirebaseDataSource.kt` — integração remota
- `ReceiptRepositoryImpl.kt` — camada intermediária
- `AddReceiptViewModel.kt` e `ReceiptsListViewModel.kt` — controle de UI
- `NavGraph.kt` — navegação Compose
- `MainActivity.kt` — ponto de entrada

## 🧠 Explicação didática
> O app segue o padrão **MVVM (Model-View-ViewModel)** para separar responsabilidades e facilitar manutenção.
> 
> - **Model:** `Receipt.kt`
> - **ViewModel:** controla a lógica e estado da UI
> - **View (Compose):** exibe dados e interage com o usuário
