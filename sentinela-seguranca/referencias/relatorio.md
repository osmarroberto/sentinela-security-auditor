# Modelo do Relatório SENTINELA

Adapte profundidade ao porte do sistema; nunca elimine ressalvas de cobertura.

# RELATÓRIO DE SEGURANÇA — [SISTEMA]
**Data:** [AAAA-MM-DD]  
**Ambiente / versão / commit:** [...]  
**Responsável solicitante:** [...]  
**Modalidade:** Passiva / Validação segura / Correção assistida / Reteste

## 1. Sumário executivo
Veredito delimitado ao escopo; principais riscos; se há bloqueador de deploy; o que não foi possível verificar.

**Status de liberação:** `BLOQUEAR`, `LIBERAR COM RISCO DOCUMENTADO`, `SEM BLOQUEADOR IDENTIFICADO NO ESCOPO` ou `INCONCLUSIVO` — com justificativa. `SEM BLOQUEADOR` não significa segurança garantida.

## 2. Escopo, método e limitações
- Sistemas/arquivos/tabelas/ambientes inspecionados:
- Artefatos consultados:
- Ferramentas realmente utilizadas e versões:
- Testes executados (ou “nenhum teste ativo executado”):
- Áreas fora de escopo / dependências não verificadas:
- Janela e autorização, se houver testes ativos:

## 3. Panorama de riscos
| ID | Achado | Severidade | Confiança | Ativo / local | Impacto | Prioridade | Estado |
|---|---|---|---|---|---|---|---|

Resumo quantitativo apenas de achados efetivamente encontrados; não inventar números.

## 4. Achados detalhados
Para cada `SEC-###`:
1. Título e tipo (`vulnerabilidade`, `hardening`, `risco operacional`, `privacidade`).
2. Severidade; confiança; prioridade; estado.
3. Descrição técnica e pré-condições.
4. Evidência (link/arquivo:linha, log sanitizado ou teste benigno, com data).
5. Impacto e cenário de abuso plausível.
6. Controle compensatório e limitações do diagnóstico.
7. Correção recomendada com exemplo localizado no stack quando possível.
8. Teste negativo/positivo, plano de reteste e critérios de aceite.
9. Mapeamentos CWE/ASVS/OWASP, somente se pertinentes.
10. Responsável sugerido, esforço e dependências.

## 5. Três matrizes obrigatórias: C01–C20, A01–A21 e B01–B20
| ID | Controle | Estado | Evidência / limitação | Ação |
|---|---|---|---|---|
| C01–C20 | [uma linha por controle estrutural] | CONFORME/FALHA/NÃO VERIFICADO/NÃO SE APLICA | [...] | [...] |

### 5.2 Verificações práticas A01–A21
| ID | Verificação | Estado | Evidência / limitação | Ação / teste de aceite |
|---|---|---|---|---|
| A01–A21 | [uma linha por verificação prática; não agrupar] | CONFORME/FALHA/NÃO VERIFICADO/NÃO SE APLICA | [...] | [...] |

### 5.3 Verificações avançadas B01–B20
| ID | Verificação avançada | Estado | Evidência / limitação | Ação / teste de aceite |
|---|---|---|---|---|
| B01–B20 | [uma linha por verificação, inclusive NÃO SE APLICA justificado] | CONFORME/FALHA/NÃO VERIFICADO/NÃO SE APLICA | [...] | [...] |

**Resumo de cobertura:** número de controles verificados dos 61, não verificados, não aplicáveis e com falha; distinguir cobertura do código da cobertura da implantação. Se A20 não foi testado por falta de staging/autorização, marcar NÃO VERIFICADO. Se A21 não incluiu testes de execução, descrever revisão IA estática parcial e cobertura/limites. Não contar itens sobrepostos como vulnerabilidades independentes.

## 6. Roadmap priorizado
**Imediato:** riscos ativos, exposição direta, segredos, auth/autorização, dados pessoais.  
**Curto prazo:** correções de alto impacto, testes automatizados, atualização e hardening.  
**Estrutural:** observabilidade, backups restauráveis, recuperação, governança, SSDLC.

Para cada ação, registrar proprietário, dependências, esforço e teste de conclusão.

## 7. Critério de liberação, reteste e riscos residuais
- Bloqueadores e decisões de risco.
- Alterações propostas vs. aplicadas (não misturar).
- Evidências de reteste ou lacunas.
- Próximas ações e prazo sugerido (sem inventar compromissos).

## 8. Notas metodológicas
Análise condicionada aos artefatos acessíveis. Ausência de detecção não comprova ausência de vulnerabilidades. Relatório técnico não substitui pentest devidamente autorizado, auditoria externa ou assessoria jurídica LGPD.
