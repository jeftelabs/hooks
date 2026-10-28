# Visão da arquitetura

## Responsabilidades

`Hooks<Events>` mantém um `Map` de nomes de evento para `Set` de handlers. O `Set` preserva a ordem de registro e evita cadastrar duas vezes a mesma referência. A classe oferece operações de configuração; ela não sabe como o domínio executa o evento real.

`Runner<HookArgs, CleanupArgs>` executa o conjunto associado a um evento. O consumidor fornece os argumentos e decide quando chamar cleanup. Os tipos genéricos relacionam argumentos de cada handler com os argumentos aceitos pelas compensações que ele retorna.

## Ciclo de vida típico

1. Crie uma instância de `Hooks`, opcionalmente com um mapa de eventos tipado.
2. Registre handlers durante configuração da aplicação.
3. Para uma operação, crie um runner específico com `hooks.runner(event)`.
4. Execute `run(...args)` ou `runReverse(...args)`. Handlers assíncronos são aguardados um por vez.
5. Um handler pode retornar cleanup. A aplicação executa a operação principal e chama `runner.cleanup(...args)` no caminho de sucesso ou de falha, segundo o contrato dos argumentos do evento.

## Fluxos e falhas

Handler function recebe apenas argumentos definidos para o evento. Provider recebe `(eventName, ...args)` em `handle`. Se um handler falhar, o runner propaga a falha e não inicia os próximos handlers; cleanups já registrados permanecem disponíveis ao chamador. O pacote não envolve a operação principal e não decide atomicidade.

Cleanups são chamados em ordem inversa. `cleanup()` é explícito. Se um cleanup rejeitar, os posteriores não são executados naquela chamada; a máquina de estado não oferece retry depois de iniciada.

## Limites importantes

- `Runner` é one-shot e não suporta reuso concorrente.
- `all()` devolve o `Map` interno vivo; mutações diretas contornam a API de `Hooks` e devem ser evitadas.
- Um runner referencia o `Set` daquele evento. Mudanças no registro antes de iniciar o runner podem alterar o conjunto observado; durante execução, o loop percorre sua coleção de handlers.
- Não há scheduler, persistência, integração de framework, rollback automático nem captura de erros da operação principal.
