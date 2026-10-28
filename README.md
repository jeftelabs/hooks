# @jeftelabs/hooks

Biblioteca TypeScript/ESM para registrar e executar hooks de ciclo de vida em uma ordem previsível. Os handlers podem ser síncronos ou assíncronos e podem retornar funções de cleanup para desfazer recursos preparados antes de uma operação.

## Recursos

- Registro por nome de evento, com funções ou objetos provider.
- Execução sequencial em ordem de registro ou em ordem inversa.
- Argumentos tipados por evento para handlers e cleanups.
- Filtragem de handlers por nome ou supressão de todos.
- Cleanups executados em ordem inversa, uma única vez, quando solicitados pela aplicação.
- Sem dependências de runtime; funciona com Node.js e Bun.

## Instalação

```sh
npm install @jeftelabs/hooks
# ou
bun add @jeftelabs/hooks
```

## Uso básico

```ts
import Hooks from '@jeftelabs/hooks'

const hooks = new Hooks()

hooks.add('saving', async (record: { id: string }) => {
  console.log(`Salvando ${record.id}`)
})

await hooks.runner('saving').run({ id: '42' })
```

Os hooks de um evento são executados na ordem de registro. Cada execução usa um runner novo. Um runner já iniciado não executa novamente os handlers.

## Tipos por evento

O mapa genérico associa cada evento a uma tupla com os argumentos do handler e os argumentos do cleanup:

```ts
import Hooks from '@jeftelabs/hooks'

type Record = { id: string }
type Events = {
  saving: [[record: Record], [error: Error | null, record: Record]]
}

const hooks = new Hooks<Events>()

hooks.add('saving', async (record) => {
  await prepare(record)

  return async (error, savedRecord) => {
    if (error) await undoPreparation(savedRecord)
  }
})

const runner = hooks.runner('saving')
try {
  await runner.run({ id: '42' })
  await persistRecord()
} catch (error) {
  await runner.cleanup(error as Error, { id: '42' })
  throw error
}

await runner.cleanup(null, { id: '42' })
```

O pacote executa handlers e cleanups; a aplicação controla a operação principal e decide quando chamar `cleanup()` e quais argumentos passar.

## Providers e ordem reversa

Um provider permite definir um nome estável para filtrar o handler. Seu método `handle` recebe o nome do evento antes dos demais argumentos.

```ts
hooks.add('saving', {
  name: 'audit.beforeSave',
  async handle(event, record) {
    console.log(event, record.id)
  },
})

await hooks.runner('saving').without(['audit.beforeSave']).run(record)
await hooks.runner('saving').runReverse(record)
```

Funções são identificadas para filtragem pelo `Function.name`. `without()` sem argumentos ignora todos os handlers daquela execução.

## Execução e cleanup

`run()` e `runReverse()` aguardam cada handler antes do próximo. Se um handler lançar ou rejeitar, a execução para e sua Promise rejeita. Cleanups retornados até então ficam pendentes e podem ser chamados pelo consumidor no caminho de erro.

`cleanup()` executa callbacks na ordem inversa à ordem de conclusão/registro e passa a cada um os argumentos de cleanup do evento. O runner evita uma segunda execução de handlers e de cleanups concluídos. Uma falha no próprio cleanup interrompe a sequência; não há retry automático.

## API resumida

- `add(event, handler)`: registra uma função ou provider; a mesma referência não é duplicada.
- `has(event, handler?)`: verifica se há handlers ou se uma referência específica está registrada.
- `remove(event, handler)`: remove pela identidade e informa se removeu.
- `clear(event?)`: limpa um evento ou todos.
- `merge(other)`: acrescenta os handlers da outra instância, mantendo a ordem e a deduplicação por identidade.
- `all()`: retorna o `Map` interno vivo; trate-o como somente leitura.
- `runner(event)`: cria um runner para o evento.
- `Runner.without(names?)`, `run(...args)`, `runReverse(...args)`, `cleanup(...args)` e `isCleanupPending` controlam a execução.

Os tipos `Runner`, `CleanupHandler`, `HookHandler`, `HookHandlerProvider` e `ExtractHookHandlerArgs` são importados por `@jeftelabs/hooks/types`.

## Desenvolvimento

```sh
npm ci
npm test
npm run typecheck
npm run build
```

Com Bun:

```sh
bun install --frozen-lockfile
bun run test:bun
bun run typecheck
bun run build:bun
```

A documentação completa está em [`docs/README.md`](docs/README.md). As regras de contribuição e documentação obrigatória estão em [`AGENTS.md`](AGENTS.md).
