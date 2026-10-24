# Evidências, testes seguros e priorização

## Sumário
1. Protocolo de evidência
2. Testes permitidos por modo
3. Cálculo de risco e confiança
4. Modelo do achado
5. Critérios de reteste

## 1. Protocolo de evidência

Para **cada achado**:
- Identificador único `SEC-001`, `SEC-002`...
- Componente/endpoint/tabela e arquivo/linha quando existentes.
- Versão/branch/data/ambiente analisados e se é evidência estática ou execução observada.
- Comportamento esperado vs. observado; origem do dado e percurso até o sink, guard/controle ausente ou inadequado.
- Condições para abuso e controle compensatório encontrado (se houver).
- Confiança: `CONFIRMADO`, `PROVÁVEL` ou `HIPÓTESE`. Resultado do teste e ferramentas, quando houver.
- Trecho mínimo, sem PII, tokens completos, endpoints sensíveis privados ou dumps de dados.
- Severidade, impacto, correção em causa raiz, critério de aceite e teste de regressão.

Não cite números de linhas não verificados. Se a ferramenta retornar apenas amostra de arquivo, declarar a limitação. Achado sem confirmação suficiente deve permanecer hipótese com plano de verificação.

## 2. Testes seguros por modo

**PASSIVO (padrão):** ler arquivos já disponibilizados, rastrear chamadas, inspecionar SQL/migrations, analisar IaC e dependências, comparar configurações, procurar segredos localmente (com mascaramento), revisar logs já fornecidos com minimização de dados.

**VALIDAÇÃO_SEGURA (somente autorizada):** executar testes funcionais benignos em staging/ambiente explicitamente definido, com contas descartáveis e dados fictícios. Exemplos: verificar que usuário A não lê objeto de B; verificar que requisição inválida retorna 400; conferir resposta 401/403; reproduzir regressões com testes unitários isolados. Restringir chamadas, respeitar rate limit e cessar sob erro anormal.

**CORREÇÃO_ASSISTIDA:** preparar diff, testes, migração reversível ou plano forward-fix, pedir aprovação para gravar; avaliar impacto, manter backup/branch e não subir para produção sem aprovação separada.

**RETESTE:** comparar evidência original com teste atual, registrar PASSOU/FALHOU/INCONCLUSIVO, versão testada, mudanças relacionadas e risco residual. Não marcar resolvido por mera presença de patch.

Em produção não executar força bruta, injeções agressivas, scanners amplos, enumeração em massa, modificações de dados, dumps, exploração de terceiros, testes de indisponibilidade ou uso de credenciais reais de outros usuários. Não fornecer nem executar técnicas de exploração para exfiltrar dados de produção.

## 3. Cálculo de risco e confiança

Não atribuir uma pontuação CVSS sem dados suficientes para preencher métricas. Se existir CVE, checar versão e configuração afetadas. EPSS é informação estatística adicional, não confirmação de exposição.

| Severidade | Critério orientativo (dependente do contexto) |
|---|---|
| CRÍTICO | Controle do sistema, dados de vários clientes, acesso administrativo ou vazamento relevante sem barreiras plausíveis, ou indisponibilidade severa facilmente provocável |
| ALTO | Acesso indevido material, escalada de privilégio, vazamento importante ou comprometimento parcial com condições plausíveis |
| MÉDIO | Exploração condicionada, escopo limitado ou controle compensatório forte; dano concreto ainda possível |
| BAIXO | Risco restrito, defesa em profundidade, exploração muito condicionada |
| INFORMATIVO | Melhoria de maturidade, recomendação preventiva ou necessidade de observabilidade sem vulnerabilidade material demonstrada |

Confiança:
- **CONFIRMADO:** reprodução segura em ambiente autorizado ou prova estática inequívoca de um caminho vulnerável e alcançável.
- **PROVÁVEL:** evidência forte de código/config, mas sem validação de controles externos ou condições de runtime.
- **HIPÓTESE:** indício incompleto ou ausência de acesso suficiente.

Prioridade de correção considera: severidade, exposição, criticidade do serviço, PII/dados financeiros, alcance multi-tenant, detectabilidade e facilidade de mitigação. Nunca some pontos inventados nem confunda prioridade com CVSS.

## 4. Exemplo de descrição profissional de achado

**SEC-001 — Possível leitura cruzada entre organizações**
- **Status:** PROVÁVEL; **Severidade:** ALTO (a confirmar com validação autorizada).
- **Local:** `api/orders.ts`, função `getOrder()` (somente se inspecionada).
- **Evidência:** busca por ID de pedido com filtro apenas por ID; não foi localizado filtro por organização na consulta. É necessário conferir middleware/RLS.
- **Impacto plausível:** usuário autenticado poderia visualizar pedido de outro tenant se não houver proteção no DB.
- **Correção:** aplicar e testar autorização por organização no backend e regra por linha no banco, se aplicável.
- **Teste seguro:** criar usuários A/B e pedidos fictícios em tenants separados; verificar `403/404` quando A tenta consultar pedido de B.
- **Critério de aceite:** duas camadas de controle validadas ou justificativa documentada; teste automatizado passa.

Não use este exemplo como achado real sem evidência do sistema auditado.

## 5. Fechamento e reteste

Definição de pronto para corrigido:
1. Código/config alteração aprovada e rastreável.
2. Teste de segurança passa, cobrindo caso negativo e caso positivo legítimo.
3. Validação no ambiente pretendido ou limitação registrada.
4. Risco residual, efeitos adversos, regressões e plano de rollback revisados.
5. Evidência e data de fechamento anexadas ao achado.

Estado final: `ABERTO`, `EM CORREÇÃO`, `AGUARDANDO TESTE`, `CORRIGIDO / RETESTADO`, `RISCO ACEITO`, `FALSO POSITIVO` ou `INCONCLUSIVO`.
