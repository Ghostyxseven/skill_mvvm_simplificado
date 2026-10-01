# Injeção de Dependências e Testabilidade

No **MVVM Sofisticado** a View não instancia serviços nem repositórios da infraestrutura (`new FirebaseAuthService()`). Uma **Factory** monta as peças e entrega a ViewModel pronta.

## O padrão Factory

```typescript
// src/factories/loginFactory.ts
import { FirestoreUserRepository } from "@/infra/repositories/FirestoreUserRepository";
import { FirebaseAuthService } from "@/infra/services/FirebaseAuthService";
import { AuthUseCases } from "@/model/usecases/AuthUseCases";
import { useLoginViewModel } from "@/viewmodel/useLoginViewModel";

// 1. Infra e UseCase são criados UMA vez (nível de módulo), nunca dentro do render.
const authUseCases = new AuthUseCases(
  new FirebaseAuthService(),
  new FirestoreUserRepository()
);

// 2. Chama um hook, então o nome começa com "use" (regra dos hooks / ESLint).
export function useLoginViewModelFactory() {
  return useLoginViewModel(authUseCases);
}
```

Por que fora do render? Criar `new ...()` dentro do componente recria tudo a cada renderização, perde estado interno (caches, listeners) e dispara efeitos desnecessários.

## Consumindo a Factory na View

```tsx
// src/app/index.tsx
import { useLoginViewModelFactory } from "@/factories/loginFactory";

const Login = () => {
  const { loading, error, handleLogin } = useLoginViewModelFactory();
  // ...só renderiza e chama handleLogin
};
export default Login;
```

A View não sabe que o Firebase existe. Trocar Firebase por Supabase = escrever novas classes em `infra/` e mudar **uma** linha na Factory.

## Testabilidade

Como tudo depende de interfaces, dá para injetar *fakes*.

### Testando o UseCase (sem React)

```typescript
// src/model/usecases/AuthUseCases.test.ts
import { AuthUseCases } from "./AuthUseCases";
import { ValidationError } from "@/model/errors/DomainErrors";
import { IAuthService } from "@/model/services/IAuthService";
import { IUserRepository } from "@/model/repositories/IUserRepository";

const fakeService: IAuthService = {
  login: async () => ({ uID: "1", userName: "test" }),
  signup: async () => ({ uID: "1", userName: "test" }),
  logout: async () => {},
  onAuthStateChanged: () => {},
};
const fakeRepo: IUserRepository = {
  save: async () => {},
  findById: async () => null,
};

test("login com senha vazia lança ValidationError", async () => {
  const useCases = new AuthUseCases(fakeService, fakeRepo);
  await expect(useCases.login("a@b.com", "")).rejects.toBeInstanceOf(ValidationError);
});
```

### Testando a ViewModel (hook precisa de `renderHook`)

```typescript
// src/viewmodel/useLoginViewModel.test.ts
import { act, renderHook } from "@testing-library/react-native";
import { AuthFailedError } from "@/model/errors/DomainErrors";
import { IAuthUseCases } from "@/model/usecases/IAuthUseCases";
import { useLoginViewModel } from "./useLoginViewModel";

const failing: IAuthUseCases = {
  login: async () => { throw new AuthFailedError("Usuário ou senha inválidos."); },
  signup: async () => ({ uID: "1", userName: "x" }),
  logout: async () => {},
  onAuthStateChanged: () => {},
};

test("erro de domínio vira vm.error e loading volta a false", async () => {
  const { result } = renderHook(() => useLoginViewModel(failing));
  await act(async () => { await result.current.handleLogin("a@b.com", "x"); });
  expect(result.current.error).toBe("Usuário ou senha inválidos.");
  expect(result.current.loading).toBe(false);
});
```

Hooks só rodam dentro de um componente: chamar `useLoginViewModel(...)` direto num teste lança erro.
