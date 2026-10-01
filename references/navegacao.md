# Navegação com Expo Router (padrão da skill)

A skill usa **sempre o Expo Router** (navegação por arquivos). Não use `@react-navigation/*` direto nem `NavigationContainer`/`App.tsx`. Se o projeto antigo já usa React Navigation, mantenha e avise o usuário.

## Regras

1. Em `src/app/` ficam **só telas e `_layout.tsx`**. Componentes, hooks, dados e tipos ficam fora (`view/components`, `viewmodel`, `model`).
2. Cada arquivo de rota tem `export default`. Nomes em minúsculas com hífen (`meus-favoritos.tsx`).
3. `index.tsx` = raiz da pasta; `[id].tsx` = rota dinâmica; `(grupo)/` = agrupa sem mudar a URL; `+not-found.tsx` = endereço inexistente; `_layout.tsx` = como as rotas da pasta se relacionam.
4. **Quem navega é a View.** A ViewModel não importa `expo-router`: ela expõe estado (`userId`, `saved`) e a View reage com `useEffect` + `router.replace(...)`.
5. Parâmetros de rota chegam como **string**: converta (`Number(id)`) e trate valor inválido.
6. Use objetos no `href` (`{ pathname: '/ifpimons/[id]', params: { id } }`), não concatenação de texto. Mantenha `typedRoutes` ativo.

## Escolhendo a operação

| Quero... | Use |
|---|---|
| Navegar por um toque | `<Link href="/sobre">` (ou `asChild` envolvendo um `Pressable`) |
| Navegar depois de lógica | `router.push` (empilha), `router.navigate` |
| Sair sem poder voltar (ex.: pós-login) | `router.replace("/home")` |
| Voltar | `router.back()` (cheque `router.canGoBack()`; senão `router.replace("/")`) |
| Voltar a uma tela já na pilha | `router.dismissTo("/")` — `push("/")` empilharia cópia |

## Layout raiz

```tsx
// src/app/_layout.tsx
import { Stack } from "expo-router";

const RootLayout = () => (
  <Stack>
    <Stack.Screen name="index" options={{ title: "Login" }} />
    <Stack.Screen name="ifpimons/[id]" options={{ title: "Detalhes" }} />
  </Stack>
);

export default RootLayout;
```

`Stack.Screen` só configura; quem cria a rota é o arquivo. Um `name` sem arquivo gera aviso.

## Rota dinâmica com MVVM

A View lê o parâmetro; a ViewModel recebe um valor já tipado e trata "não encontrado".

```typescript
// src/viewmodel/useIfpimonDetailsViewModel.ts
import { IFPIMon } from "@/model/entities/IFPIMon";
import { IfpimonRepository } from "@/model/repositories/IfpimonRepository";

const repository = new IfpimonRepository();

export function useIfpimonDetailsViewModel(rawId: string | undefined) {
  const id = Number(rawId);
  const ifpimon: IFPIMon | undefined = Number.isInteger(id) ? repository.findById(id) : undefined;
  return { ifpimon, notFound: !ifpimon };
}
```

```tsx
// src/app/ifpimons/[id].tsx
import { Stack, router, useLocalSearchParams } from "expo-router";
import { Pressable, StyleSheet, Text, View } from "react-native";
import { useIfpimonDetailsViewModel } from "@/viewmodel/useIfpimonDetailsViewModel";

const IfpimonDetails = () => {
  const { id } = useLocalSearchParams<{ id: string }>();
  const { ifpimon, notFound } = useIfpimonDetailsViewModel(id);

  if (notFound) {
    return (
      <View style={styles.container}>
        <Stack.Screen options={{ title: "Não encontrado" }} />
        <Text>Nenhum IFPIMon com o id {id}.</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Stack.Screen options={{ title: ifpimon!.name }} />
      <Text style={styles.name}>{ifpimon!.name}</Text>
      <Pressable style={styles.button} onPress={() => router.dismissTo("/")}>
        <Text style={styles.buttonText}>Voltar ao início</Text>
      </Pressable>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: "center", justifyContent: "center", gap: 12, padding: 24 },
  name: { fontSize: 28, fontWeight: "bold" },
  button: { backgroundColor: "#4630EB", borderRadius: 8, paddingVertical: 12, paddingHorizontal: 24 },
  buttonText: { color: "#fff", fontWeight: "bold" },
});

export default IfpimonDetails;
```

Item da lista que leva aos detalhes (componente em `view/components/`):

```tsx
<Link href={{ pathname: "/ifpimons/[id]", params: { id: String(ifpimon.id) } }} asChild>
  <Pressable style={styles.item}>
    <Text>{ifpimon.name}</Text>
  </Pressable>
</Link>
```

## Tela não encontrada

```tsx
// src/app/+not-found.tsx
import { Link, Stack } from "expo-router";
import { StyleSheet, Text, View } from "react-native";

const NotFound = () => (
  <View style={styles.container}>
    <Stack.Screen options={{ title: "Ops!" }} />
    <Text>Esta tela não existe.</Text>
    <Link href="/">Voltar ao início</Link>
  </View>
);

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: "center", justifyContent: "center", gap: 16 },
});

export default NotFound;
```

## Tabs e Drawer (também no Expo Router)

O livro mostra Tabs/Drawer com `@react-navigation`; nesta skill use o equivalente em arquivos. As telas continuam sendo Views com ViewModel própria.

```
src/app/
├── _layout.tsx            ← Stack raiz
├── index.tsx              ← login
└── (tabs)/
    ├── _layout.tsx        ← Tabs
    ├── home.tsx           ← rota /home
    └── settings.tsx       ← rota /settings
```

```tsx
// src/app/(tabs)/_layout.tsx
import { Tabs } from "expo-router";

const TabsLayout = () => (
  <Tabs>
    <Tabs.Screen name="home" options={{ title: "Início" }} />
    <Tabs.Screen name="settings" options={{ title: "Configurações" }} />
  </Tabs>
);

export default TabsLayout;
```

Para menu lateral use `Drawer` de `expo-router/drawer` (exige `react-native-gesture-handler` e `react-native-reanimated`; confira a documentação da versão do SDK em uso). Grupos `(tabs)` não aparecem na URL: `router.replace("/home")` continua válido.
