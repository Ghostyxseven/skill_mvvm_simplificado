# Consumo de APIs (fetch / axios) no MVVM

Chamada HTTP **nunca** fica na View nem na ViewModel. Ela vive em um Repository (Simplificado) ou em uma implementação na Infra atrás de uma interface (Sofisticado).

## Regras

- Sempre cheque `response.ok` no `fetch` (ele não lança erro em 404/500). O `axios` já lança em respostas não-2xx.
- Converta o JSON para a **entidade do domínio** no repository; a ViewModel não conhece o formato da API.
- Traduza o erro técnico em mensagem/erro de domínio (`references/padroes_estado.md`).
- A View de lista trata os 4 estados e usa `FlatList` (não `.map` em `ScrollView`) para listas longas, com `keyExtractor`.
- `axios` já traz seus tipos: `npm install axios` basta (não instale `@types/axios`).

## Simplificado — Repository no Model

```typescript
// src/model/entities/Post.ts
export type Post = { id: number; title: string; body: string };
```

```typescript
// src/model/repositories/PostRepository.ts
import { Post } from "@/model/entities/Post";

export class PostRepository {
  async findAll(): Promise<Post[]> {
    const response = await fetch("https://jsonplaceholder.typicode.com/posts");
    if (!response.ok) throw new Error("Não foi possível carregar os posts.");
    return response.json();
  }

  async findComments(postId: number): Promise<{ id: number; body: string }[]> {
    const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${postId}/comments`);
    if (!response.ok) throw new Error("Não foi possível carregar os comentários.");
    return response.json();
  }
}
```

Com axios o corpo muda só por dentro:

```typescript
import axios from "axios";
async findAll(): Promise<Post[]> {
  try {
    const { data } = await axios.get<Post[]>("https://jsonplaceholder.typicode.com/posts");
    return data;
  } catch {
    throw new Error("Não foi possível carregar os posts.");
  }
}
```

ViewModel e View: mesmo formato de `examples/expo.md` (`posts`, `loading`, `error`, `loadPosts`; View com `FlatList`, retry e mensagem de lista vazia). Dispare o carregamento inicial na View com `useEffect(() => { loadPosts(); }, [loadPosts])`, com `loadPosts` em `useCallback`.

## Sofisticado — interface no Model, implementação na Infra

```typescript
// src/model/repositories/IPostRepository.ts
import { Post } from "@/model/entities/Post";
export interface IPostRepository {
  findAll(): Promise<Post[]>;
}
```

```typescript
// src/infra/repositories/HttpPostRepository.ts
import axios from "axios";
import { Post } from "@/model/entities/Post";
import { NetworkError } from "@/model/errors/DomainErrors";
import { IPostRepository } from "@/model/repositories/IPostRepository";

export class HttpPostRepository implements IPostRepository {
  async findAll(): Promise<Post[]> {
    try {
      const { data } = await axios.get<Post[]>("https://jsonplaceholder.typicode.com/posts");
      return data;
    } catch {
      throw new NetworkError("Sem conexão ou servidor indisponível.");
    }
  }
}
```

Adicione `export class NetworkError extends DomainError {}` em `model/errors/DomainErrors.ts`. O UseCase (`ListPostsUseCase`) recebe `IPostRepository` pelo construtor e a Factory monta `new ListPostsUseCase(new HttpPostRepository())`.

## Paginação / pull-to-refresh

`FlatList` aceita `onEndReached` (próxima página) e `refreshing` + `onRefresh` (puxar para atualizar). Os dois chamam ações da ViewModel (`loadMore`, `reload`); o estado da página fica na ViewModel.
