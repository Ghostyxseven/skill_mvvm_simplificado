# MVVM Sofisticado — Padrão do Professor (PDM)

Quando a aplicação cresce, o MVVM Simplificado pode ser evoluído para o **MVVM Sofisticado**. O professor de PDM (Programação para Dispositivos Móveis) ensina esta arquitetura em 5 camadas:

## As 5 Camadas

1. **View** – Componentes React Native responsáveis apenas por renderização;
2. **ViewModel** – Hooks que gerenciam estado da View e expõem Actions;
3. **UseCases** – Camada de orquestração de regras de negócio complexas;
4. **Domínio (Model)** – Entidades, interfaces de serviços e repositórios;
5. **Infraestrutura** – Implementações concretas dos Repositórios e Serviços (ex: Firebase, Axios).

---

## Estrutura de Pastas

```
src/
├── app/                        ← Telas (Expo Router)
│   ├── index.tsx               ← View
│   └── home.tsx
├── viewmodel/
│   └── useLoginViewModel.ts    ← ViewModel (Hooks)
├── domain/                     ← Model/Domínio (Não conhece React/Firebase)
│   ├── entities/
│   │   └── User.ts             ← Entidades Puras
│   ├── services/
│   │   └── IAuthService.ts     ← Interfaces de Serviços
│   ├── repositories/
│   │   └── IUserRepository.ts  ← Interfaces de Repositórios
│   └── usecases/
│       ├── IAuthUseCases.ts    ← Interfaces de Casos de Uso
│       └── AuthUseCases.ts     ← Implementação das Regras de Negócio
└── infra/                      ← Infraestrutura
    ├── services/
    │   └── FirebaseAuthService.ts ← Implementação concreta (Firebase)
    └── repositories/
        └── FirestoreUserRepository.ts
```

---

## Exemplo: Fluxo de Autenticação

### 1. Domínio (Model) - Entidades e Interfaces

```typescript
// src/domain/entities/User.ts
export class User {
  constructor(
    public readonly uID: string,
    public readonly userName: string
  ) {}
}

// src/domain/services/IAuthService.ts
import { User } from "../entities/User";
export interface IAuthService {
  login(email: string, password: string): Promise<User>;
}
```

### 2. Casos de Uso (Regras de Negócio)

```typescript
// src/domain/usecases/AuthUseCases.ts
import { IAuthService } from "../services/IAuthService";
import { User } from "../entities/User";

export interface IAuthUseCases {
  login(email: string, password: string): Promise<User>;
}

export class AuthUseCases implements IAuthUseCases {
  // Injeção de dependência baseada em abstração
  constructor(private authService: IAuthService) {}

  async login(email: string, password: string): Promise<User> {
    if (!email || !password) {
      throw new Error("Validation Error: Email and password required");
    }
    // Aqui vai orquestração (salvar em repositório, disparar analytics, etc)
    return this.authService.login(email, password);
  }
}
```

### 3. Infraestrutura (Implementação Concreta)

```typescript
// src/infra/services/FirebaseAuthService.ts
import { IAuthService } from "../../domain/services/IAuthService";
import { User } from "../../domain/entities/User";

export class FirebaseAuthService implements IAuthService {
  async login(email: string, password: string): Promise<User> {
    // Código real do Firebase auth iria aqui
    return new User("123", "User Name");
  }
}
```

### 4. ViewModel

```typescript
// src/viewmodel/useLoginViewModel.ts
import { useState } from "react";
import { IAuthUseCases } from "../domain/usecases/AuthUseCases";
import { User } from "../domain/entities/User";

// A ViewModel agora recebe o UseCase (geralmente injetado via Factory ou Provider)
export function useLoginViewModel(authUseCases: IAuthUseCases) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function handleLogin(email: string, password: string) {
    try {
      setLoading(true);
      setError(null);
      const user = await authUseCases.login(email, password);
      // Navegar ou salvar estado
    } catch (err: any) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  return { loading, error, handleLogin };
}
```

### 5. View (Tela)

```tsx
// src/app/index.tsx
import { useState } from "react";
import { View, TextInput, Button, Text } from "react-native";
import { useLoginViewModel } from "../viewmodel/useLoginViewModel";

// Factories são úteis para injetar as dependências concretas
import { AuthUseCases } from "../domain/usecases/AuthUseCases";
import { FirebaseAuthService } from "../infra/services/FirebaseAuthService";

const authService = new FirebaseAuthService();
const authUseCases = new AuthUseCases(authService);

export default function LoginScreen() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  
  const { loading, error, handleLogin } = useLoginViewModel(authUseCases);

  return (
    <View>
      <TextInput value={email} onChangeText={setEmail} placeholder="Email" />
      <TextInput value={password} onChangeText={setPassword} placeholder="Password" />
      <Button title="Login" onPress={() => handleLogin(email, password)} />
      {loading && <Text>Carregando...</Text>}
      {error && <Text>{error}</Text>}
    </View>
  );
}
```

## Resumo das Responsabilidades no Sofisticado

- **Domínio**: Contém as **regras de negócio**, não sabe de onde vêm os dados (API, Firebase, Banco de dados local) e não sabe como eles são exibidos (React).
- **Infraestrutura**: Sabe **como** buscar/salvar os dados usando as bibliotecas específicas (Axios, Firebase SDK, SQLite).
- **ViewModel**: Converte os erros/dados do negócio para estados que a View entenda.
- **View**: Apenas exibe o que a ViewModel manda e avisa a ViewModel quando o usuário interage.
