---
name: "TDR - Impacto cross-repo"
description: "Rastrear mudanças que impactam outros repositórios do ecossistema jrigo23/TDR*"
title: "[TDR][Cross-Repo] <resumo curto da mudança>"
labels: ["tdr", "cross-repo", "impact"]
assignees: []
---

## Contexto
Descreva a mudança (o que/por que) e o repositório de origem.

- **Repo de origem**: 
- **PR/Commit/Release**: 
- **Motivação**: 

## O que mudou (detalhes)
Liste mudanças relevantes com bullets.

- 

## Tipo de impacto
Marque o que se aplica:

- [ ] Breaking change de contrato (DTO/request/response)
- [ ] Breaking change de rota/endpoint
- [ ] Mudança de validação/defaults
- [ ] Mudança de autenticação/autorização (claims/roles/scopes)
- [ ] Mudança de código/payload de erro
- [ ] Mudança de paginação/ordenação
- [ ] Mudança de regra de negócio com efeito cross-service
- [ ] Mudança de banco (schema/objects/migrations/scripts SQL)
- [ ] Performance/timeout/retry observável por clientes
- [ ] Outro (descrever)

## Repositórios potencialmente afetados (jrigo23/TDR*)
Liste os repos que podem precisar de ação.

- [ ] jrigo23/TDR____
- [ ] jrigo23/TDR____

## Ações necessárias por repositório
Para cada repo afetado, descreva o que precisa ser feito e (se possível) a issue/PR correspondente.

### jrigo23/TDR____
- Ação: 
- Referências: 
- Responsável: 

## Compatibilidade / rollout
Explique como manter compatibilidade (quando aplicável).

- Estratégia: (ex.: versionamento, feature flag, rota paralela, comportamento antigo mantido por X dias)
- Janela de rollout: 
- Plano de rollback: 

## Como validar / testar
Passos para validar o impacto e confirmar que tudo continua funcionando.

1. 

## Checklist
- [ ] Issue(s) abertas para todos os repositórios afetados
- [ ] PR/issue de origem referenciada acima
- [ ] Impactos e ações documentados
- [ ] Plano de compatibilidade/rollout definido (quando aplicável)
- [ ] Passos de validação/teste documentados