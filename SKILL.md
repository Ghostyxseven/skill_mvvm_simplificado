---
name: skill_mvvm_simplificado
description: Use when creating, editing, or reviewing any application code — especially mobile (Flutter, React Native, Expo) or frontend (React, Vue, Angular). This skill enforces simplified MVVM architecture and MUST be followed whenever creating screens, components, pages, services, or data layers. Always apply when the user mentions ViewModel, Model, View, screens, components, state management, repositories, or business logic. Do NOT allow the AI to mix responsibilities between layers or skip any required layer.
---

# MVVM Simplificado — Arquitetura Obrigatória

## Regra Central

**Todo código de aplicação DEVE seguir MVVM simplificado.**
Não há exceção: seja uma tela simples, um componente pequeno ou um fluxo complexo — a separação de camadas é obrigatória.

Misturar responsabilidades entre camadas é proibido. Não importe regras de negócio na View. Não faça chamadas de API direto no componente. Não coloque lógica de UI no ViewModel.

---

## As 3 Camadas

### 1. Model (Dados)
**Responsabilidade:** Representar os dados e se comunicar com fontes externas.

- Define as entidades/classes de dados
- Contém Repositórios que acessam API, banco de dados, cache
- **Não conhece** a UI nem o ViewModel
- **Não tem** lógica de apresentação

```
models/
  user.dart / User.ts          ← entidade de dados
repositories/
  user_repository.dart         ← acessa API, DB, cache
```

### 2. ViewModel (Lógica de negócio + Estado)
**Responsabilidade:** Processar dados, expor estado para a View, responder a ações do usuário.

- Chama o Repository e transforma os dados
- Gerencia o estado da tela (loading, erro, sucesso, dados)
- Expõe métodos que a View pode chamar (ex: `carregarUsuarios()`)
- **Não importa** nada de UI (sem widgets, sem componentes visuais)
- **Não faz** chamadas de API diretamente

```
viewmodels/
  user_viewmodel.dart / useUserViewModel.ts
```

### 3. View (Interface)
**Responsabilidade:** Exibir dados e capturar ações do usuário. Nada mais.

- Lê o estado do ViewModel e renderiza
- Delega todas as ações ao ViewModel
- **Não contém** lógica de negócio
- **Não faz** chamadas de API
- **Não transforma** dados — só exibe

```
views/ ou screens/ ou pages/
  user_screen.dart / UserPage.tsx
```

---

## Estrutura de Pastas Obrigatória

```
lib/ (ou src/)
├── models/
│   └── user.dart
├── repositories/
│   └── user_repository.dart
├── viewmodels/
│   └── user_viewmodel.dart
├── views/ (ou screens/ ou pages/)
│   └── user_screen.dart
└── core/
    ├── services/          ← integrações externas (HTTP, storage)
    └── utils/             ← funções utilitárias puras
```

---

## Regras que NUNCA podem ser quebradas

| ❌ Proibido | ✅ Correto |
|---|---|
| Chamar API direto na View | View chama ViewModel, ViewModel chama Repository |
| Colocar `if/else` de negócio na View | Lógica vai no ViewModel |
| Importar Widget/Component no ViewModel | ViewModel é puro — sem dependências de UI |
| Criar uma tela sem ViewModel | Toda tela tem seu ViewModel |
| Misturar Model e ViewModel no mesmo arquivo | Sempre arquivos separados |
| Fazer fetch de dados no componente principal | Sempre via Repository |

---

## Fluxo de Dados (sempre nesta direção)

```
View → ViewModel → Repository → API/DB
         ↑               ↓
       Estado          Dados brutos
```

A View **nunca** acessa o Repository diretamente.
O Repository **nunca** conhece o ViewModel ou a View.

---

## Nomeação Obrigatória

| Camada | Sufixo/Prefixo | Exemplo |
|---|---|---|
| Entidade | sem sufixo | `User`, `Product`, `Order` |
| Repository | `Repository` | `UserRepository` |
| ViewModel | `ViewModel` ou `useXxxViewModel` | `UserViewModel`, `useUserViewModel` |
| View/Tela | `Screen`, `Page` ou `View` | `UserScreen`, `UserPage` |
| Service | `Service` | `HttpService`, `StorageService` |

---

## Exemplo Mínimo (Flutter/Dart)

**Model:**
```dart
// models/user.dart
class User {
  final String id;
  final String nome;
  User({required this.id, required this.nome});
}
```

**Repository:**
```dart
// repositories/user_repository.dart
class UserRepository {
  Future<List<User>> buscarTodos() async {
    // chama a API aqui
  }
}
```

**ViewModel:**
```dart
// viewmodels/user_viewmodel.dart
class UserViewModel extends ChangeNotifier {
  final UserRepository _repo = UserRepository();
  List<User> usuarios = [];
  bool carregando = false;

  Future<void> carregarUsuarios() async {
    carregando = true;
    notifyListeners();
    usuarios = await _repo.buscarTodos();
    carregando = false;
    notifyListeners();
  }
}
```

**View:**
```dart
// views/user_screen.dart
class UserScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final vm = context.watch<UserViewModel>();
    if (vm.carregando) return CircularProgressIndicator();
    return ListView(
      children: vm.usuarios.map((u) => Text(u.nome)).toList(),
    );
  }
}
```

---

## Exemplos para outros frameworks

Leia os arquivos em `examples/` para ver implementações em:
- `examples/react_typescript.md` — React + TypeScript + hooks
- `examples/vue.md` — Vue 3 + Composition API
- `examples/expo.md` — Expo / React Native

---

## Sinais de que o código está errado (red flags)

- Você vê `fetch()`, `axios.get()` ou `http.get()` dentro de um componente/widget → **ERRADO**
- Você vê lógica condicional de negócio (`if usuarioAdmin`, `calcular desconto`) dentro de uma View → **ERRADO**
- O ViewModel importa `useState`, `Widget`, ou qualquer coisa visual → **ERRADO**
- Existe uma tela que não tem um ViewModel correspondente → **ERRADO**
- Model e ViewModel estão no mesmo arquivo → **ERRADO**

Ao encontrar qualquer red flag acima, refatore antes de continuar.

---

## Para mais exemplos e referências

- `references/padroes_estado.md` — como gerenciar estado (loading, erro, sucesso, vazio)
- `references/testes.md` — como testar cada camada separadamente
