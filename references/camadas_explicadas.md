# 📚 O que é cada camada? (Explicação Didática)

O papel de cada pasta, em linguagem simples. As camadas 4 e 5 aparecem só no **MVVM Sofisticado**; no Simplificado, o `model/` tem entidades, services e repositories concretos.

---

### 🎨 1. View (`app/` e `view/components/`) — "A Vitrine"
É a tela: arquivos `.tsx` com botões, textos e listas.
- **Personalidade:** "burra" e obediente.
- **O que faz:** olha a ViewModel e desenha: carregando → spinner; erro → mensagem; clique → avisa a ViewModel.
- **Proibido:** regra de negócio (`if (senha.length < 6)`), `fetch`/`axios`, `try/catch`, importar `infra`.
- **Atenção:** em `app/` só ficam telas e `_layout.tsx` (Expo Router). Componentes vão em `view/components/`.

---

### 🧠 2. ViewModel (`viewmodel/`) — "O Gerente da Tela"
Um Custom Hook (`useLoginViewModel.ts`).
- **O que faz:** guarda `loading`, `error` e os dados. Quando a View avisa "clicou em Login", chama o Model (Simplificado) ou o UseCase (Sofisticado).
- **Proibido:** componentes visuais (`Text`, `View`) e, no Sofisticado, regra de negócio.

---

### 👑 3. Model / Domínio (`model/`) — "O Chefe das Regras"
Não sabe que React existe.
- `entities/` — o formato das coisas (`User.ts`).
- `errors/` — erros de domínio (`ValidationError`, `AuthFailedError`). *(Sofisticado)*
- `services/` e `repositories/` — **Simplificado:** implementações concretas. **Sofisticado:** só contratos (`IAuthService`), "preciso de alguém que autentique", sem dizer como.
- `usecases/` — *(Sofisticado)* regras e orquestração: "valida os dados, autentica, salva o usuário".

---

### 👷 4. Infraestrutura (`infra/`) — "O Operário" *(Sofisticado)*
Conversa com o mundo externo (Firebase, Axios, SQLite, GPS).
- **O que faz:** cumpre os contratos do Model. `FirebaseAuthService` chama o SDK, converte a resposta em `User` e **traduz erros do SDK em erros de domínio**.
- **Vantagem:** trocar Firebase por Node.js = reescrever só `infra/`.

---

### 🏭 5. Factories (`factories/`) — "O Montador" *(Sofisticado)*
A View não deve saber que o Firebase existe. A Factory pega o Operário (Infra), entrega ao UseCase, injeta o UseCase na ViewModel e devolve o hook pronto.
- **Exemplo:** `useLoginViewModelFactory()` (começa com `use` porque chama um hook).
- As dependências são criadas **uma vez**, fora do componente.
