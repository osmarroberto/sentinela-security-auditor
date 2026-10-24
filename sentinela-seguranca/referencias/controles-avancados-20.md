# SENTINELA V3 — 20 verificações avançadas (B01–B20)

Esta é a **terceira matriz** da SENTINELA. Integra-se aos 20 controles estruturais `C01–C20` e às 21 práticas `A01–A21` da versão anterior, totalizando **61 pontos de verificação**, não 61 vulnerabilidades diferentes. **Não crie achados duplicados:** um achado pode apontar para vários controles. Evite aplicar controles irrelevantes; marque `NÃO SE APLICA` com justificativa concreta. A execução de testes ativos continua sujeita ao escopo, autorização e às regras de segurança definidas no `SKILL.md`.

## Como auditar cada B

Para cada B01–B20 registre: **estado** (`CONFORME`/`FALHA`/`NÃO VERIFICADO`/`NÃO SE APLICA`), **evidência** rastreável, superfície analisada, **confiança** (`CONFIRMADO`/`PROVÁVEL`/`HIPÓTESE`), impacto, ação, teste de aceite, dependências de validação em ambiente real. Nunca confunda existência de biblioteca/configuração com efetividade. Marque como `NÃO VERIFICADO` o que exigir acesso ou teste não disponível. Regras descritas a seguir são *hipóteses de avaliação*, não afirmações de falha no projeto.

### B01 — Ciclo de vida de sessão, cookies e tokens
- **Inspecionar:** validade, revogação, rotação de refresh token, logout, mudança de senha, bloqueio/desativação, sessão de outros dispositivos, cookies `HttpOnly`/`Secure`/`SameSite` adequados, armazenamento de tokens e proteção contra fixação e reuso indevido.
- **Risco característico:** token continua utilizável após mudança relevante, sessão compartilhada indevidamente, refresh token sem política coerente de rotação/revogação.
- **Aceite:** regras documentadas e aplicadas pelo servidor/provedor; testes unitários/integrados com tokens sintéticos demonstram invalidação conforme política; não imprimir tokens reais. Tokens JWT já emitidos podem ter revogação dependente do modelo: descreva o risco residual, não pressuponha blacklist universal.

### B02 — OAuth/OIDC, MFA e recuperação de conta
- **Inspecionar:** `state`, `nonce` quando aplicável, PKCE, validação de `issuer`/`audience`, `redirect_uri`, vinculação de contas, MFA/reauth em mudanças críticas, verificação de e-mail, tokens de redefinição com uso único e expiração, proteção contra enumeração e mudança indevida de titularidade.
- **Risco característico:** tomada de conta por callback inseguro, token de reset reutilizável, troca de e-mail/senha sem confirmação apropriada.
- **Aceite:** testes em staging com identidades fictícias para callbacks alterados, tokens expirados e vínculos não autorizados; uso de bibliotecas e provedores conforme documentação e avaliação do fluxo real.

### B03 — XSS, CSRF, template injection e segurança de saída
- **Inspecionar:** renderização HTML/Markdown/rich text, `dangerouslySetInnerHTML`, templates, arquivos SVG, tratamento contextual de saída, CSP e execução de scripts de terceiros; nas sessões com cookies, conferir defesa CSRF apropriada para ações mutáveis.
- **Risco característico:** texto de usuário executado como HTML/JS, proteção baseada só em 'limpar strings', ação autenticada forjada por origem externa.
- **Aceite:** encoding conforme contexto, tratamento explícito de HTML quando necessário, política CSP apropriada, proteção CSRF comprovada conforme padrão de autenticação; testes apenas com dados inofensivos.

### B04 — SSRF, redirect, acesso de saída e endpoints internos
- **Inspecionar:** importação de URLs, previsualização de links, webhooks de saída, proxies de imagens/documentos, fetch no backend, redirecionamentos e resolução DNS; considerar endereços de loopback, rede privada e endpoints de metadados de cloud.
- **Risco característico:** aplicação busca URL fornecida pelo usuário sem restrições e alcança serviços internos.
- **Aceite:** allowlist por serviço quando viável, egress control, revalidação de IP/DNS/redirecionamentos e protocolo, limitações de tempo/tamanho; auditoria estática primeiro, testes somente em laboratório/ambiente autorizado, sem atingir serviços internos reais.

### B05 — Integridade dos fluxos e regras de negócio
- **Inspecionar:** valores financeiros, descontos, permissões de aprovação, etapas de onboarding, ações de contratação, operações que dependem de estado, convites e limites de uso. Mapear invariantes de negócio explícitas.
- **Risco característico:** pular etapa obrigatória ou aceitar `price`, `status`, `approved`, `role` e `owner` controlados pelo cliente, mesmo com dados tecnicamente bem formatados.
- **Aceite:** servidor recalcula preços e escopo e aplica máquina de estados e checagem de autorização por transição; casos negativos de tentativa de pular/repetir/reordenar etapas.

### B06 — Condições de corrida, transações e idempotência
- **Inspecionar:** check-then-act, estoques, crédito, cupons, assinatura, agendamento, geração de cobrança, pagamentos, endpoints idempotentes, retries e jobs concorrentes.
- **Risco característico:** duas solicitações simultâneas utilizam o mesmo saldo, aprovam o mesmo item ou geram efeitos duplicados.
- **Aceite:** operações sensíveis atômicas (transação/bloqueio/condição) e idempotência durável; teste concorrente benigno em banco isolado usando dados fictícios e sem impacto operacional.

### B07 — Permissões por campo, mass assignment e respostas excessivas
- **Inspecionar:** `PUT/PATCH` com objetos completos, `select('*')`, GraphQL mutations/queries, serialização de modelos, atualização de perfil, campos `role`, `tenant_id`, `is_admin`, `user_id`, `payment_status` e exportações.
- **Risco característico:** usuário edita propriedade privilegiada ou recebe campos que a UI oculta.
- **Aceite:** allowlist de atributos por papel/operação, DTOs de saída mínimos, campo sensível não atualizável por usuário comum; testes de contrato de API e comparação autorizada entre usuários/tenants.

### B08 — Autenticidade de webhooks e APIs terceirizadas
- **Inspecionar:** endpoints que recebem eventos de Stripe, gateways, CRM, e-mails, GitHub, Firebase e provedores; assinatura conforme provedor, leitura de corpo bruto, janela temporal, ID do evento, deduplicação, reprocessamento, retries, timeouts e confiança na resposta externa.
- **Risco característico:** webhook falsificado confirma pagamento ou altera permissões; mesma mensagem processada duas vezes.
- **Aceite:** assinatura/autenticidade verificadas, timestamp e prevenção de replay onde suportados, idempotência por evento, limitação de tamanho, validação de esquema e filas seguras. Não assumir que header de origem comprova autenticidade.

### B09 — Dependências, CVEs, SBOM e licenças de execução
- **Inspecionar:** dependências diretas/transitivas, arquivos lock, versões descontinuadas, imagens base, bibliotecas não usadas e scripts de install; auditar avisos de segurança e origem dos pacotes.
- **Risco característico:** biblioteca explorável em caminho de execução, dependency confusion, inclusão de pacote comprometido.
- **Aceite:** lockfile consistente, SCA automatizada quando disponível, inventário/SBOM adequado ao porte, análise de aplicabilidade e plano de atualização com testes. **CVE encontrada no pacote não prova exploração da aplicação.**

### B10 — Pipeline CI/CD, integridade de build e proteção do repositório
- **Inspecionar:** GitHub Actions/GitLab, permissões de `GITHUB_TOKEN`, pull requests externos, jobs com secrets, branch protection, revisão de código, workflows reutilizáveis, assinatura/proveniência do artefato quando pertinente e segregação de deploy.
- **Risco característico:** mudança não revisada ou pipeline com permissões amplas publica em produção; segredo acessível a job não confiável.
- **Aceite:** privilégio mínimo e tokens curtos/OIDC quando suportado, aprovação de produção, separação de ambientes e rastreabilidade de artefatos. Não disparar pipelines de publicação durante análise.

### B11 — Infraestrutura, IAM, rede e configuração como código
- **Inspecionar:** Dockerfile, Compose, Terraform, Vercel/Netlify/cloud, service accounts, portas, políticas de rede, bucket exposure, permissões `*`, execução como root, volumes, console administrativa e separação dev/staging/prod.
- **Risco característico:** serviço exposto indevidamente, execução privilegiada de container, principal IAM com acesso universal.
- **Aceite:** menor privilégio, fronteiras de rede, imagens mantidas, entradas privadas não expostas, regras IaC revisadas; distinguir configuração declarada de estado implantado.

### B12 — Criptografia, gerenciamento de chaves e proteção de dados
- **Inspecionar:** TLS, dados sigilosos em repouso, algoritmos atuais, geração de aleatoriedade, gestão/rotação de chaves, KMS, backups criptografados, assinatura e senhas com hashing adequado.
- **Risco característico:** cifra obsoleta, segredo de criptografia hardcoded, IV/nonce reutilizado ou senhas reversivelmente criptografadas.
- **Aceite:** bibliotecas consolidadas, algoritmos aprovados e configuração coerente, chaves segregadas por ambiente e rotacionáveis, mecanismos de hashing apropriados; não criar criptografia caseira.

### B13 — Detecção, trilhas de auditoria e resposta a incidentes
- **Inspecionar:** logs de login negado, elevação de privilégio, mudanças de regra, acesso/exportação sensível, alertas, retenção e integridade de logs, ID de correlação, playbooks e responsáveis.
- **Risco característico:** invasão silenciosa, registro de token/PII em texto claro ou impossibilidade de reconstruir evento.
- **Aceite:** eventos acionáveis, alarmes testáveis, dados sensíveis redigidos, acessos ao log limitados e plano de contenção/revogação/restauração; não atribuir monitoramento ativo apenas pela biblioteca instalada.

### B14 — Backups, restauração e continuidade
- **Inspecionar:** backup de dados/metadados/arquivos, frequência, retenção, separação de credenciais/contas, restauração em ambiente isolado, RPO/RTO, migrações e plano de rollback.
- **Risco característico:** cópia existe mas está corrompida, inacessível, desatualizada ou depende de mesmo administrador comprometido.
- **Aceite:** evidência de restauração recente e íntegra, metas de recuperação definidas, backups protegidos e teste de recuperação não destrutivo; sem executar restore em produção.

### B15 — Consumo de recursos, quotas e indisponibilidade
- **Inspecionar:** rate limit por função e usuário, tamanho/custo de query, paginação, GraphQL depth, loops, compressão, regex complexa, uploads, processamento de vídeo e filas; concorrência e orçamento de serviços pagos.
- **Risco característico:** uma requisição permitida consome recursos excessivos ou custos inesperados, mesmo sem brute force.
- **Aceite:** limites de tamanho/tempo/custo/concorrência, filas com backpressure, observabilidade e quotas por identidade; testar limites em ambiente controlado, sem gerar negação de serviço.

### B16 — Jobs assíncronos, cache, Realtime e compartilhamento entre contas
- **Inspecionar:** workers, cron, scheduled functions, queues, Redis/CDN/SSR cache, web sockets, canais Supabase Realtime, push notifications, exportações assíncronas e URLs pré-assinadas.
- **Risco característico:** cache serve informações de outro tenant, job executa com conta de serviço sem checar escopo do usuário originador, subscrição permite ler eventos alheios.
- **Aceite:** chaves/cache variam conforme identidade quando necessário, autorização por assinatura e evento, validação do owner em worker/job, TTL de URLs e segregação por tenant.

### B17 — Privacidade, LGPD e ciclo de vida de dados
- **Inspecionar:** minimização de coleta, necessidade e finalidade, campos sensíveis, consentimento quando aplicável, retenção, exclusão/anonymização, exportação de dados, suboperadores, telemetry, suporte e acessos internos.
- **Risco característico:** dados pessoais desnecessários em logs/analytics, exportações indevidas, ausência de regra de retenção ou acesso excessivo por suporte.
- **Aceite:** práticas técnicas evidenciadas, acesso mínimo, retenção e descarte definidos, exclusão refletida em sistemas pertinentes conforme obrigações; **não afirmar conformidade legal automática**.

### B18 — Segurança de recursos com IA e agentes (somente quando aplicável)
- **Inspecionar:** assistentes com ferramentas, RAG, documentos externos, mensagens de terceiros, plugins, chamadas LLM, uso de API keys, acesso a dados e ações automatizadas. Distinguir instruções confiáveis de conteúdo recuperado.
- **Risco característico:** prompt injection indireto faz agente ignorar limites, revelar dados ou acionar ferramentas com permissões excessivas; respostas de LLM são executadas sem validação.
- **Aceite:** autorização independente no backend, ferramentas com escopo mínimo, revisão humana para ações sensíveis, validação de saídas, sandbox quando necessário e testes com exemplos inofensivos. Se o produto **não incorpora IA**, marque `NÃO SE APLICA`: usar Claude para auditar **não** torna o produto um sistema com IA.

### B19 — Arquivos, caminhos, parsing e download seguro
- **Inspecionar:** download e preview, traversal, zip slip, descompressão, metadata EXIF, conteúdo ativo em SVG/PDF, transformação de imagem/vídeo, bibliotecas de parsing, assinaturas de URL, permissões de arquivo e `Content-Disposition`.
- **Risco característico:** arquivo legítimo aparentemente seguro dispara processamento perigoso, leitura de caminho indevido ou disponibilização de anexo a outro usuário.
- **Aceite:** MIME/extensão/conteúdo verificados conforme formato, diretórios isolados, nomes internos gerados, limites de expansão, escaneamento quando risco justificar e entrega privada autenticada. Evitar recomendar antivírus como controle único.

### B20 — Modelo de ameaças, testes de regressão e gate de lançamento
- **Inspecionar:** documentos de arquitetura/limites de confiança, testes de autorização, migrations e policies, coverage de auth, verificação de segurança por PR, critérios de no-go, inventário de APIs expostas e responsáveis pelos riscos.
- **Risco característico:** correção isolada sem teste, novo endpoint contorna regra antiga, mudança de tabela quebra RLS, regressão passa despercebida.
- **Aceite:** requisitos críticos viram testes automatizados negativos/positivos, plano de reteste alinhado aos achados, criticidade e cobertura explicitadas e decisão de deploy baseada em evidências. Não calcular `61/61 seguro`: contagem de conformidade não é certificado.

## Roteiro de priorização por criticidade e contexto

1. Antes de auditar, identifique se o produto lida com **pagamentos, dados pessoais, multi-tenancy, perfis admin, integrações terceirizadas ou agentes de IA**.
2. Em projeto com **Supabase/Firebase**, aplique primeiro A05–A10, A20, B07, B16 e B20; dê atenção às policies por operação e às instâncias reais, não apenas às migrations locais.
3. Em produto com **pagamentos, aprovação ou precificação**, aplique B05, B06 e B08 antes de detalhes cosméticos de headers.
4. Em aplicação com **importação de URL, webhooks ou proxy**, priorize B04 e B08.
5. Em projeto com **IA integrada ao produto**, adicione B18; se usa Claude apenas como auditor, B18 não se aplica ao produto auditado.
6. Em **pré-deploy**, observe B09–B14 e B20 e garanta que testou restauração de backup em contexto não produtivo quando permitido.
7. Para auditoria rápida, priorize pelos riscos e exponha claramente os controles pendentes, mas para auditoria integral gere 3 matrizes C/A/B. Nunca multiplique a gravidade de um achado por referenciar vários IDs.

## Referências reconhecidas (consultar versões vigentes quando possível)

- OWASP ASVS 5.0: https://asvs.owasp.org/
- OWASP Top 10:2025: https://top10.owasp.org/2025/
- OWASP API Security Top 10:2023: https://owasp.org/API-Security/
- OWASP Business Logic Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html
- OWASP Webhook Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Webhook_Security_Cheat_Sheet.html
- OWASP Software Supply Chain Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Software_Supply_Chain_Security_Cheat_Sheet.html
- OWASP CI/CD Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/CI_CD_Security_Cheat_Sheet.html
- OWASP LLM Top 10:2025: https://genai.owasp.org/llm-top-10/
- NIST SSDF SP 800-218: https://csrc.nist.gov/pubs/sp/800/218/final
