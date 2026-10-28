# Desenvolvimento e runtimes

## Ambientes validados

O projeto mantém suporte testado a Node.js e Bun. A suíte pode ser executada com o loader `@jeftelabs/exec` no Node e diretamente pelo runtime Bun. O pacote publicado é ESM TypeScript compilado para `build/`.

## Instalar dependências

```sh
npm ci
# ou
bun install --frozen-lockfile
```

## Scripts

| Ação | Node.js/npm | Bun |
| --- | --- | --- |
| Lint + testes | `npm test` | `bun run test:bun` |
| Testes sem lint | `npm run quick:test` | `bun run quick:test:bun` |
| Typecheck | `npm run typecheck` | `bun run typecheck` |
| Build | `npm run build` | `bun run build:bun` |

Build usa `tsup-node` para gerar ESM em `build/` e TypeScript para gerar declarações. `test:bun` inclui lint e executa `bin/test.ts` diretamente.

## CI e contribuição

`.github/workflows/checks.yml` roda Node e Bun em pull requests e pushes para `main`, instalando pelo lockfile de cada gerenciador e validando testes, tipos e build. Para mudança que afete runtime, execução, build ou dependências, valide ambos localmente e alinhe os lockfiles.

Toda mudança exige revisão da documentação: consulte `AGENTS.md`, `CLAUDE.md` e este índice. `npm test` executa lint como pretest; o hook `.husky/pre-commit` também chama `npm test`.
