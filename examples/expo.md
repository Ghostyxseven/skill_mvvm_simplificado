# MVVM Simplificado — Expo / React Native (lista de dados com 4 estados)

Exemplo de uma lista que busca dados remotos e trata **loading, erro, vazio e sucesso**.

## Estrutura de pastas

```
src/
├── app/
│   ├── _layout.tsx
│   └── users.tsx               ← View (rota /users)
├── model/
│   ├── entities/User.ts
│   └── repositories/UserRepository.ts
└── viewmodel/useUsersViewModel.ts
```

## Model

```typescript
// src/model/entities/User.ts
export type User = {
  id: string;
  name: string;
};
```

```typescript
// src/model/repositories/UserRepository.ts
import { User } from "@/model/entities/User";

export class UserRepository {
  async findAll(): Promise<User[]> {
    const response = await fetch("https://api.exemplo.com/users");
    if (!response.ok) throw new Error("Não foi possível carregar os usuários.");
    return response.json();
  }
}
```

## ViewModel

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
// src/app/users.tsx
import { useEffect } from "react";
import { ActivityIndicator, FlatList, Pressable, StyleSheet, Text, View } from "react-native";
import { useUsersViewModel } from "@/viewmodel/useUsersViewModel";

const Users = () => {
  const { users, loading, error, loadUsers } = useUsersViewModel();

  useEffect(() => {
    loadUsers();
  }, [loadUsers]);

  if (loading) return <ActivityIndicator style={styles.center} />;

  if (error) {
    return (
      <View style={styles.center}>
        <Text>{error}</Text>
        <Pressable style={styles.button} onPress={loadUsers}>
          <Text style={styles.buttonText}>Tentar novamente</Text>
        </Pressable>
      </View>
    );
  }

  if (users.length === 0) {
    return (
      <View style={styles.center}>
        <Text>Nenhum usuário encontrado.</Text>
      </View>
    );
  }

  return (
    <FlatList
      data={users}
      keyExtractor={(user) => user.id}
      renderItem={({ item }) => <Text style={styles.item}>{item.name}</Text>}
    />
  );
};

const styles = StyleSheet.create({
  center: { flex: 1, justifyContent: "center", alignItems: "center", gap: 12 },
  item: { padding: 16, borderBottomWidth: 1, borderBottomColor: "#eee" },
  button: { backgroundColor: "#4630EB", borderRadius: 8, paddingVertical: 10, paddingHorizontal: 20 },
  buttonText: { color: "#fff", fontWeight: "600" },
});

export default Users;
```
