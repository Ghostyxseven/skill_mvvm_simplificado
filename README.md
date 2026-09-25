# skill_mvvm_simplificado

> Skill para agentes de IA que impõe MVVM simplificado como arquitetura obrigatória em qualquer projeto de aplicação.

## O que essa skill faz

Quando instalada, os agentes de IA (Antigravity, Claude Code, Gemini CLI, Copilot CLI) **são obrigados** a seguir a arquitetura MVVM simplificada sempre que criarem ou editarem código de aplicação — sem precisar ser lembrados a cada conversa.

A skill cobre:
- **Model** → entidades de dados e repositórios (acesso a API/DB)
- **ViewModel** → lógica de negócio, estado e transformação de dados
- **View** → apenas renderização e captura de ações do usuário

## Instalação

```bash
# Opção 1: clonar direto na pasta de skills
git clone https://github.com/seu-usuario/skill_mvvm_simplificado ~/.agents/skills/skill_mvvm_simplificado
```

Após clonar, o agente reconhece automaticamente a skill na próxima sessão.

## Estrutura

```
skill_mvvm_simplificado/
├── SKILL.md                          ← regras obrigatórias (arquivo principal)
├── examples/
│   ├── react_typescript.md           ← exemplo completo React + TS
│   └── expo.md                       ← exemplo completo Expo / React Native
└── references/
    └── padroes_estado.md             ← como gerenciar os 4 estados (loading/erro/vazio/sucesso)
```

## Frameworks suportados pelos exemplos

- Flutter / Dart
- React + TypeScript
- Expo / React Native
- Vue 3 (em breve)

## Compatibilidade

| Agente | Suporte |
|---|---|
| Antigravity (AGY) | ✅ |
| Claude Code | ✅ |
| Gemini CLI | ✅ |
| Copilot CLI | ✅ |
| Qualquer agente que leia `~/.agents/skills/` | ✅ |

## Licença

MIT
