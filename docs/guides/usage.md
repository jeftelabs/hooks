# Guia de uso

## Instalar e importar

```sh
npm install @jeftelabs/hooks
# ou
bun add @jeftelabs/hooks
```

```ts
import Hooks from '@jeftelabs/hooks'
import type { HookHandler, HookHandlerProvider, CleanupHandler } from '@jeftelabs/hooks/types'
```

A entrada raiz exporta `Hooks` como default. `Runner` e aliases de callback são expostos pela subpath `types`.

## Registrar handlers

```ts
const hooks = new Hooks()

hooks.add('created', (userId: string) => {
  console.log(`created ${userId}`)
})

hooks.add('created', {
  name: 'metrics.created',
  handle(event, userId: string) {
    console.log(event, userId)
  },
})
```

O nome do evento é fornecido a `add`, mas só é passado como primeiro argumento para providers. Handlers função recebem os argumentos de execução sem o nome do evento. Uma mesma referência só é adicionada uma vez ao evento.

## Executar e filtrar

```ts
await hooks.runner('created').run('u-123')
await hooks.runner('created').runReverse('u-123')
await hooks.runner('created').without(['metrics.created']).run('u-123')
await hooks.runner('created').without().run('u-123') // nenhum handler
```

Handlers executam um a um e são aguardados. A execução para se algum handler lançar/rejeitar. Use runner novo para cada execução; chamar `run` de novo na mesma instância é no-op.

## Definir os tipos dos argumentos

Cada evento mapeia para `[argsDoHandler, argsDoCleanup]`:

```ts
type Events = {
  deleting: [[id: number], [error: Error | null, id: number]]
}

const hooks = new Hooks<Events>()
hooks.add('deleting', async (id) => {
  await stageDelete(id)
  return async (error, cleanupId) => {
    if (error) await restore(cleanupId)
  }
})
```

O compilador verifica o nome do evento e tipos/ordem dos parâmetros. Tipos compartilhados são documentados na [referência da API](../references/api.md).

## Administração do registro

`has(event)` verifica se há handlers; `has(event, handler)` verifica identidade específica. `remove(event, handler)` retorna `true` se removeu. `clear(event)` remove o evento; `clear()` limpa tudo. `merge(other)` acrescenta handlers de outra instância e preserva deduplicação.

`all()` retorna o Map interno. Leia-o para inspeção, mas faça alterações por `add`, `remove`, `clear` ou `merge`.
