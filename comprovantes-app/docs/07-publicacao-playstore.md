# 07 – Publicação na Play Store

## 🎯 Objetivo
Gerar o build assinado, preparar os requisitos de publicação e lançar o **ComprovantesApp** na **Google Play Store**, com política de privacidade e informações obrigatórias.

---

## 🧭 Etapas Principais

| Etapa | Descrição | Status |
|-------|------------|--------|
| 1 | Gerar build assinado (`.aab`) | ⏳ |
| 2 | Criar conta de desenvolvedor Google Play | ⏳ |
| 3 | Escrever descrição e preparar imagens | ⏳ |
| 4 | Adicionar política de privacidade | ⏳ |
| 5 | Enviar build para análise | ⏳ |
| 6 | Publicar o app | ⏳ |

---

## ⚙️ 1. Gerar build assinado

No Android Studio:
1. Vá em **Build → Generate Signed App Bundle / APK...**
2. Escolha **Android App Bundle (.aab)** → clique em **Next**
3. Crie ou use uma **keystore**:
   - Local seguro (ex: `C:\keystores\comprovantes.jks`)
   - Defina senha e alias → **guarde-as com segurança!**
4. Clique em **Next → Finish**
Após compilar, o arquivo `.aab` ficará em:
app/release/app-release.aab


Esse é o arquivo que você enviará para o Google Play Console.

---

## 💳 2. Criar conta de desenvolvedor Google Play

1. Acesse: [https://play.google.com/console](https://play.google.com/console)
2. Faça login com uma conta Google.
3. Pague a taxa única de **US$ 25**.
4. Complete o cadastro do desenvolvedor (nome, e-mail, país, etc).

---

## 🧾 3. Informações obrigatórias do aplicativo

Na tela do Console (após criar o app):
- **Nome do app:** ComprovantesApp  
- **Breve descrição:**  
  “Guarde e compartilhe comprovantes de pagamento com segurança.”  
- **Descrição completa:**  
  > Um aplicativo para casais e famílias organizarem comprovantes de pagamentos mensais.  
  > Permite armazenar recibos, sincronizar automaticamente e enviar notificações quando novas contas são pagas.

- **Categoria:** Ferramentas / Finanças  
- **Tipo:** Aplicativo  
- **Ícone:** 512×512 PNG  
- **Capturas de tela:**  
  - Tela inicial  
  - Lista de comprovantes  
  - Tela de envio / upload  
  - Tela de login  

---

## ⚖️ 4. Política de Privacidade

Crie o arquivo `docs/politica-privacidade.md` no repositório com algo como:

```md
# Política de Privacidade – ComprovantesApp

O ComprovantesApp armazena dados locais no dispositivo (Room) e sincroniza dados financeiros com o Firebase (Auth, Firestore e Storage).

- Nenhum dado é compartilhado com terceiros.
- Apenas membros do mesmo household têm acesso aos comprovantes compartilhados.
- O usuário pode solicitar exclusão total dos dados enviando um e-mail para [email de suporte].

Data de vigência: 01/10/2025
Depois, publique esse arquivo (ou página) no GitHub Pages e copie o link para colar no campo “Privacy Policy URL” da Play Store.

🧩 5. Assinar e enviar o build

No Google Play Console:

Clique em Criar nova versão → Upload App Bundle (.aab)

Adicione release notes (ex: “Versão inicial de testes”).

Salve e envie para Internal Testing (teste interno) ou Closed Testing (beta).

🧪 6. Testar internamente

Crie uma lista de testadores:

Adicione e-mails das contas Google dos participantes.

Compartilhe o link de instalação (Google gerará automaticamente).

Peça feedback sobre login, sincronização e notificações.

🌍 7. Publicar para todos

Após validar o app:

Vá em Produção → Criar nova versão

Suba o .aab final

Clique em Revisar → Publicar

O app entrará em revisão do Google, que pode levar de 1 a 5 dias úteis.

📦 8. Checklist final
Item	Verificado
Build .aab assinado e testado	✅
Política de privacidade publicada	✅
Ícone e screenshots enviados	✅
Descrição completa e curta preenchidas	✅
Regras do Firebase configuradas	✅
Teste interno aprovado	✅
💡 Dicas adicionais

Mantenha o versCode e versionName atualizados em build.gradle:

defaultConfig {
    versionCode 3
    versionName "1.2.0"
}


Atualize a descrição e screenshots a cada nova versão.

Use o Google Play Console → Estatísticas para acompanhar downloads e avaliações.

Crie uma página simples de suporte ou contato (GitHub Pages ou Notion).

✅ Resultado esperado

Aplicativo publicado na Play Store.

Política de privacidade válida.

Versões futuras facilmente distribuídas.

Usuários podem baixar e usar o app com login, sincronização e notificações.

🏁 Fim do Projeto 🎉

Parabéns! Você finalizou todas as etapas:

Setup do ambiente

Estrutura do app

Banco local (Room)

Autenticação (Firebase)

Sincronização (Firestore + Storage)

Notificações (Listener + FCM)

Publicação Play Store

✨ O ComprovantesApp está pronto para o mundo!


---

Com esse arquivo, você fecha **toda a documentação oficial do projeto** — pronta para ensinar ou publicar no GitHub.  




Após compilar, o arquivo `.aab` ficará em:
