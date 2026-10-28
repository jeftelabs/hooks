# Instruções para agentes — @jeftelabs/hooks

Leia este arquivo e o índice [docs/README.md](docs/README.md) antes de trabalhar. Código, testes, `package.json`, lockfiles, workflows e documentação são fontes de verdade; confirme divergências na implementação e atualize o material afetado.

## Escopo

`@jeftelabs/hooks` é uma biblioteca TypeScript/ESM para registrar e executar handlers de eventos de ciclo de vida. Handlers podem ser funções ou providers com `name` e `handle(event, ...args)`. Execuções são ordenadas, suportam ordem reversa, filtragem por nome e funções de cleanup retornadas pelos handlers. Tipos genéricos permitem definir argumentos dos handlers e dos cleanups por evento.

O pacote não integra framework, não dispara eventos de domínio automaticamente, não persiste estado e não executa cleanup por conta própria: quem usa a biblioteca deve chamar `runner.cleanup(...)` após a operação protegida.

## Documentação obrigatória

É obrigatório criar ou atualizar documentação detalhada sempre que algo novo for criado ou qualquer parte do projeto for modificada: código, API, tipos, testes, exemplos, dependências, configuração, scripts, build, CI, release ou documentação. A atualização deve ocorrer na mesma tarefa.

- Explique para humanos e agentes de IA propósito, fluxo, entradas, saídas, efeitos, falhas, limites e manutenção.
- Organize páginas em `docs/` por arquitetura, guides, references, planning e ferramentas, conforme o assunto e o padrão existente.
- Registre cada página em `docs/README.md`; mantenha links relativos válidos.
- Atualize `README.md` quando instalação, compatibilidade, API pública ou exemplos mudarem.
- Confira afirmações em código, testes, manifesto, lockfiles e workflows; não copie instruções de outro pacote sem validar.

## Mapa do código

- `index.ts`: export default da classe `Hooks`.
- `src/hooks.ts`: armazenamento de handlers por evento e API de registro/consulta/remoção/merge.
- `src/runner.ts`: execução serial ou reversa, filtro `without` e ciclo de cleanup.
- `src/types.ts`: tipos públicos `Runner`, `CleanupHandler`, `HookHandler`, `HookHandlerProvider` e `ExtractHookHandlerArgs`.
- `tests/`: contratos da classe, runner, cleanup e inferência de tipos.
- `bin/test.ts`: configuração e execução do runner de testes.
- `.github/workflows/`: validação Node/Bun, releases, deploy, labels e fechamento de itens antigos.

## Regras de implementação

- Preserve ESM, exports publicados em `package.json` e imports relativos com extensão `.js` nas fontes, salvo alteração deliberada com build/testes comprovados.
- Preserve a ordem de inserção, deduplicação de mesma referência via `Set` e execução sequencial aguardada.
- `Runner` é one-shot: após `run()` ou `runReverse()`, chamadas posteriores não executam handlers novamente. A execução malsucedida ainda pode deixar cleanups pendentes.
- Cleanups registrados rodam em ordem inversa à execução dos handlers. `run()` não os chama; a aplicação chama `cleanup()` com os argumentos de cleanup definidos para o evento.
- `without(names)` compara os nomes dos handlers; funções usam `Function.name`, providers usam `name`. `without()` sem argumentos ignora todos os handlers.
- Um erro em handler propaga ao chamador e interrompe os handlers seguintes. Cleanups já registrados continuam disponíveis para tentativa explícita do chamador.
- Um cleanup rejeitado interrompe o ciclo e o runner não oferece retry; documente e teste qualquer alteração nessa semântica.
- Não altere `build/`, `coverage/` ou `node_modules/` como fonte. Confirme scripts em `package.json`.
- O CI deve validar Node.js e Bun. Mudanças de runtime/execução exigem testes, typecheck e build nos dois.
- Use Conventional Commits e preserve alterações preexistentes.
