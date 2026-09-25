---
name: skill_mvvm_simplificado
description: Use when creating, editing, or reviewing any React Native or Expo application code. This skill enforces the simplified MVVM architecture exactly as taught in the PDM course (Programação para Dispositivos Móveis). MUST be followed whenever creating screens, components, services, entities, or viewmodels. Apply when the user mentions tela, componente, serviço, entidade, ViewModel, hook, autenticação, navegação, estado, ou qualquer código de aplicação móvel. Do NOT allow mixing responsibilities between layers.
---

# MVVM Simplificado — Padrão do Professor (PDM)

## Regra Central

**Todo código de aplicação React Native / Expo DEVE seguir MVVM Simplificado exatamente como ensinado na disciplina PDM.**

Evite o padrão "CR — Codifica & Remenda" (também chamado "Big Tripe"): concentrar tudo — interface, regras de negócio e navegação — em um único arquivo. Isso torna o código difícil de testar, reaproveitar e evoluir.

---

## As 3 Camadas

### Model
Concentra todo o domínio da aplicação. É dividido em:

- **`entities/`** — tipos e entidades puras (ex: `User.ts`)
- **`services/`** — regras de negócio e operações específicas (ex: `AuthService.ts`)
- **`repositories/`** — abstração de acesso a dados (ex: `TaskRepository.ts`)

O Model **não conhece** React, hooks nem nada de UI.

### ViewModel
Implementada como **Custom Hook** (ex: `useLoginViewModel.ts`).

Responsabilidades:
- Gerenciar o estado da tela (`loading`, `error`, dados)
- Expor as ações que a View pode chamar (`handleLogin`, `handleLogout`)
- Chamar os serviços/repositórios do Model

A ViewModel **não contém** elementos de interface (sem JSX, sem componentes visuais).

### View
Representada pelas telas e componentes (arquivos `.tsx`).

Responsabilidades:
- Importar a ViewModel e consumir seu estado e ações
- Renderizar o conteúdo com base no estado
- Estados de UI puros (como valor digitado num campo) podem ficar na View

A View **não contém** lógica de negócio nem chama serviços diretamente.

---

## Estrutura de Pastas Obrigatória

```
src/
├── app/                        ← telas gerenciadas pelo Expo Router
│   ├── _layout.tsx             ← estrutura de navegação principal
│   ├── index.tsx               ← tela de Login (View)
│   └── home.tsx                ← tela Home (View)
├── model/
│   ├── entities/
│   │   └── User.ts             ← entidades e tipos puros
│   ├── services/
│   │   └── AuthService.ts      ← regras de negócio
│   └── repositories/
│       └── (repositórios futuros)
├── viewmodel/
│   └── useLoginViewModel.ts    ← Custom Hook = ViewModel
└── view/
    └── components/             ← componentes visuais reutilizáveis
```

> **Dica do professor:** use `camelCase` para hooks e `PascalCase` para componentes React.
> Exemplo: `useLoginViewModel.ts` e `LoginView.tsx`

---

## Exemplo Completo (exatamente como o professor ensina)

### Etapa 1 — Entidade no Model

```typescript
// src/model/entities/User.ts
export type User = {
  uID: string;
  userName: string;
};
```

### Etapa 2 — Service no Model

```typescript
// src/model/services/AuthService.ts
import { User } from "../entities/user";

export class AuthService {
  async login(email: string, password: string): Promise<User> {
    await new Promise((resolve) => setTimeout(resolve, 1000));
    if (email !== "user@example.com" || password !== "password") {
      throw new Error("invalid credentials");
    }
    return {
      uID: "123",
      userName: "user123",
    };
  }
}
```

### Etapa 3 — ViewModel (Custom Hook)

```typescript
// src/viewmodel/useLoginViewModel.ts
import { useState } from "react";
import { User } from "../model/entities/user";
import { AuthService } from "../model/services/authService";

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

  const service = new AuthService();

  async function handleLogin(email: string, password: string) {
    try {
      setLoading(true);
      setError(null);
      const user: User = await service.login(email, password);
      setUserId(user.uID);
    } catch (err: any) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  return { userId, loading, error, handleLogin };
}
```

### Etapa 4 — View consumindo a ViewModel

```tsx
// src/app/index.tsx
import { router } from "expo-router";
import { useEffect, useState } from "react";
import { Button, StyleSheet, Text, TextInput, View } from "react-native";
import { useLoginViewModel } from "../viewmodel/useLoginViewModel";

const Index = () => {
  // estados de UI puros ficam na View
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  // estado e ações da aplicação vêm da ViewModel
  const { userId, loading, error, handleLogin } = useLoginViewModel();

  useEffect(() => {
    if (userId) {
      router.replace("/home");
    } else {
      setEmail('');
      setPassword('');
    }
  }, [userId, error]);

  if (loading) {
    return <Text>loading...</Text>;
  }

  return (
    <View style={styles.container}>
      <TextInput
        style={styles.input}
        placeholder="email"
        value={email}
        onChangeText={setEmail}
      />
      <TextInput
        style={styles.input}
        placeholder="password"
        value={password}
        onChangeText={setPassword}
      />
      <Button
        title="login"
        onPress={() => handleLogin(email, password)}
      />
      {error && <Text>error: {error}</Text>}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    padding: 24,
  },
  input: {
    borderWidth: 1,
    borderColor: "#ccc",
    borderRadius: 8,
    padding: 12,
    marginBottom: 12,
  },
});

export default Index;
```

---

## O que é estado da aplicação vs. estado de UI?

O professor faz essa distinção importante:

| Estado de UI (fica na View) | Estado da aplicação (fica na ViewModel) |
|---|---|
| Valor digitado num campo de texto | `userId` (usuário autenticado) |
| Aba selecionada | `loading` (operação em andamento) |
| Modal aberto/fechado | `error` (mensagem de erro do negócio) |
| Accordion expandido | Dados carregados da API |

Com o tempo você vai aprimorar a capacidade de diferenciar estes estados.

---

## Regras que NUNCA podem ser quebradas

| ❌ Proibido | ✅ Correto |
|---|---|
| Lógica de autenticação dentro do componente da tela | Criar `AuthService` em `model/services/` |
| Tipo/entidade declarado dentro do arquivo da tela | Criar arquivo em `model/entities/` |
| `fetch()` ou chamada de API direto na View | View chama ViewModel, ViewModel chama Service |
| JSX ou componente visual dentro da ViewModel | ViewModel é um hook puro sem UI |
| Toda a lógica em um único arquivo ("Big Tripe") | Separar em Model, ViewModel e View |

---

## Evolução futura (MVVM Sofisticado)

O professor ensina que, após dominar o MVVM Simplificado, o próximo passo é:

- Extrair interfaces de repositórios para o Model
- Mover implementações concretas para uma camada de **Infraestrutura**
- Transformar ViewModels em classes independentes do React (facilita testes unitários)

Por enquanto, mantenha ViewModels como Custom Hooks — é a abordagem prática ensinada na disciplina.

---

## Sinais de que o código está errado (red flags)

- Você vê `fetch()`, `axios` ou lógica de autenticação dentro de um `.tsx` de tela → **ERRADO**
- Tipo ou entidade declarado dentro do arquivo de tela → **ERRADO**
- A ViewModel importa algo de `react-native` (View, Text, Button etc.) → **ERRADO**
- Uma tela sem Custom Hook correspondente → **ERRADO**
- Toda a lógica de uma funcionalidade em um único arquivo → **padrão "Big Tripe" — ERRADO**

Ao encontrar qualquer red flag, refatore antes de continuar.
