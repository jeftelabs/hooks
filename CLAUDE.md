# Orientações para Claude e agentes de IA

As instruções canônicas estão em [AGENTS.md](./AGENTS.md). Leia também [docs/README.md](./docs/README.md) e as páginas relevantes antes de investigar ou alterar o projeto.

Este repositório contém `@jeftelabs/hooks`, uma biblioteca TypeScript/ESM para registrar handlers de eventos, executá-los em série ou em ordem reversa e executar callbacks de cleanup que eles retornem. A aplicação cliente controla os eventos de domínio e deve chamar cleanup explicitamente.

## Documentação obrigatória

É extremamente obrigatório criar ou atualizar documentação detalhada toda vez que algo novo for criado ou qualquer parte do projeto for modificada. Isso inclui código, API, tipos, testes, exemplos, dependências, configuração, scripts, build, CI, release e documentos. Atualize a documentação na mesma tarefa.

- Descreva comportamento real, fluxo, entradas, saídas, efeitos, erros, limites e manutenção para humanos e IAs.
- Organize o material em `docs/` por arquitetura, guias, referências, planejamento e ferramentas, conforme necessário.
- Inclua cada nova página no índice `docs/README.md` e preserve os links relativos.
- Mantenha README público alinhado com compatibilidade e API pública.
- Confira toda afirmação em código, testes, manifesto, lockfiles e workflows.

## Cuidados específicos

- O export default de `index.ts` é `Hooks`; os tipos ficam no subpath `@jeftelabs/hooks/types`.
- `Hooks<EventMap>` recebe um mapa no formato `evento: [argumentosDoHandler, argumentosDoCleanup]`.
- Handlers de função recebem os argumentos definidos para o evento; providers recebem primeiro o nome do evento em `handle(event, ...args)`.
- `Runner.run()` e `runReverse()` executam uma vez, em série, aguardando cada handler. Um erro interrompe a execução e rejeita a Promise.
- Handlers podem retornar cleanup síncrono ou assíncrono. O cleanup é explícito, roda na ordem inversa e aceita seus próprios argumentos tipados.
- Se uma execução falha, limpezas já retornadas podem ser chamadas pelo código consumidor em `catch`. O pacote não chama cleanup automaticamente.
- `cleanup()` que rejeita interrompe a sequência; o runner não permite nova tentativa por meio de `cleanup()`.
- `without(['nome'])` filtra por `Function.name` ou pelo `name` de um provider; `without()` ignora todos os handlers.
- Preserve Node.js e Bun. Leia scripts reais em `package.json`; execute validação adequada nos dois runtimes quando houver impacto.
- Não invente comportamento transacional, rollback automático, persistência, integração de framework ou execução concorrente.
