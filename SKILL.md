---
name: skill_mvvm_simplificado
description: Use when creating, editing or reviewing React Native / Expo (Expo Router) application code - screens, components, services, entities, repositories, use cases, ViewModels/hooks, authentication, navigation or state. Enforces the MVVM architecture taught in the PDM course (Programação para Dispositivos Móveis), in its Simplified (3 layers) and Sophisticated (use cases + infra + factories) versions. Do NOT allow mixing responsibilities between layers.
---

# MVVM — Padrão da disciplina PDM

## Regra Central

Todo código de aplicação React Native / Expo DEVE separar **View**, **ViewModel** e **Model**. Evite o "CR — Codifica & Remenda" (Big Tripe): interface, regras de negócio, chamadas de API e navegação num único arquivo.

Existem duas versões. Escolha pela tabela:

| Situação | Versão | Referência |
|---|---|---|
| Projeto novo, poucas telas, regra de negócio simples | **Simplificado** (3 camadas) | este arquivo |
| Há regras de negócio além de CRUD, troca de backend prevista, ou o usuário pediu testes/"sofisticado" | **Sofisticado** (UseCases + Infra + Factories) | `examples/mvvm_sofisticado.md` |

Se o projeto já usa uma versão, **continue nela**. Não misture as duas na mesma feature. Migre para o sofisticado quando a ViewModel começar a ter `if` de regra de negócio ou precisar trocar o backend.

---

## Convenções (valem para as duas versões)

- **Navegação:** Expo Router. Em `src/app` ficam **somente telas e `_layout.tsx`**. Componentes, hooks, dados e tipos ficam em outras pastas de `src/`.
- **Imports:** use o alias `@/` (aponta para `src/`), nunca `../../..`.
- **Nomes de arquivo:** o import deve ter a **mesma caixa** do arquivo (`User.ts` → `@/model/entities/User`). Em `src/app`, arquivos de rota em minúsculas com hífen (`meus-favoritos.tsx`).
- **Componentes:** `const Tela = () => {...}; export default Tela;` (arrow function, `export default` separado). Telas e rotas exigem `export default`; componentes em `view/components` usam `export` nomeado.
- **Estilo:** `StyleSheet.create` no **fim** do arquivo, com nomes que descrevem o papel (`card`, `title`), não a aparência.
- **Botões:** prefira `Pressable`. `Button` só em protótipo rápido (não aceita `style`).
- **Senha:** todo `TextInput` de senha leva `secureTextEntry`.
- **Identificadores** em inglês (`loading`, `error`, `handleLogin`); textos de UI em português.
- **ViewModel = Custom Hook** `useXxxViewModel` (camelCase, começa com `use`). Componentes em PascalCase.

---

## Versão Simplificada — 3 camadas

### Model (`src/model/`)
Domínio da aplicação, sem React/Expo/hooks/JSX.
- `entities/` — tipos e entidades puras (`User.ts`)
- `services/` — regras de negócio e operações (`AuthService.ts`, implementação concreta)
- `repositories/` — acesso a dados (`TaskRepository.ts`)

### ViewModel (`src/viewmodel/`)
Custom Hook. Gerencia o estado da tela (`loading`, `error`, dados), expõe ações (`handleLogin`) e chama o Model. **Sem JSX nem componentes visuais.**

### View (`src/app/` e `src/view/components/`)
Renderiza o estado e dispara ações da ViewModel. Pode guardar **estado de UI puro** (texto digitado, aba selecionada). **Sem** regra de negócio, `fetch`/`axios`, nem `try/catch`.

### Estrutura de pastas
```
src/
├── app/                        ← Views: telas e rotas (Expo Router)
│   ├── _layout.tsx
│   ├── index.tsx
│   └── home.tsx
├── model/
│   ├── entities/User.ts
│   ├── services/AuthService.ts
│   └── repositories/
├── viewmodel/useLoginViewModel.ts
└── view/components/            ← componentes reutilizáveis
```

---

## Fluxo de trabalho obrigatório

Ordem: **Model → ViewModel → View**. Em cada etapa: criou → **verificou** → deu erro? consertou → verificou de novo → só então avança.

"Verificou" significa executar algo de verdade, nunca apenas declarar que funciona:
1. **Model:** rode `npx tsc --noEmit` e teste o service (Jest) cobrindo sucesso **e** falha.
2. **ViewModel:** teste o hook com `renderHook` (`@testing-library/react-native`): `loading` liga/desliga, `error` é preenchido na falha, ação funciona.
3. **View:** rode o app (`npx expo start`) ou teste de componente e confira os estados: loading mostra indicador, erro aparece, vazio tem mensagem.

Se o projeto não tem Jest configurado, rode ao menos `npx tsc --noEmit` e o lint, e diga ao usuário o que **não** foi testado.

---

## Estados da tela

Toda tela que carrega dados trata os 4 estados: **loading**, **erro** (com retry quando fizer sentido), **vazio** e **sucesso**. Detalhes em `references/padroes_estado.md`.

| Estado de UI (fica na View) | Estado da aplicação (fica na ViewModel) |
|---|---|
| Texto digitado num campo | `userId`, lista de dados |
| Aba selecionada | `loading`, `error` |

---

## Exemplo completo (Simplificado)

```typescript
// src/model/entities/User.ts
export type User = {
  uID: string;
  userName: string;
};
```

```typescript
// src/model/services/AuthService.ts
import { User } from "@/model/entities/User";

export class AuthService {
  async login(email: string, password: string): Promise<User> {
    await new Promise((resolve) => setTimeout(resolve, 1000));
    if (email !== "user@example.com" || password !== "password") {
      throw new Error("Credenciais inválidas");
    }
    return { uID: "123", userName: "user123" };
  }
}
```

```typescript
// src/viewmodel/useLoginViewModel.ts
import { useState } from "react";
import { AuthService } from "@/model/services/AuthService";

const authService = new AuthService(); // criado uma vez, fora do render

export type LoginState = {
  userId: string | null;
  loading: boolean;
  error: string | null;
};

export type LoginActions = {
  handleLogin: (email: string, password: string) => Promise<void>;
};

export function useLoginViewModel(): LoginState & LoginActions {
  const [userId, setUserId] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function handleLogin(email: string, password: string) {
    try {
      setLoading(true);
      setError(null);
      const user = await authService.login(email, password);
      setUserId(user.uID);
    } catch (err) {
      setError(err instanceof Error ? err.message : "Falha inesperada.");
    } finally {
      setLoading(false);
    }
  }

  return { userId, loading, error, handleLogin };
}
```

```tsx
// src/app/index.tsx
import { router } from "expo-router";
import { useEffect, useState } from "react";
import { ActivityIndicator, Pressable, StyleSheet, Text, TextInput, View } from "react-native";
import { useLoginViewModel } from "@/viewmodel/useLoginViewModel";

const Login = () => {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const { userId, loading, error, handleLogin } = useLoginViewModel();

  useEffect(() => {
    if (userId) router.replace("/home");
  }, [userId]);

  if (loading) return <ActivityIndicator style={styles.loading} />;

  return (
    <View style={styles.container}>
      <TextInput style={styles.input} placeholder="E-mail" value={email} onChangeText={setEmail} autoCapitalize="none" />
      <TextInput style={styles.input} placeholder="Senha" value={password} onChangeText={setPassword} secureTextEntry />
      <Pressable style={styles.button} onPress={() => handleLogin(email, password)}>
        <Text style={styles.buttonText}>Entrar</Text>
      </Pressable>
      {error && <Text style={styles.error}>{error}</Text>}
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: "center", padding: 24, gap: 12 },
  loading: { flex: 1 },
  input: { borderWidth: 1, borderColor: "#ccc", borderRadius: 8, padding: 12 },
  button: { backgroundColor: "#4630EB", borderRadius: 8, padding: 12, alignItems: "center" },
  buttonText: { color: "#fff", fontWeight: "600" },
  error: { color: "red" },
});

export default Login;
```

---

## ✅ Checklist de autocorreção

Antes de entregar, revise o que escreveu:

- [ ] A **View** tem `if` de regra de negócio, `try/catch`, `fetch`/`axios`? → mova para ViewModel/Model.
- [ ] A **ViewModel** importa `View`, `Text` ou qualquer JSX? → remova.
- [ ] O **Model** importa algo de `react`, `react-native` ou `expo-*`? → remova.
- [ ] Há tela em `src/app` sem ViewModel correspondente? → crie o hook.
- [ ] Há algo em `src/app` que não é tela nem layout? → mova para fora.
- [ ] Os imports usam `@/` e a caixa do nome do arquivo está correta?
- [ ] Serviços são criados **uma vez** (módulo ou factory), não a cada render?
- [ ] A tela trata loading, erro e vazio? A senha usa `secureTextEntry`?
- [ ] Rodou `tsc`/testes/app (ou avisou o que não foi verificado)?

Se achar qualquer violação, **corrija antes de declarar pronto**.

---

## Evolução (MVVM Sofisticado)

Veja `examples/mvvm_sofisticado.md` (UseCases, interfaces no Model, Infra, Factories, erros de domínio), `references/injecao_dependencias.md` e `references/padroes_estado.md`. Ao migrar: o service concreto vira interface em `model/services/`, a implementação vai para `infra/`, a regra de negócio vai para um UseCase e a ViewModel passa a receber o UseCase por parâmetro.

## Divergências em relação ao livro da disciplina

Esta skill segue o livro, com estas correções deliberadas: caixa dos imports corrigida; UseCase único e consistente (`IAuthUseCases`); pasta `factories/` no lugar de `di/`; UseCases dentro de `model/usecases/`; erros de domínio explícitos; Factory sem recriar dependências a cada render; `Pressable` e `secureTextEntry`. Se o usuário pedir a estrutura literal do livro, siga o pedido dele.
