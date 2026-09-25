# 🏗️ Skill: MVVM Simplificado e Sofisticado (Padrão PDM)

> Skill para agentes de IA que impõe rigorosamente a arquitetura MVVM ensinada na disciplina de PDM (Programação para Dispositivos Móveis).

## 🎯 O que essa skill faz

Quando instalada, os agentes de IA (Antigravity, Claude Code, Gemini CLI) **são obrigados** a seguir o padrão MVVM nas suas versões Simplificada ou Sofisticada. A IA para de fazer o padrão "CR - Codifica e Remenda" (código espaguete) e começa a separar responsabilidades, realizar validações passo a passo e aplicar Injeção de Dependências.

## 📂 Estrutura de Pastas (Como seu projeto vai ficar)

A IA é instruída a montar os seus projetos exatamente com esta árvore:

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

## 🚀 Como Usar

A skill roda sozinha de forma transparente. Após instalada, basta pedir para a IA criar uma tela ou funcionalidade. 

**Exemplo de pedido:**
> *"Crie uma tela de Perfil de Usuário para alterar a foto do avatar."*

**O que a IA vai fazer automaticamente:**
1. Ler as regras do `SKILL.md`.
2. Criar a camada `Model` (Entidade e Interface do Repositório).
3. Criar a `Infra` e o `UseCase` (Regra de negócio).
4. Criar a `ViewModel` gerenciando os 4 estados obrigatórios.
5. Criar a `View` puxando as funções através da `Factory`.
6. Rodar o **Checklist de Autocorreção** (revisar se não misturou as responsabilidades).
7. Só entregar o código pra você quando estiver testado e validado.

## 📚 Conteúdo da Skill

- **`SKILL.md`**: Arquivo principal com as regras de ouro, ordem de implementação e checklist de validação para a IA.
- **`examples/mvvm_sofisticado.md`**: Exemplo completo da arquitetura de 5 camadas.
- **`examples/react_typescript.md`** & **`expo.md`**: Exemplos para aplicações React e React Native na versão simplificada.
- **`references/padroes_estado.md`**: A regra exata de 5 passos para tratar erros sem usar `try/catch` na View.
- **`references/injecao_dependencias.md`**: Como criar e usar Factories (Padrão Factory) para testabilidade.

## 💻 Instalação

```bash
# Clone este repositório
git clone https://github.com/SEU-USUARIO/skill_mvvm_simplificado.git

# Crie um link simbólico para a pasta de skills da sua IA
mkdir -p ~/.agents/skills
ln -s $(pwd)/skill_mvvm_simplificado ~/.agents/skills/skill_mvvm_simplificado
```
