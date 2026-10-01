# MVVM Sofisticado — Padrão PDM

Evolução do MVVM Simplificado para aplicações maiores: a regra de negócio vai para **UseCases**, o acesso externo vai para **Infra**, e as peças são montadas por **Factories** (injeção de dependências). Dependências sempre apontam para abstrações.

## Camadas

1. **View** — renderiza estado e dispara ações.
2. **ViewModel** — hook que guarda estado da tela e traduz erros em texto. Sem regra de negócio.
3. **UseCases** — orquestram regras de negócio (dentro de `model/usecases/`).
4. **Model (Domínio)** — entidades, erros de domínio e **interfaces** de serviços/repositórios/usecases. Puro: sem React, Expo ou Firebase.
5. **Infra** — implementações concretas (Firebase, Axios, SQLite) que **traduzem erros técnicos em erros de domínio**.
6. **Factories** — montam Infra → UseCase → ViewModel.

Fluxo de dependência: `View → ViewModel → UseCase → Model(interfaces) ← Infra`. A View **não importa** `infra` nem `model/usecases`.

## Estrutura de pastas

```
src/
├── app/                          ← Views (Expo Router)
│   ├── index.tsx
│   └── home.tsx
├── model/
│   ├── entities/User.ts
│   ├── errors/DomainErrors.ts
│   ├── services/IAuthService.ts
│   ├── repositories/IUserRepository.ts
│   └── usecases/
│       ├── IAuthUseCases.ts
│       └── AuthUseCases.ts
├── viewmodel/useLoginViewModel.ts
├── infra/
│   ├── services/FirebaseAuthService.ts
│   └── repositories/FirestoreUserRepository.ts
├── factories/loginFactory.ts
└── view/components/
```

---

## 1. Model — entidade, erros e contratos

```typescript
// src/model/entities/User.ts
export type User = {
  uID: string;
  userName: string;
};
```

```typescript
// src/model/errors/DomainErrors.ts
export class DomainError extends Error {
  constructor(message: string) {
    super(message);
    this.name = new.target.name;
  }
}
export class ValidationError extends DomainError {}
export class AuthFailedError extends DomainError {}
```

```typescript
// src/model/services/IAuthService.ts
import { User } from "@/model/entities/User";

export interface IAuthService {
  login(userName: string, password: string): Promise<User>;
  signup(userName: string, password: string): Promise<User>;
  logout(): Promise<void>;
  onAuthStateChanged(callback: (user: User | null) => void): void;
}
```

```typescript
// src/model/repositories/IUserRepository.ts
import { User } from "@/model/entities/User";

export interface IUserRepository {
  save(user: User): Promise<void>;
  findById(id: string): Promise<User | null>;
}
```

## 2. Model — UseCases (regras de negócio)

```typescript
// src/model/usecases/IAuthUseCases.ts
import { User } from "@/model/entities/User";

export interface IAuthUseCases {
  login(userName: string, password: string): Promise<User>;
  signup(userName: string, password: string): Promise<User>;
  logout(): Promise<void>;
  onAuthStateChanged(callback: (user: User | null) => void): void;
}
```

```typescript
// src/model/usecases/AuthUseCases.ts
import { User } from "@/model/entities/User";
import { ValidationError } from "@/model/errors/DomainErrors";
import { IUserRepository } from "@/model/repositories/IUserRepository";
import { IAuthService } from "@/model/services/IAuthService";
import { IAuthUseCases } from "./IAuthUseCases";

export class AuthUseCases implements IAuthUseCases {
  constructor(
    private authService: IAuthService,
    private userRepository: IUserRepository
  ) {}

  async login(userName: string, password: string): Promise<User> {
    this.validate(userName, password);
    return this.authService.login(userName, password);
  }

  async signup(userName: string, password: string): Promise<User> {
    this.validate(userName, password);
    const user = await this.authService.signup(userName, password);
    await this.userRepository.save(user); // orquestração: cria também o registro do usuário
    return user;
  }

  logout(): Promise<void> {
    return this.authService.logout();
  }

  onAuthStateChanged(callback: (user: User | null) => void): void {
    this.authService.onAuthStateChanged(callback);
  }

  private validate(userName: string, password: string): void {
    if (!userName) throw new ValidationError("Informe o usuário.");
    if (!password) throw new ValidationError("Informe a senha.");
  }
}
```

> Mesmo quando a interface do UseCase parece repetir a do service, mantenha o UseCase: é nele que a regra cresce (aqui, `signup` também salva o usuário). Ao aprender o padrão, crie sempre os UseCases.

## 3. Infra — implementação concreta e tradução de erros

```typescript
// src/infra/services/FirebaseAuthService.ts
import { User } from "@/model/entities/User";
import { AuthFailedError } from "@/model/errors/DomainErrors";
import { IAuthService } from "@/model/services/IAuthService";

export class FirebaseAuthService implements IAuthService {
  async login(userName: string, password: string): Promise<User> {
    try {
      // Aqui entraria o SDK real: signInWithEmailAndPassword(auth, userName, password)
      await new Promise((resolve) => setTimeout(resolve, 1000));
      if (userName !== "user@example.com" || password !== "password") {
        throw new Error("auth/invalid-credential"); // erro técnico do SDK
      }
      return { uID: "123", userName: "user123" };
    } catch {
      // A Infra nunca deixa vazar erro de biblioteca: traduz para erro de domínio.
      throw new AuthFailedError("Usuário ou senha inválidos.");
    }
  }

  async signup(userName: string, password: string): Promise<User> {
    throw new AuthFailedError("signup ainda não implementado.");
  }

  async logout(): Promise<void> {}

  onAuthStateChanged(callback: (user: User | null) => void): void {
    callback(null);
  }
}
```

```typescript
// src/infra/repositories/FirestoreUserRepository.ts
import { User } from "@/model/entities/User";
import { IUserRepository } from "@/model/repositories/IUserRepository";

export class FirestoreUserRepository implements IUserRepository {
  async save(user: User): Promise<void> {
    console.log("saving user...", user); // setDoc(...) no Firestore real
  }

  async findById(id: string): Promise<User | null> {
    return { uID: id, userName: "user123" };
  }
}
```

## 4. ViewModel

```typescript
// src/viewmodel/useLoginViewModel.ts
import { useState } from "react";
import { DomainError } from "@/model/errors/DomainErrors";
import { IAuthUseCases } from "@/model/usecases/IAuthUseCases";

export type LoginState = {
  userId: string | null;
  loading: boolean;
  error: string | null;
};

export type LoginActions = {
  handleLogin: (email: string, password: string) => Promise<void>;
};

export function useLoginViewModel(authUseCases: IAuthUseCases): LoginState & LoginActions {
  const [userId, setUserId] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function handleLogin(email: string, password: string) {
    try {
      setLoading(true);
      setError(null);
      const user = await authUseCases.login(email, password);
      setUserId(user.uID);
    } catch (err) {
      // Erro de domínio -> mensagem legível; qualquer outro -> mensagem genérica
      setError(err instanceof DomainError ? err.message : "Falha inesperada.");
    } finally {
      setLoading(false);
    }
  }

  return { userId, loading, error, handleLogin };
}
```

## 5. Factory (injeção de dependências)

```typescript
// src/factories/loginFactory.ts
import { FirestoreUserRepository } from "@/infra/repositories/FirestoreUserRepository";
import { FirebaseAuthService } from "@/infra/services/FirebaseAuthService";
import { AuthUseCases } from "@/model/usecases/AuthUseCases";
import { useLoginViewModel } from "@/viewmodel/useLoginViewModel";

// Dependências montadas UMA vez, fora do render.
const authUseCases = new AuthUseCases(
  new FirebaseAuthService(),
  new FirestoreUserRepository()
);

// Começa com "use" porque chama um hook: só pode ser usada no topo de um componente.
export function useLoginViewModelFactory() {
  return useLoginViewModel(authUseCases);
}
```

## 6. View

```tsx
// src/app/index.tsx
import { router } from "expo-router";
import { useEffect, useState } from "react";
import { ActivityIndicator, Pressable, StyleSheet, Text, TextInput, View } from "react-native";
import { useLoginViewModelFactory } from "@/factories/loginFactory";

const Login = () => {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const { userId, loading, error, handleLogin } = useLoginViewModelFactory();

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

A View só conhece a Factory e a ViewModel; não importa `infra` nem `model`.

## Responsabilidades (resumo)

| Camada | Faz | Não faz |
|---|---|---|
| View | Renderiza, dispara ações | Regra, `try/catch`, importar infra/model |
| ViewModel | Estado da tela, traduz erro em texto | Regra de negócio, JSX |
| UseCase | Regras e orquestração | Conhecer React, Firebase, HTTP |
| Model | Entidades, erros, interfaces | Importar React/Expo/SDKs |
| Infra | Fala com SDKs e traduz erros | Deixar erro de biblioteca vazar |
| Factory | Monta e injeta dependências | Conter lógica |
