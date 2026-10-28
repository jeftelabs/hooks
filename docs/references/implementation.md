# Implementação e estados

## Registro

`src/hooks.ts` armazena `Map<Event, Set<Handler | Provider>>`. Um novo evento ganha Set na primeira inserção. Identidade de função/objeto determina deduplicação e remoção. `merge` itera o Map/Set de origem e chama `add` no destino.

`all()` devolve o objeto privado diretamente. A API não fornece cópia defensiva: mutá-lo externamente pode contornar as operações documentadas.

## Seleção e execução

`Hooks.runner(event)` passa ao runner o Set armazenado para o evento. Runner guarda a referência a esse Set ou cria um vazio. `#exec` aceita execução apenas no estado `idle`, muda o estado para `cleanup_pending`, aplica filtro opcional e percorre handlers com `await` sequencial.

Handlers função são chamados com `(...args)`. Providers usam `.handle(action, ...args)`. Só retornos que sejam funções são enfileirados como cleanup.

## Estado de cleanup

Estados: `idle` → `cleanup_pending` ao começar execução → `cleanup_initiated` ao começar cleanup → `cleanup_completed` depois de todos os callbacks resolverem. `isCleanupPending` é verdadeiro apenas no segundo estado.

Ao iniciar cleanup, os callbacks são consumidos do fim para o começo. Se um callback rejeita, o estado permanece `cleanup_initiated`; chamadas posteriores não passam na condição `isCleanupPending`, portanto não há retry da sequência.

## Semânticas de erro

Erro de execução não é capturado dentro de `Runner`; rejeita a Promise e preserva cleanups que tenham sido coletados antes da falha. Handlers subsequentes não são iniciados. É responsabilidade do chamador decidir se executa cleanup e como mantém o erro original.

## Filtro

O filtro consulta `handler.name`. Em funções, isso é `Function.name`; em providers, é a propriedade obrigatória `name`. Lista vazia não ignora nenhum. `without()` define flag para ignorar todos. Um novo runner evita carregar filtros de execuções anteriores.
