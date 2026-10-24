# SENTINELA — 21 verificações práticas obrigatórias de segurança

> Esta matriz complementa — e NÃO substitui — os 20 controles estruturais `C01–C20` de `controles.md`. Aplique os três conjuntos (C01–C20, A01–A21 e B01–B20) em toda auditoria completa, consultando também `controles-avancados-20.md`. Evite duplicar o mesmo achado; vincule cada achado a todos os controles que cobre.

## Protocolo comum de avaliação

Para **cada um** dos `A01–A21`, registrar: `CONFORME`, `FALHA`, `NÃO VERIFICADO` ou `NÃO SE APLICA` (com justificativa); superfície analisada; fonte de evidência (`arquivo:linha`, configuração, migração, política, teste autorizado, ambiente); confiança (`CONFIRMADO`, `PROVÁVEL`, `HIPÓTESE`); impacto e correção; teste de aceite. Não atribua conformidade apenas pela existência de biblioteca, `middleware`, `.gitignore`, comentário, interface bonita ou login visual.

Um controle pode estar conforme no código inspecionado e **não verificado na implantação**: especifique o alcance e não esconda essa lacuna. Leia e confira o contexto antes de rotular como falha.

## Matriz de inspeção e critérios de aceite

### A01 — Chave de API protegida
- **Inspecione:** código, bundles JavaScript, commits, CI/CD, variáveis de ambiente, edge functions, logs, endpoints proxy, arquivos de configuração e permissões das chaves.
- **Falha:** segredo com privilégio foi exposto no cliente ou repositório; chave universal usada sem restrições ou em contextos indevidos.
- **Verifique/aceite:** segredos somente em servidor ou cofre; escopo mínimo, rotacionados se vazados, com controles de uso. Diferencie chave **publicável/identificadora** de chave **secreta** (ex.: Supabase publishable/legacy `anon` e configuração cliente do Firebase podem ser públicas; Supabase `service_role`/secret, chaves privadas e Admin SDK não).
- **Teste seguro:** inspeção estática de bundle e configuração, sem reproduzir ou divulgar valor secreto. Se houver vazamento, mascarar, orientar revogação e investigar uso indevido.
- **Base:** C12, C10, C17.

### A02 — `.env` nunca exposto
- **Inspecione:** `.env*`, `.gitignore`, build outputs, public root, Docker/artefatos, arquivos estáticos, configuração da hospedagem, anexos e logs de CI.
- **Falha:** arquivo contendo segredos publicado no Git, pacote cliente, artefato público ou servido por HTTP.
- **Verifique/aceite:** `.env` real excluído de versionamento e publicação, injeção segura de valores por ambiente e proteção contra arquivos dotfiles; `.env.example` sem segredos pode ficar no repositório. `NEXT_PUBLIC_*`, `VITE_*`, `PUBLIC_*` são públicos no bundle: usar apenas para dados próprios para exposição.
- **Teste seguro:** examinar artefatos e regras de servidor autorizadas, sem pedir chaves verdadeiras.
- **Base:** C12, C17.

### A03 — Nada de senha no código
- **Inspecione:** fixtures, scripts, testes, migrations, URLs de conexão, strings e históricos do Git.
- **Falha:** credenciais reais hardcoded ou defaults utilizáveis em produção; hash/senha fixa para entrar no admin.
- **Verifique/aceite:** credenciais de operação vêm de cofre/variáveis secretas, sem valores de fallback inseguros; contas de exemplo inertes e claramente fictícias.
- **Teste seguro:** busca local com valores sempre mascarados; orientar rotação se houver exposição.
- **Base:** C02, C12.

### A04 — Login de verdade
- **Inspecione:** telas, endpoints, middleware, sessão, JWT, OAuth, refresh, logout, recuperação de senha e validação de identidade.
- **Falha:** senha conferida no browser, autenticação simulada, `localStorage` como prova de identidade, token não validado no servidor, rota que confia apenas em flag `isLoggedIn` do cliente.
- **Verifique/aceite:** autenticação server-side/provedor confiável, validação de assinatura/expiração/issuer/audience quando aplicável, sessões protegidas e encerramento coerente; MFA onde o risco exigir; proteção contra CSRF para autenticação por cookies em ações mutáveis.
- **Teste seguro:** sem sessão => acesso protegido negado; sessão inválida => negada; usuário válido => acesso somente autorizado.
- **Base:** C02, C03, C10, C11.

### A05 — Permissão no servidor
- **Inspecione:** controllers, server actions, funções de backend, RPCs, webhooks, policies e checagem de papéis/grants.
- **Falha:** botão oculto ou `if (user.role)` no frontend é único bloqueio; middleware cobre tela mas não endpoints associados.
- **Verifique/aceite:** cada operação de leitura/criação/edição/exclusão verifica identidade, papel e autorização por recurso no servidor ou camada de dados que de fato execute a regra; `deny by default`.
- **Teste seguro:** usuário comum tentando operação administrativa com conta teste é rejeitado sem mutações.
- **Base:** C10.

### A06 — Não confiar no ID da tela
- **Inspecione:** IDs em URL, query string, body, cabeçalhos, hidden inputs, organização/tenant enviado pelo cliente, RPCs e query de detalhe.
- **Falha:** acesso por `GET /items/{id}` ou mutação aceita ID arbitrário sem verificar pertencimento ou vínculo autorizado.
- **Verifique/aceite:** servidor obtém identidade de credencial verificada e consulta relacionamentos/autorização para o recurso solicitado; alterações de owner/role/org pelo cliente são rejeitadas.
- **Teste seguro:** com duas contas fictícias, trocar o ID de A pelo de B retorna 403/404 sem vazar metadados; operações legítimas continuam funcionando.
- **Base:** C10.

### A07 — Cada um só vê o seu
- **Inspecione:** consultas de lista, busca, detalhes, exportações, downloads, websocket/realtime, cache, índices, jobs e joins multiempresa.
- **Falha:** resultados cruzam contas/organizações ou a regra protege detalhe mas não listagem/exportação; `OR`/joins quebram isolamento.
- **Verifique/aceite:** isolamento horizontal por `user_id`/`tenant_id` validado no backend e/ou RLS; acessos compartilhados somente por relacionamento explícito e verificado.
- **Teste seguro:** matriz de duas contas e duas organizações, cobrindo leitura, gravação, exportação, pesquisa e arquivo; testar permissão positiva do dono e negativa do estranho.
- **Base:** C10, C05.

### A08 — Banco travado
- **Inspecione:** grants, papéis, RLS/policies, views, functions, migrations, usuários de conexão, exposição de portas, backups e recursos públicos.
- **Falha:** cliente pode acessar tabela sem controles, role com privilégio excessivo, policies permissivas (`true`) em dados privados, procedures que ignoram autorização.
- **Verifique/aceite:** privilégio mínimo, roles segregadas, defaults restritivos, RLS nos objetos expostos quando aplicável e validação segura das funções com privilégios elevados. **RLS e GRANT são camadas diferentes.**
- **Teste seguro:** revisar políticas/migrations e plano de acesso com credenciais fictícias, sem apagar nem modificar dados reais.
- **Base:** C07, C08, C10, C12.

### A09 — Firebase, Supabase e Storage
- **Supabase:** inspecionar `anon`/publishable versus `service_role`/secret, grants de schemas, RLS em todas as tabelas expostas, `USING` e `WITH CHECK` para `SELECT/INSERT/UPDATE/DELETE`, RPC `SECURITY DEFINER`, Edge Functions, JWT e policies de `storage.objects`. Chave de serviço/secret que bypassa RLS nunca no navegador. Storage público somente quando intencional, inclusive listagem e link de download.
- **Firebase:** inspecionar Firebase Auth, Firestore Rules, Realtime Database Rules, Storage Rules, Cloud Functions e Admin SDK/IAM. Configuração Web client não é segredo por si só; regras públicas amplas são problema. **Admin SDK/servidores privilegiados podem contornar Rules**, exigindo checagem no código de servidor. App Check é defesa contra abuso complementar, não substitui Auth/Rules.
- **Storage genérico:** bucket público, ACLs, pasta por dono, link assinado com validade, permissões de `read/list/write/delete`, transformação/preview, separação de dados privados e públicos.
- **Verifique/aceite:** evidências por **tabela/coleção/bucket e operação**, não apenas flag global; negativos entre usuários e tenants em staging.
- **Base:** C10, C12, C15, C16.

### A10 — Admin protegido
- **Inspecione:** páginas `/admin`, APIs, server actions, RPCs, jobs, funções de elevação, impersonação, exportações e políticas de acesso.
- **Falha:** UI restrita, mas API administrativa aceita chamada direta; `admin=true` enviado em body ou token falso; papel lido do perfil editável pelo próprio usuário.
- **Verifique/aceite:** identidade e permissão privilegiada validadas em todas as rotas/recursos; MFA e reautenticação em operações sensíveis quando apropriado; trilha de auditoria; papéis alteráveis apenas por administradores autorizados.
- **Teste seguro:** conta comum e sessão anônima não acessam endpoints de admin.
- **Base:** C03, C10, C14.

### A11 — Debug desligado
- **Inspecione:** build de produção, flags (`DEBUG`, `NODE_ENV`), source maps acessíveis, consoles administrativos, devtools, exception pages, endpoints de diagnóstico e logs.
- **Falha:** stack trace, interface debug ou painel de diagnóstico exposto a usuários externos sem controle.
- **Verifique/aceite:** modo produção correto, debug restrito/ausente publicamente, diagnósticos internos autenticados, telemetria privada. Source maps exigem avaliação contextual (não constituem automaticamente uma vulnerabilidade).
- **Base:** C14, C19.

### A12 — Erro sem detalhe interno
- **Inspecione:** tratamento global de exceções, endpoints e integrações.
- **Falha:** resposta HTTP inclui SQL, credenciais, caminhos internos, dump de ambiente, stack trace ou PII desnecessária.
- **Verifique/aceite:** mensagem externa genérica adequada; ID de correlação; detalhes em log privado sanitizado; distinção clara entre 4xx e 5xx sem revelar acesso de outros usuários.
- **Teste seguro:** entradas inválidas em testes unitários/integrados e inspeção de handlers; sem forçar exceções em produção.
- **Base:** C05, C14.

### A13 — Validar tudo no servidor
- **Inspecione:** rotas, schemas, parsers, IDs, tipos, limites, enums, limites numéricos, estados de negócio e webhooks.
- **Falha:** `required`, `minLength` ou tipo validados apenas na interface; preço, papel, status ou quantidade aceitos da tela sem rechecagem.
- **Verifique/aceite:** schema server-side em todos os limites de entrada, dados normalizados quando adequado, campo sensível derivado do backend; mensagens coerentes.
- **Teste seguro:** casos inválidos, ausentes, tamanhos excessivos e limites de negócio com dados fictícios.
- **Base:** C05, C10.

### A14 — Limpar o que o usuário manda
- **Inspecione:** textos ricos, HTML, Markdown, comentários, nomes de arquivos, CSV, e-mail, logs, templates e páginas que exibem conteúdo do usuário.
- **Falha:** concatenação insegura em HTML/JS/CSS/URL/SQL/log ou execução de scripts armazenados.
- **Verifique/aceite:** **escaping/encoding de saída por contexto** é central contra XSS; sanitizar apenas HTML rico permitido com biblioteca confiável/allowlist; consultas SQL sempre parametrizadas. Não recomendar remover caracteres indiscriminadamente (isso não resolve injeções e pode corromper dados).
- **Teste seguro:** strings inertes de teste e revisão de sinks (`innerHTML`, template literal, comandos etc.).
- **Base:** C05, C06, C07.

### A15 — Upload protegido
- **Inspecione:** formulário, endpoint, upload direto ao bucket, assinatura, extensão/MIME/conteúdo, tamanho, nome, caminho, armazenamento e download.
- **Falha:** upload sem autenticação/autorização quando necessário, tipos executáveis indevidos, paths arbitrários, links públicos não intencionais, leitura cruzada de arquivos.
- **Verifique/aceite:** allowlist de tipos realmente aceitos, verificação de conteúdo adequada ao risco, limites de tamanho e quantidade, nome/chave gerados pelo servidor, política de bucket por proprietário, proteção contra malware quando pertinente, processamento seguro e storage não executável.
- **Teste seguro:** arquivo fictício pequeno permitido e arquivos inválidos inofensivos, sem malware real.
- **Base:** C04, C05, C10, C12.

### A16 — Sem injeção de SQL
- **Inspecione:** query builders, ORMs, concatenação de strings, filtros, `orderBy`, migrations, RPC e raw SQL.
- **Falha:** entrada não confiável concatenada como SQL; identifier dinâmico não controlado; execução de SQL livre via API.
- **Verifique/aceite:** bind parameters/prepared statements, allowlists para nomes de colunas/ordenação, acesso mínimo e testes de regressão. ORM presente não é garantia; `raw` inseguro e procedures precisam de revisão.
- **Teste seguro:** análise estática e testes de integração com entradas artificiais não destrutivas; nada de exploração de produção.
- **Base:** C07.

### A17 — Limite de tentativas
- **Inspecione:** login, reset, OTP, MFA, convite, upload, busca, autenticação da API e endpoints de custo elevado.
- **Falha:** ausência de limitação viável em endpoint exposto a abuso; enumeração, tentativas ou gasto automatizado sem proteção.
- **Verifique/aceite:** rate limiting contextual por conta, endereço de origem e sinais adicionais; resistência a proxy spoofing, limites por rota, retorno 429/retry quando apropriado; regras anti-enumeração e sem bloqueio permanente por terceiro.
- **Teste seguro:** inspecionar código/config e validar com simulação/local ou poucos acessos em staging, nunca força bruta.
- **Base:** C04, C19.

### A18 — Git sem senha vazada
- **Inspecione:** arquivos rastreados, branches relevantes, commits acessíveis, exemplos, segredos em workflows, artefatos e logs de CI.
- **Falha:** token, senha, private key ou URI de banco real em qualquer histórico/artefato público ou acessível indevidamente.
- **Verifique/aceite:** secret scanning local/CI autorizado, `.gitignore` adequado, proteção de branch, permissões de CI, rotação/revogação de material comprometido. **Remover do commit atual não invalida segredo já vazado**; limpeza do histórico é tarefa supervisionada que não substitui rotação.
- **Teste seguro:** ferramentas autorizadas e saída mascarada; jamais republicar segredos completos.
- **Base:** C12, C17.

### A19 — Headers e CORS corretos
- **Inspecione:** respostas de produção/staging autorizados, reverse proxy, cookies, CSP, HSTS, framing, cache, content-type e preflight.
- **Falha:** `Access-Control-Allow-Origin` reflexivo/permissivo com credenciais, CSP ausente em cenário relevante, cache de conteúdo privado compartilhado, cookies sem flags protetivas aplicáveis, headers inconsistentes.
- **Verifique/aceite:** CORS restrito conforme chamadas legítimas, validação de origem no backend quando necessário, `Vary: Origin` quando apropriado, CSP testada, `X-Content-Type-Options: nosniff`, `frame-ancestors`, `Referrer-Policy`, cookies `Secure`/`HttpOnly`/`SameSite` segundo contexto, HSTS somente quando todo o domínio estiver apto. CORS **não é autenticação/autorização** e não protege clientes não navegador.
- **Base:** C01, C11, C13.

### A20 — Testar como um estranho
- **Inspecione:** toda superfície acessível anonimamente, usuário comum, outras organizações e conta admin de teste; comparar com comportamento do usuário dono.
- **Método seguro:** em repositório/testes locais ou **staging explicitamente autorizado**, simular três perfis: `ANÔNIMO`, `USUÁRIO A`, `USUÁRIO B` (+ `ADMIN DE TESTE` se necessário); tentar **somente operações não destrutivas ou simulações** no próprio conjunto de dados fictícios; mudar IDs e chamadas de acesso; verificar listagem, detalhes, downloads, buscas, uploads, exportações, RPC e interfaces administrativas.
- **Verifique/aceite:** anônimo vê apenas o público; A não enxerga/modifica B; B não enxerga/modifica A; admin cumpre política definida; toda tentativa é rastreável. 401/403/404 são aceitáveis conforme contrato e política de não enumeração.
- **Limites:** nunca executar contra terceiro ou produção sem autorização, escopo e controle de carga. Não forçar autenticação nem explorar dados reais.
- **Base:** C10, C11, C14.

### A21 — Auditoria completa com IA, verificável
- **Inspecione:** arquitetura, front, backend, rotas, papéis, APIs, banco, storage, CI/CD, dependências, configuração, deploy e modelo de ameaça. Combine revisão estática e ferramentas autorizadas, não somente opinião de IA.
- **Verifique/aceite:** executar fluxo `mapear -> classificar superfícies -> verificar C01–C20 e A01–A21 -> evidenciar -> tentar refutar -> priorizar -> corrigir sob aprovação -> retestar`. Faça rastreamento arquivo:linha, referência a configuração/policy ou teste realizado de verdade. Registre componentes não acessados, falsos positivos, hipóteses, cobertura de ferramentas, versão/commit e status da evidência.
- **Entrega obrigatória:** tabela `A01–A21`, principais achados `SEC-###`, plano P0/P1/P2, testes negativos/positivos e veredito de deploy condicionado ao escopo. Se faltar código/acesso, documente como `NÃO VERIFICADO` e **não simule testes**.
- **Base:** todos C01–C20, evidencias.md, relatorio.md.

## Matriz de cenários mínimos para isolamento (somente autorização e ambiente de teste)

| Cenário | Identidade | Recurso | Expectativa segura |
|---|---|---|---|
| Público | anônima | dado público intencional | acesso conforme regra declarada |
| Protegido sem login | anônima | rota privada | bloqueado |
| Próprio | usuário A | item de A | permitido conforme ação/papel |
| ID trocado | usuário A | item de B | negado sem vazar dados |
| Lista/pesquisa | usuário A | itens de A e B | apenas escopo permitido |
| Mutação | usuário A | item de B | negado, sem alteração |
| Exportação/download | usuário A | arquivo de B | negado, sem revelar URL privada |
| Admin | usuário A sem privilégio | ação de admin | negado |
| Admin autorizado | admin de teste | ação autorizada | permitido e auditado |

> Registre a conta e o ambiente de teste **sem registrar senha, token ou identificadores pessoais reais**. Se alguma operação ativa não for autorizada, apenas gere plano de testes e marque `NÃO VERIFICADO`.

## Bloqueadores sugeridos para produção (quando confirmados e aplicáveis)

- Segredo privilegiado exposto e ainda válido.
- Falha de autenticação que permite acesso a recursos privados sem identidade validada.
- IDOR/BOLA ou isolamento entre usuários/empresas quebrado com acesso real a dados privados.
- Permissões administrativas burláveis ou SQL injection com impacto relevante.
- Storage/DB de dados privados exposto publicamente contra a política do produto.
- Ausência de validação/permite upload explorável com impacto severo.

Não interprete `NÃO VERIFICADO` como aprovado. Bloqueio de deploy sempre deve considerar evidência, superfície real, criticidade e medidas mitigadoras; nunca alegar que completou testes que não realizou.

## Referências oficiais para consulta quando houver navegação

- OWASP ASVS e Cheat Sheet Series: https://owasp.org/www-project-application-security-verification-standard/ e https://cheatsheetseries.owasp.org/
- Supabase — Secure data: https://supabase.com/docs/guides/database/secure-data
- Supabase — RLS: https://supabase.com/docs/guides/database/postgres/row-level-security
- Supabase — Storage access control: https://supabase.com/docs/guides/storage/security/access-control
- Firebase — Rules e exemplos inseguros: https://firebase.google.com/docs/rules/insecure-rules?hl=pt-BR
