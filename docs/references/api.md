# Referência da API

## Entradas do pacote

- `@jeftelabs/hooks`: export default `Hooks`.
- `@jeftelabs/hooks/types`: exports de tipos `Runner`, `CleanupHandler`, `HookHandler`, `HookHandlerProvider` e `ExtractHookHandlerArgs`.

## `Hooks<Events>`

O tipo `Events` associa nome a `[argumentosDoHandler, argumentosDoCleanup]`. O padrão aceita nomes string e argumentos amplos.

- `all()` retorna `Map<Event, Set<Handler | Provider>>` interno vivo.
- `has(event, handler?)`: presença de handlers no evento ou presença de referência específica.
- `add(event, handler): this`: registra função ou provider; Set ignora mesma referência repetida.
- `remove(event, handler): boolean`: remove por identidade; `false` se evento ou referência não existir.
- `clear(event?)`: apaga um evento ou toda a coleção.
- `merge(other)`: acrescenta handlers preservando ordem e deduplicação por identidade.
- `runner(event)`: cria `Runner` associado ao Set atual do evento, ou a Set vazio.

## `Runner<HookArgs, CleanupArgs>`

- `action`: nome do evento associado.
- `isCleanupPending`: verdadeiro enquanto cleanup ainda pode começar.
- `without(names?)`: ignora handlers com nomes informados; sem argumentos ignora todos. Retorna `this`.
- `run(...args): Promise<void>`: percorre handlers em ordem de registro.
- `runReverse(...args): Promise<void>`: percorre handlers na ordem inversa.
- `cleanup(...args): Promise<void>`: executa cleanups coletados na ordem inversa; no-op se não estiver pendente.

O runner executa uma vez. Falha de handler rejeita o método de execução; falha de cleanup interrompe essa sequência.

## Tipos

- `CleanupHandler<Args>`: função que recebe os argumentos do cleanup e retorna `void` ou `Promise<void>`.
- `HookHandler<Args, CleanUpArgs>`: função que recebe argumentos do hook e retorna `void`, cleanup ou Promise correspondente.
- `HookHandlerProvider<Args, CleanUpArgs>`: objeto `{ name, handle(event, ...args) }` com o mesmo contrato de retorno.
- `ExtractHookHandlerArgs<Handler>`: utilitário condicional para extrair tuplas de argumentos de um handler.
