# 🏗️ Skill: MVVM Simplificado e Sofisticado (Padrão PDM)

> Skill para agentes de IA que impõe rigorosamente a arquitetura MVVM ensinada na disciplina de PDM (Programação para Dispositivos Móveis).

## 🎯 O que essa skill faz

Quando instalada, os agentes de IA (Antigravity, Claude Code, Gemini CLI) **são obrigados** a seguir o padrão MVVM nas suas versões Simplificada ou Sofisticada. A IA para de fazer o padrão "CR - Codifica e Remenda" (código espaguete) e começa a separar responsabilidades, realizar validações passo a passo e aplicar Injeção de Dependências.

## 💻 Como Instalar a Skill

Diferente de bibliotecas normais, você **não** instala essa skill dentro do seu projeto (ela não vai no `package.json`). Você a instala **globalmente no seu assistente de IA**. 

Assim, ela funciona para *qualquer* projeto mobile que você abrir no seu computador.

**Opção 1: Usando o comando CLI de skills (Recomendado)**
```bash
skills install github:SEU-USUARIO/skill_mvvm_simplificado
```

**Opção 2: Instalação Manual (Git Clone)**
Se você não tiver o comando `skills` configurado, basta clonar direto na pasta de inteligência artificial do seu sistema:
```bash
# Cria a pasta de skills da IA (caso não exista)
mkdir -p ~/.agents/skills

# Clona a skill diretamente para lá
git clone https://github.com/SEU-USUARIO/skill_mvvm_simplificado.git ~/.agents/skills/skill_mvvm_simplificado
```

> **Pronto!** A partir de agora, qualquer assistente de IA que você abrir no seu terminal já conhecerá as regras da disciplina.

---

## 🚀 Como Usar no Dia a Dia (Na Prática)

Você não precisa decorar a arquitetura, criar pastas manualmente ou ficar lembrando a IA de seguir boas práticas. A skill cuida disso. Veja como é simples:

### Passo 1: Crie seu projeto normalmente
Crie seu aplicativo Expo ou React Native do zero (ou abra um existente):
```bash
npx create-expo-app meu-projeto
cd meu-projeto
```

### Passo 2: Faça o pedido para a IA
Abra seu agente de IA (Antigravity, Claude Code, etc) dentro do projeto e simplesmente peça a funcionalidade, sem precisar explicar arquitetura:
> **Você:** *"Crie uma tela de Login que autentica usando Firebase, contendo os campos de e-mail e senha."*

### Passo 3: O Trabalho da IA
A IA vai detectar automaticamente que é um projeto mobile e vai puxar as regras desta skill. Você verá a IA criando as coisas **nesta ordem exata**:
1. ⚙️ Cria a entidade `User` e a regra de negócio (`Model`).
2. 🔌 Cria a conexão com o Firebase isolada (`Infraestrutura`).
3. 🧠 Cria o Custom Hook gerenciando loading e erro (`ViewModel`).
4. 🎨 Desenha a interface puxando o hook (`View`).
5. ✅ Faz uma auto-leitura do código para garantir que nenhuma regra do MVVM foi quebrada.

### Passo 4: Código Limpo
No final, a sua pasta `src/` estará perfeitamente modularizada e pronta para tirar 10 na disciplina, sem você ter tido o estresse de organizar as pastas manualmente.

---

## 📂 Estrutura de Pastas (Como seu projeto vai ficar)

```text
src/
├── app/                        ← View: Telas e rotas (Expo Router)
│   └── index.tsx               (Apenas renderiza e interage com o usuário)
├── viewmodel/                  ← ViewModel: Custom Hooks (ex: useLogin.ts)
│   └── useLoginViewModel.ts    (Gerencia loading, erro e chama Casos de Uso)
├── model/                      ← Model/Domínio: Regras de negócio puras
│   ├── entities/               (Entidades, ex: User.ts)
│   ├── usecases/               (Regras, ex: AuthUseCases.ts)
│   └── services/               (Contratos/Interfaces puras)
├── infra/                      ← Infraestrutura: Onde fica o código "sujo"
│   └── services/               (Implementações reais: Firebase, Axios, SQLite)
└── di/                         ← Dependency Injection (Injeção de dependências)
    └── loginFactory.ts         (Fábricas que montam a ViewModel para a View)
```

## 📚 Conteúdo da Skill

- **`SKILL.md`**: Arquivo principal com as regras de ouro, ordem de implementação e checklist de validação para a IA.
- **`examples/mvvm_sofisticado.md`**: Exemplo completo da arquitetura de 5 camadas.
- **`examples/react_typescript.md`** & **`expo.md`**: Exemplos para aplicações React e React Native na versão simplificada.
- **`references/padroes_estado.md`**: A regra exata de 5 passos para tratar erros sem usar `try/catch` na View.
- **`references/injecao_dependencias.md`**: Como criar e usar Factories (Padrão Factory) para testabilidade.
