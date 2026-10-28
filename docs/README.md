# Documentação de @jeftelabs/hooks

Este índice orienta pessoas e agentes de IA que usam, mantêm ou integram o pacote. Confirme os contratos em `src/`, `tests/`, `package.json`, lockfiles e workflows; atualize as páginas nesta mesma tarefa quando mudar comportamento.

## Comece aqui

- [Visão geral da arquitetura](architecture/overview.md): componentes e ciclo de execução.
- [Guia de uso](guides/usage.md): registro, tipagem, execução e filtro.
- [Guia de cleanup](guides/cleanup.md): callbacks de compensação, ordem, sucesso e falha.
- [Referência da API](references/api.md): exports, métodos, retornos e tipos.

## Referências de manutenção

- [Implementação e estados](references/implementation.md): detalhes do runner e comportamentos observáveis.
- [Desenvolvimento e runtimes](references/development.md): scripts, Node.js, Bun, CI e build.
- [Planejamento de manutenção](planning/maintenance.md): contratos, checklist e pontos para avaliar.

## Mapa do código

| Caminho | Responsabilidade |
| --- | --- |
| `index.ts` | Export default da classe `Hooks` |
| `src/hooks.ts` | Armazenamento e administração de handlers por evento |
| `src/runner.ts` | Execução serial/reversa, filtro e cleanup |
| `src/types.ts` | Tipos dos callbacks e export do tipo `Runner` |
| `tests/` | Contratos de registro, execução, cleanup e tipos |
| `bin/test.ts` | Inicializa e executa a suíte `@t8ngs/runner` |
| `.github/workflows/checks.yml` | Lint/testes, typecheck e build em Node.js e Bun |

## Regra de atualização

Toda criação ou modificação no projeto exige atualização documental detalhada no mesmo trabalho. Consulte [`AGENTS.md`](../AGENTS.md) e [`CLAUDE.md`](../CLAUDE.md); mantenha este índice navegável e verifique links relativos.
