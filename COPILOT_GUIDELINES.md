# Diretrizes de uso do GitHub Copilot (C#/.NET 8/9 + ADO.NET + Firebird 5)

Estas diretrizes valem para o repositório. Ao gerar/refatorar código, o Copilot deve seguir as regras abaixo.

## 1) Princípios (ordem de prioridade)
1. **Correção e segurança** > observabilidade (logs/métricas) > performance > conveniência.
2. **Consistência**: seguir padrões já existentes no repo (estrutura, nomes, estilo).
3. **Mudanças pequenas**: preferir commits/PRs pequenos e revisáveis.
4. **Mudança de comportamento exige testes** (unit e/ou integração).

## Ecossistema TDR (multi-repositório: .NET + Firebird)

Este repositório faz parte do ecossistema **TDR**: um conjunto de backends **.NET** com acesso a dados via **ADO.NET + Firebird**, que expõem recursos para um ERP **totalmente baseado em APIs**.

Considere como “repositórios relacionados” **somente** os repositórios listados em `repositories.md` (allowlist).

### Fonte de verdade: lista de repositórios do projeto (obrigatório)

**Sempre** use o arquivo `repositories.md` (na raiz deste repositório `jrigo23/.github`) como **allowlist** de repositórios que fazem parte do projeto/ecossistema TDR para análises, implementações, ajustes e avaliação de impacto cross-repo.

- Não inferir `jrigo23/TDR*` por prefixo.
- Se um repositório não estiver listado em `repositories.md`, trate como **fora de escopo** até o arquivo ser atualizado.

### Regras obrigatórias (impacto entre repositórios)
- **Não presuma** que o conteúdo/código dos outros repositórios está disponível no contexto atual.
- Ao alterar qualquer **contrato público** ou comportamento observável por clientes, você deve tratar como potencial impacto em outros repositórios listados em `repositories.md`.

**Exemplos de mudanças que exigem ação coordenada:**
- DTOs e contratos de API (request/response), validações, defaults, paginação/ordenação.
- Rotas, nomes de endpoints, códigos de erro e payloads de erro.
- Autenticação/autorização (claims/roles/scopes), headers, correlation IDs.
- Regras de negócio com efeito cross-service.
- SQL, schemas, objetos do banco, scripts/migrations, queries críticas.
- Qualquer mudança com potencial breaking-change.

### Processo obrigatório quando houver impacto
Se houver qualquer risco de impacto em outros repositórios listados em `repositories.md`, **sempre**:
1) **Abrir issue(s)** nos repositórios afetados descrevendo:
   - o que mudou;
   - por que;
   - impacto esperado;
   - ações necessárias;
   - como validar/testar.
2) Referenciar as issues abertas no PR/commit (ex.: seção "Impacto no ecossistema TDR").
3) Evitar merge sem um plano claro de compatibilidade/rollout (quando aplicável).

> Se não for possível determinar os repositórios afetados com segurança, ainda assim deve-se abrir ao menos 1 issue de rastreamento (“TDR - avaliar impacto cross-repo”) em um repositório apropriado e listar hipóteses/pendências.

## 2) Padrões C# / .NET (ASP.NET Core e Worker)
- Usar `async/await` para I/O (principalmente acesso ao banco). Evitar `.Result`/`.Wait()`.
- Preferir **injeção de dependência** (DI) e `IOptions<T>` para configurações.
- `CancellationToken`:
  - em controllers/endpoints: propagar o token para services e chamadas ao DB quando suportado;
  - em Workers: respeitar o token no loop e em delays.
- Exceções:
  - não capturar `Exception` sem ação; quando capturar, **adicionar contexto** e re-lançar ou converter para erro de domínio;
  - evitar exceções “genéricas” para fluxo normal.
- Logging:
  - usar logging estruturado (`logger.LogInformation("... {Id}", id)` etc.);
  - **não** logar connection strings, senhas, tokens, PII;
  - logar IDs de correlação quando existirem (request/trace).

## 3) Acesso a dados com ADO.NET (Firebird 5)
### 3.1 Regras obrigatórias (segurança e consistência)
- **Sempre** usar SQL parametrizado (`FbCommand` + parâmetros). Nunca concatenar input em SQL.
- Sempre fechar/dispensar recursos: `await using`/`using` para `FbConnection`, `FbCommand`, `FbDataReader`, `FbTransaction`.
- Nunca manter conexão como singleton; conexões devem ser de curto prazo (pooling faz o resto).

### 3.2 Padrão de execução
- Para leitura:
  - preferir `ExecuteReaderAsync` com `CommandBehavior.SequentialAccess` quando lidar com payload grande (se aplicável);
  - mapear explicitamente colunas (evitar `SELECT *`).
- Para escrita:
  - usar `ExecuteNonQueryAsync`;
  - checar linhas afetadas quando fizer sentido (ex.: update de entidade deve afetar 1 linha).
- Transações:
  - se a operação tiver múltiplos comandos que precisam ser atômicos, **abrir transação** e associar ao command;
  - se uma função recebe `FbTransaction` (padrão “ambient”), não deve criar outra sem necessidade.

### 3.3 Convenções de SQL
- Escrever SQL legível (quebras de linha, alinhamento).
- Usar nomes de parâmetros consistentes (ex.: `@ClienteId`, `@DataInicial`).
- Ordenação/paginação: sempre determinística (ex.: `ORDER BY Id`).
- Evitar N+1 queries em loops; preferir operações em lote.

### 3.4 Erros e retorno
- Se o banco retornar erro, registrar contexto **sem vazar dados sensíveis** (ex.: nome da operação/repositório, IDs relevantes).
- Não expor detalhes internos do banco diretamente para API pública (mapear para erro amigável/seguro quando for endpoint).

## 4) Testes
- Mudança de regra de negócio => atualizar/adicionar teste.
- Para código com ADO.NET:
  - quando possível, extrair mapeamento/transformações para funções testáveis sem DB;
  - testes de integração com Firebird devem ser isolados (DB de teste) e documentados (como rodar).

## 5) Estrutura e legibilidade
- Funções pequenas, responsabilidade única.
- Evitar duplicação: extrair helpers/repositories/services.
- Comentários: explicar **por que** e suposições importantes (ex.: “Firebird usa X por causa de Y”).
- Não introduzir complexidade “invisível” (reflection/dynamic/hacks) sem justificativa clara.

## 6) Dependências
- Não adicionar pacote NuGet novo sem justificativa no PR.
- Preferir BCL e libs já existentes no repo.

---

# Diretrizes de processo (GitHub)

## 7) Commits
- Commits pequenos e com mensagem clara.
- **Conventional Commits recomendado** (não obrigatório): `feat:`, `fix:`, `refactor:`, `test:`, `docs:`.

## 8) Pull Requests (PR)
Todo PR deve conter:
- O que mudou (bullets)
- Por que (motivação)
- Como testar (passos)
- Riscos/impactos (se aplicável)

### Diretriz: Documentação sempre alinhada às mudanças

Sempre que uma mudança **alterar comportamento observável** ou **introduzir/alterar funcionalidades, padrões, configurações ou integrações**, a documentação deve ser readequada no mesmo PR/commit, no mínimo em um dos itens abaixo (conforme aplicável):

- `README.md` (visão geral, como rodar, como configurar, como usar)
- documentação da API (Swagger/OpenAPI, exemplos de request/response, códigos de erro, paginação/ordenação)
- docs internas (pasta `docs/`, wiki, ADRs, comentários “por que”, etc.)
- instruções de migração/upgrade (breaking changes, mudanças de default, flags/configs novas)

### Gatilhos obrigatórios (quando atualizar docs/README)
Atualizar docs/README quando houver, por exemplo:
- novo endpoint/rota ou alteração de contrato (DTOs, validações, defaults)
- nova regra de negócio, mudança de fluxo, ou alteração de comportamento esperado
- novos parâmetros de configuração (env vars/appsettings), novas permissões/roles/claims
- mudança de padrão arquitetural (ex.: novo padrão de repository/service, logging, tratamento de erros)
- alterações relevantes de SQL/schema/migrations ou requisitos de banco
- adição/remoção de feature flags, jobs/workers, integrações externas

### Critério de aceite
- O PR deve conter uma seção **“Documentação”** descrevendo objetivamente:
  - quais arquivos foram atualizados (ex.: `README.md`, `docs/xyz.md`);
  - ou justificar explicitamente: **“Sem impacto de documentação”**.

Checklist:
- [ ] Build/CI passando
- [ ] Testes atualizados/criados (se houve mudança de comportamento)
- [ ] SQL parametrizado (sem concatenação de input)
- [ ] Transação usada quando necessário
- [ ] Sem secrets em código/config/logs
- [ ] Documentação/README atualizados quando houver mudança de funcionalidade/contrato/padrão/config (ou justificar “Sem impacto de documentação”)

## 9) Code review
- Preferir pelo menos 1 aprovação.
- Se mexer em acesso a dados, pedir review de alguém que conheça o modelo/SQL.
- Se deixar TODO, criar issue e referenciar no PR.

## 10) CI/CD (mínimo)
- Build + test obrigatórios antes de merge.
- Regras de branch protection recomendadas:
  - exigir PR para merge na `main`
  - exigir checks obrigatórios (build/test)
  - bloquear force-push
