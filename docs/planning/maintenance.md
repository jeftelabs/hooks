# Planejamento de manutenção

## Contratos a preservar

- Eventos mantêm handlers em ordem de inserção; mesma referência não duplica.
- Handlers assíncronos são aguardados serialmente; falha interrompe execução e rejeita.
- `run` e `runReverse` são alternativos de uma única execução por Runner.
- Handlers função recebem argumentos do evento; providers recebem event name antes desses argumentos.
- Cleanup é explicitamente acionado pelo consumidor e executado em ordem inversa.
- Tupla genérica do evento controla argumentos de handler e de cleanup separadamente.
- Exports ESM, subpath pública de tipos e artefatos de declaração devem permanecer coerentes.
- Node.js e Bun devem continuar cobertos pelo CI.

## Checklist de mudança

1. Identifique o contrato e os arquivos afetados em `src/`, `tests/`, `package.json`, lockfiles e workflows.
2. Atualize testes para execução normal, ordem, filtros, erros, limpeza e tipos relevantes.
3. Atualize o guia ou referência correspondente, este índice e README público se a mudança afetar uso/compatibilidade.
4. Rode lint/testes, typecheck e build em Node e Bun se alterar execução, compatibilidade ou dependências.
5. Confira exports publicados, `npm pack --dry-run` se alterar empacotamento, links e `git diff --check`.

## Pontos para avaliação futura

- Avaliar cópia defensiva em `all()` para impedir mutações diretas ao registro.
- Definir/documentar política para múltiplas chamadas de `without()` antes de execução.
- Considerar recuperação ou resultado agregado se um cleanup falhar; qualquer mudança altera contrato de falhas e exige testes.
- Revisar suporte a event names não-string/strings vazias conforme tipagem e comportamento desejado.
- Incluir testes para mutações do Set durante iteração caso esse comportamento passe a ser garantido.
