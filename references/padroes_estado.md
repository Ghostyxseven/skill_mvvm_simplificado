# Padrões de Estado e Tratamento de Erros

## Os 4 estados da tela

Toda tela que carrega dados trata explicitamente:

| Estado | Quando ocorre | O que a View mostra |
|---|---|---|
| **loading** | Buscando/processando | Spinner / skeleton |
| **erro** | Falha na operação | Mensagem + botão de tentar novamente |
| **vazio** | Sucesso, mas sem dados | Mensagem informativa |
| **sucesso** | Dados carregados | Lista / conteúdo |

- Sem `loading`: a tela pisca ou mostra dado velho.
- Sem `erro`: o usuário não sabe o que houve.
- Sem `vazio`: parece que está carregando para sempre.

A ViewModel expõe `loading`, `error` e os dados; **vazio** é derivado (`data.length === 0 && !loading && !error`). Ordem de checagem na View: loading → erro → vazio → sucesso. Exemplo completo em `examples/expo.md`.

## Fluxo de erros (MVVM Sofisticado) — 5 passos

1. **Infra** captura o erro técnico da biblioteca (Firebase, HTTP, SQLite).
2. **Infra traduz** para um erro de domínio (`AuthFailedError`, definido em `model/errors/`).
3. **UseCase** valida e pode lançar erros de negócio (`ValidationError`) ou remapear mensagens.
4. **ViewModel** captura e converte em texto legível (`vm.error`). Erro que não for de domínio vira "Falha inesperada.".
5. **View** apenas exibe `error`.

```typescript
// Infra
catch { throw new AuthFailedError("Usuário ou senha inválidos."); }

// ViewModel
catch (err) {
  setError(err instanceof DomainError ? err.message : "Falha inesperada.");
}
```

No **Simplificado**, o service lança `Error` com mensagem legível e a ViewModel usa `err instanceof Error ? err.message : "Falha inesperada."`. Evite `catch (err: any)`.

> 🚨 **Regra de ouro:** a View **nunca** faz `try/catch`.
