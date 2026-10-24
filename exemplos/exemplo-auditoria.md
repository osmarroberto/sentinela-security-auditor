# Exemplos de uso da Skill SENTINELA

## Exemplo A — Diagnóstico inicial sem escrever código
> Use a Skill Sentinela para auditar este repositório. Faça análise passiva, cubra C01–C20, A01–A21 e B01–B20, identifique riscos comprovados e prováveis, rastreie arquivo e linha, destaque faltas de evidência, não edite nenhum arquivo e entregue um plano P0/P1/P2.

## Exemplo B — Supabase + Next.js
> Audite meu sistema Next.js com Supabase/Postgres. Investigue autorização no servidor, RLS, RPCs SECURITY DEFINER, policies de Storage, env NEXT_PUBLIC, exposições de service_role, CORS e isolamento entre organizações. Não tente testar produção. Faça uma matriz por tabela e por perfil.

## Exemplo C — Revisão de PR
> Faça uma revisão de segurança das mudanças deste pull request. Foque nas rotas alteradas, efeitos em autenticação/autorização, novos segredos, dependências e migrações. Mostre somente achados com evidência e diferencie riscos preexistentes dos introduzidos.

## Exemplo D — Correção supervisionada
> Analise os achados SEC-001 e SEC-002, proponha um diff mínimo, testes de regressão e um plano de rollback. Não modifique os arquivos até que eu aprove.

## Exemplo E — Reteste
> Compare o commit atual com o relatório anterior. Execute apenas testes benignos autorizados no ambiente de staging. Classifique cada achado como corrigido, ainda aberto ou inconclusivo, incluindo evidências reproduzíveis.

## Exemplo F — Checklist pré-lançamento
> A aplicação está prestes a entrar em produção. Aplique as três matrizes: C01–C20, A01–A21 e B01–B20, classifique os riscos, destaque blockers de lançamento e o que exige validação humana ou acesso à infraestrutura. Não confunda ausência de evidência com aprovação.

## Exemplo G — Testes de isolamento entre usuários (não destrutivos)
> Audite o isolamento de dados entre ANÔNIMO, USUÁRIO A, USUÁRIO B e ADMIN DE TESTE. Analise endpoints, queries, policies de RLS, Firebase Rules e acesso a arquivos. Primeiro gere testes negativos e positivos; execute-os apenas no staging autorizado, com dados fictícios. Reporte A05, A06, A07, A09, A10 e A20, com evidências.

## Exemplo H — Vazamento de chaves e segredos no Git
> Revise os controles A01, A02, A03 e A18 neste repositório e em seus artefatos de build. Verifique exposição em JavaScript, `.env`, histórico Git e CI, sem imprimir tokens. Distinga chaves públicas de segredos e proponha contenção e rotação segura; não reescreva o histórico sem minha autorização.

## Exemplo I — Auditoria de lançamento com IA
> Faça uma auditoria completa de pré-produção cobrindo C01–C20, A01–A21 e B01–B20. Entregue diagnóstico rastreável, resumo executivo, riscos P0/P1/P2, matrizes com 61 itens, evidências, limitações, testes de aceite e recomendação de go/no-go. Primeiro só análise passiva, sem alterar nada.


## Exemplo J — Fraudes de negócio e concorrência
> Ative SENTINELA V3. Audite B05, B06, B07 e B08 para fluxos de preço, aprovação, pagamento e eventos externos. Identifique regras de negócio, estados autorizados, riscos de replay e necessidade de idempotência. Não execute cobranças reais nem altere produção.

## Exemplo K — Pré-deploy de aplicação SaaS com Supabase
> Use SENTINELA V3 para revisão passiva do repositório Next.js/Supabase. Priorize A01–A10, A18, B01, B07, B09–B11, B16 e B20. Mostre auth, RLS por operação, RPC, storage, cache, CI/CD e riscos de isolamento entre organizações, depois complete as três matrizes C/A/B com evidências e lacunas.

## Exemplo L — Produto com agente de IA integrado
> Audite B18 em conjunto com A05–A10, B07 e B16. Verifique se texto de clientes, documentos e RAG são tratados como não confiáveis; se ferramentas têm permissões mínimas; se há confirmação humana para ações sensíveis; e se o backend impede que respostas de IA acessem dados de outros usuários. Use testes fictícios.
