# Cleanup dos hooks

## O que é um cleanup

Um handler pode devolver uma função que será executada mais tarde para desfazer ou encerrar um recurso que preparou. O retorno pode ser síncrono ou uma Promise. A função cleanup recebe argumentos próprios, configurados na tupla do evento.

```ts
type Events = {
  saving: [[record: Record], [error: Error | null, record: Record]]
}

const hooks = new Hooks<Events>()
hooks.add('saving', async (record) => {
  const tempFile = await createTempFile(record)
  return async (error, savedRecord) => {
    if (error) await removeTempFile(tempFile)
    else await finalizeTempFile(tempFile, savedRecord)
  }
})
```

## Controlar a operação protegida

O runner não sabe qual é a operação principal. O chamador deve decidir quando executá-la e passar a cada cleanup os dados adequados:

```ts
const runner = hooks.runner('saving')
try {
  await runner.run(record)
  await saveRecord(record)
} catch (error) {
  await runner.cleanup(error as Error, record)
  throw error
}

await runner.cleanup(null, record)
```

Handlers já concluídos podem ter registrado cleanups se um handler posterior ou a operação principal falhar. Por isso, cleanup deve ser chamado pelo código consumidor também no caminho de erro.

## Ordem e idempotência

Handlers executam serialmente. Cleanups acumulados executam na ordem inversa, semelhante ao unwind de recursos. `isCleanupPending` indica se ainda é possível iniciar cleanup. Depois de iniciar e completar, uma segunda chamada é no-op.

Runner também é one-shot: a execução de hooks não é repetida. O estado de cleanup passa a pendente no começo da execução mesmo que não haja handlers ou que todos sejam ignorados.

## Falhas

Erro de handler interrompe a lista e rejeita `run()`/`runReverse()`. O chamador pode capturar esse erro, executar cleanups já coletados e relançar o erro. Erro de cleanup interrompe a sequência; a implementação muda o estado para iniciado antes de chamar callbacks e não oferece retry. Se a aplicação precisa de tentativa individual/retry, deve tratar isso no próprio callback ou criar outro mecanismo explícito.
