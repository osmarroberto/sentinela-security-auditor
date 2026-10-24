# Matriz técnica de 20 controles estruturais (C01–C20) do SENTINELA

**Complemento obrigatório:** ler também `seguranca-aplicacional-21.md` (A01–A21) e `controles-avancados-20.md` (B01–B20). Entregar **três matrizes separadas** em auditorias completas; identificadores diferentes podem se relacionar ao mesmo achado, sem duplicar a contagem de falhas.

## Sumário
1. Os 20 controles essenciais
2. Cobertura ampliada
3. Critérios de interpretação por arquitetura
4. Checklist rápido pré-produção

## 1. Os 20 controles essenciais

Avalie individualmente cada controle. A identificação é `C01` a `C20`. Para cada um: status, localização verificada, evidência, risco, correção e teste proposto. `NÃO VERIFICADO` não é sinônimo de reprovação.

| ID | Controle | Onde inspecionar | Evidências e verificações mínimas |
|---|---|---|---|
| C01 | HTTPS/TLS | ingress, proxy, CDN, certificados, redirect, HSTS | TLS obrigatório para tráfego sensível; ausência de mixed content; redirect seguro; certificados válidos e TLS legado desabilitado conforme ambiente |
| C02 | Senhas com hash forte | auth, DB, IAM, auth provider | Argon2id/scrypt/bcrypt configurado adequadamente; salt individual; nunca plaintext/MD5/SHA simples; resets por token de uso único |
| C03 | MFA | auth, IAM, admins, contas críticas | MFA real para perfis privilegiados; recuperação segura; rate limit e políticas de reautenticação em ações sensíveis |
| C04 | Rate limit/antiabuso | login, recuperação, OTP, APIs, uploads | Limites por identidade/IP conforme contexto; proteção de enumeração e custos; tratamento de proxies; sem bloqueio indevido de usuários legítimos |
| C05 | Validação de inputs | handlers, DTO, forms server-side, parser | Tipos, limites, allowlists e schemas verificados no servidor; `body` desconhecido rejeitado quando adequado; checagem de tamanho e formatos |
| C06 | Sanitização/encoding de saída | templates, HTML, markdown, logs, export | Encoding contextual anti-XSS, políticas para HTML rico, saída segura, prevenção de injeção em planilhas/logs; não substituir por sanitização genérica |
| C07 | SQL/NoSQL injection | ORM, queries, filtros dinâmicos, RPCs | Consultas parametrizadas; SQL dinâmico controlado; identificadores allowlist; autorização em procedures; validação de filtros e ordenação |
| C08 | Migrations | schema, DB grants, políticas, deployment | Versionamento, ordem, execução controlada, verificação, integridade de RLS/grants e ensaio em staging |
| C09 | Rollback | deploy, migrations, runbooks | Plano reversível ou forward-fix para mudanças irreversíveis; versões anteriores; restauração testada para schemas/dados quando aplicável |
| C10 | Controle de acesso | rotas, actions, DB/RLS, API, tenant | Default deny; server-side por recurso/operação; isolamento horizontal/vertical, IDOR/BOLA/BFLA, menor privilégio |
| C11 | Expiração de sessão | cookies, JWT, refresh, revogação | `exp`, TTL, refresh rotation conforme arquitetura, logout/revogação, idle timeout para ações sensíveis, cookie Secure/HttpOnly/SameSite |
| C12 | Secrets | `.env`, CI/CD, código, frontend, bundles, logs | Segredos fora do git/client; gerenciador/cofre; escopos mínimos; rotação; credenciais de serviço protegidas; cuidado com histórico |
| C13 | CORS | API, edge, gateway | Allowlist de origens, métodos e headers; `credentials`; tratamento correto de preflight; evitar wildcard incompatível com cookies |
| C14 | Logs e trilha de auditoria | app, proxy, cloud, auth | Eventos críticos, contexto e correlação, retenção, controle de acesso, integridade, ausência de senhas/tokens/PII em claro |
| C15 | Backups | DB, storage, plano de recuperação | Backup automático, retenção, cifragem, acesso limitado, cópia isolada e teste periódico de restauração |
| C16 | Criptografia | tráfego, DB, storage, KMS | TLS em trânsito, proteção em repouso quando necessária, gestão de chaves, rotação, mascaramento; distinguir criptografia e hashing |
| C17 | Dependências | lockfiles, contêineres, SBOM, runtime | Versões fixadas, alertas e CVEs aplicáveis, pacotes abandonados, supply-chain, imagens base e atualizações testadas |
| C18 | PMP — gestão de patches/manutenção (termo a confirmar) | governança, atualização, owners, calendário | Na imagem consta somente “PMP”, sigla ambígua. **Não presuma seu significado oficial**; confirmar com o usuário. Até lá avaliar política de manutenção/patches como eixo complementar provisório |
| C19 | Monitoramento | métricas, alertas, WAF/CDN, erros, SLOs | Detecção de autenticação suspeita, erros 5xx, picos, anomalias, backup falho; responsáveis e escalonamento definidos |
| C20 | Plano de recuperação | continuidade, runbooks, incident response | RTO/RPO definidos, responsável, contingência, comunicação, restauração ensaiada, teste de DR e resposta a incidentes |

Nota: checklist da imagem contempla palavras específicas: “Migrations”, “Rollback”, “PMP” e “Plano de Recuperação”. Adaptar os critérios ao sistema real, sem transformar convenções de projeto em obrigatoriedade universal.

## 2. Cobertura ampliada — verificar conforme a superfície

### Autenticação e identidade
- Enumeração de usuários nos fluxos de login/recuperação; bloqueio indevido; token de redefinição longo/aleatório/uso único/TTL.
- JWT: assinatura/algoritmos permitidos, chaves e audiência/issuer, validação no servidor, revogação quando exigida.
- Sessões fixadas; invalidação após troca de senha/papel; reautenticação para exportação de dados e mudanças financeiras.
- OAuth/OIDC: state/nonce/PKCE onde aplicável, `redirect_uri` estrito, escopos, refresh tokens e proteção contra CSRF de login.

### Autorização e multi-tenancy
- Manipulação de IDs e propriedades no corpo da requisição; ownership de cada recurso; todos os métodos, inclusive export/download/delete.
- Escalonamento horizontal/vertical de privilégios, ações de admin escondidas no frontend, cache compartilhado e endpoints internos expostos.
- Prisma/ORM/RLS: atenção a queries não filtradas por organização e a contas de serviço que ignoram RLS.

### Entrada, saída e lógica de negócio
- XSS armazenado/refletido/DOM, conteúdo markdown/HTML; CSP como defesa adicional, não substituto de escaping.
- SSRF por importação de URL, previews, webhooks; use allowlists e bloqueio de metadados internos quando apropriado.
- Upload: MIME real, limite, vírus quando necessário, filename seguro, storage não executável, autorização de download e links assinados.
- Pagamentos: idempotência, webhooks assinados e replay, fonte de verdade server-side, preços calculados no backend.
- Mass assignment, abuso de paginação, filtros caros, concorrência, TOCTOU, integridade transacional.

### Dados, privacidade e operação
- PII exposta em endpoints, analytics, logs, exceções, backups, exports e snapshots; políticas de retenção, eliminação e auditoria.
- Headers/CSP, cookies, redirecionamentos, cachê privado, CORS, CSRF em autenticação baseada em cookies.
- Infra: IAM mínimo, buckets e storage públicos, portas, permissões de contêiner, secrets de CI, proveniência de artefatos.
- Observabilidade: trilhas de acesso e ações administrativas, alertas acionáveis, métricas de disponibilidade, comportamento anômalo.

## 3. Interpretação específica por tecnologia

### Supabase/PostgreSQL
- Enumerar tabelas, views, buckets, funções RPC e papéis. Verificar se RLS está **habilitada** e se políticas cobrem SELECT/INSERT/UPDATE/DELETE com `USING` e `WITH CHECK` coerentes.
- Checar `auth.uid()` e associação de tenant em cada operação. Checar grants, políticas de storage e funções `SECURITY DEFINER` com search_path e controle de execução.
- Testar com dois usuários fictícios de organizações diferentes em staging quando autorizado. Chaves `anon` publicáveis por design NÃO são segredo; `service_role` é segredo privilegiado.

### Next.js / React
- `NEXT_PUBLIC_` e client bundles; proteger valores sensíveis, não apenas ocultar botões.
- Route Handlers/Server Actions: auth e autorização em cada fronteira, validação de input e tratamento de erros.
- Caching por usuário/tenant, SSR e conteúdo privado, uploads e URLs externas.

### Node.js, Express, NestJS
- Middlewares corretos por rota, DTO + schema validation, trust proxy configurado, requests grandes, erro genérico para cliente e log detalhado interno.
- Bloqueio de queries brutas inseguras, serialização que vaza propriedades privadas e bibliotecas de autenticação sem manutenção.

### WordPress e integrações
- Plugins, temas, contas admins, permissões, atualizações, XML-RPC quando aplicável, REST API, uploads, WAF, backups, exposição de senhas.
- Credenciais/API keys de serviços como pagamento, email, redes sociais e storage devem residir no backend/cofre.

## 4. Checklist rápido antes de produção

1. Há algum achado CRÍTICO/ALTO **confirmado** explorável no cenário de deploy? Se sim, recomendar bloqueio condicionado ao risco, justificar e pedir correção.
2. As permissões server-side e por tenant foram testadas com usuários de teste distintos?
3. Segredos privados estão fora de client bundle/git e com permissões/rotação adequadas?
4. Banco/storage estão fechados por padrão, com políticas verificadas?
5. HTTPS, cookies/sessões, CORS/CSRF e validação de entradas conferem com o mecanismo de autenticação real?
6. Logs, alertas, backups e procedimento de recuperação têm responsáveis definidos e evidência de teste?
7. Existem testes de regressão e plano de rollback compatíveis com a alteração?
8. Itens não verificados e decisões de risco residual estão explícitos no relatório?
