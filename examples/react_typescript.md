# MVVM Simplificado — React + TypeScript (web)

> Variante **web** (usa `<p>`, `<ul>`). A skill é voltada a React Native/Expo; use este arquivo só se o projeto for React para web. A divisão de camadas e os nomes são os mesmos do `expo.md`: o que muda é apenas a View.

## Estrutura de pastas

```
src/
├── model/
│   ├── entities/User.ts
│   └── repositories/UserRepository.ts
├── viewmodel/useUsersViewModel.ts
└── pages/UsersPage.tsx          ← View
```

## Model

```typescript
// src/model/entities/User.ts
export type User = {
  id: string;
  name: string;
  email: string;
};
```

```typescript
// src/model/repositories/UserRepository.ts
import { User } from "@/model/entities/User";

export class UserRepository {
  async findAll(): Promise<User[]> {
    const response = await fetch("/api/users");
    if (!response.ok) throw new Error("Não foi possível carregar os usuários.");
    return response.json();
  }
}
```

## ViewModel (hook)

```typescript
// src/viewmodel/useUsersViewModel.ts
import { useCallback, useState } from "react";
import { User } from "@/model/entities/User";
import { UserRepository } from "@/model/repositories/UserRepository";

const userRepository = new UserRepository();

export function useUsersViewModel() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const loadUsers = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      setUsers(await userRepository.findAll());
    } catch (err) {
      setError(err instanceof Error ? err.message : "Falha inesperada.");
    } finally {
      setLoading(false);
    }
  }, []);

  return { users, loading, error, loadUsers };
}
```

## View

```tsx
// src/pages/UsersPage.tsx
import { useEffect } from "react";
import { useUsersViewModel } from "@/viewmodel/useUsersViewModel";

const UsersPage = () => {
  const { users, loading, error, loadUsers } = useUsersViewModel();

  useEffect(() => {
    loadUsers();
  }, [loadUsers]);

  if (loading) return <p>Carregando...</p>;
  if (error) return <p>{error} <button onClick={loadUsers}>Tentar novamente</button></p>;
  if (users.length === 0) return <p>Nenhum usuário encontrado.</p>;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name} — {user.email}</li>
      ))}
    </ul>
  );
};

export default UsersPage;
```
