# Padrões de Estado no MVVM Simplificado

## Os 4 estados obrigatórios

Todo ViewModel deve gerenciar explicitamente estes 4 estados:

| Estado | Quando ocorre | O que a View mostra |
|---|---|---|
| **loading** | Buscando dados | Spinner / Skeleton |
| **erro** | Falha na operação | Mensagem + botão de retry |
| **vazio** | Sucesso mas sem dados | Mensagem informativa |
| **sucesso** | Dados carregados | Lista / conteúdo |

## Modelo de estado completo (TypeScript)

```typescript
type Estado<T> =
  | { tipo: 'loading' }
  | { tipo: 'erro'; mensagem: string }
  | { tipo: 'vazio' }
  | { tipo: 'sucesso'; dados: T };
```

## ViewModel com os 4 estados

```typescript
export function useProdutoViewModel() {
  const [estado, setEstado] = useState<Estado<Produto[]>>({ tipo: 'loading' });

  async function carregar() {
    setEstado({ tipo: 'loading' });
    try {
      const dados = await produtoRepository.buscarTodos();
      if (dados.length === 0) {
        setEstado({ tipo: 'vazio' });
      } else {
        setEstado({ tipo: 'sucesso', dados });
      }
    } catch (e) {
      setEstado({ tipo: 'erro', mensagem: 'Erro ao carregar produtos.' });
    }
  }

  return { estado, carregar };
}
```

## View respondendo aos 4 estados

```tsx
export function ProdutoPage() {
  const { estado, carregar } = useProdutoViewModel();

  useEffect(() => { carregar(); }, []);

  switch (estado.tipo) {
    case 'loading': return <Spinner />;
    case 'erro':    return <Erro mensagem={estado.mensagem} onRetry={carregar} />;
    case 'vazio':   return <p>Nenhum produto cadastrado.</p>;
    case 'sucesso': return <ListaProdutos produtos={estado.dados} />;
  }
}
```

## Regra: nunca omitir estados

- Se não tratar `loading` → a tela pisca ou mostra dados antigos
- Se não tratar `erro` → o usuário não sabe o que aconteceu
- Se não tratar `vazio` → parece que está carregando para sempre
- Se não tratar `sucesso` → qual é o ponto?
