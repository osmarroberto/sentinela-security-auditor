# SENTINELA 3.0 — Auditor de Segurança para Claude

**Skill em português para auditoria de segurança de aplicações, APIs, bancos de dados e infraestrutura.**

O SENTINELA orienta o Claude a revisar código e configurações, identificar riscos de segurança com evidências, priorizar correções e documentar o que foi ou não verificado. O projeto reúne **61 verificações** em **3 matrizes de auditoria**.

> **Importante:** este projeto é um conjunto de instruções para um agente de IA. Não é um scanner autônomo de vulnerabilidades, não concede acesso a servidores e não garante que uma aplicação esteja segura. A profundidade da análise depende dos arquivos, ferramentas e ambientes disponibilizados ao agente.

## Downloads

> **Publicação em andamento:** o código-fonte, as matrizes e a documentação já estão disponíveis. O ZIP instalável e o PDF serão enviados separadamente; os links binários ainda não estão ativos.


| Arquivo | Finalidade |
|---|---|
| **[Baixar código da Skill (pasta)](sentinela-seguranca/)** | Arquivos-fonte de instalação e referência; use **Code → Download ZIP** para baixar o repositório |
| **Guia de capacidades (.pdf)** | Publicação do PDF ainda pendente |
| **[Prompt completo (.md)](docs/PROMPT_COMPLETO_SENTINELA_CLAUDE_V3.md)** | Instruções integrais para leitura e adaptação |
| [SKILL.md](sentinela-seguranca/SKILL.md) | Instruções principais da Skill |

## O que a Skill verifica

### 1. Controles estruturais — 20 itens

HTTPS/TLS; armazenamento seguro de senhas; MFA; rate limiting; validação de entradas; tratamento seguro de conteúdo; SQL/NoSQL injection; migrations; rollback; controle de acesso; expiração de sessão; segredos; CORS; logs; backups; criptografia; dependências; manutenção e atualizações; monitoramento; recuperação de incidentes.

### 2. Controles práticos — 21 itens

Proteção de chaves de API e `.env`; senhas no código e no Git; login validado no servidor; autorização no backend; IDOR/BOLA; isolamento por usuário; regras de acesso ao banco; **Supabase RLS, Firebase Security Rules e Storage**; proteção de administradores; debug; tratamento de erros; validação e tratamento de dados; uploads; SQL parametrizado; bloqueio de tentativas; headers de segurança; simulação de usuário não autorizado; auditoria assistida por IA com evidências.

### 3. Controles avançados — 20 itens

Sessões e tokens; OAuth e recuperação de contas; XSS/CSRF; SSRF; lógica de negócio; concorrência e transações; permissões por campo; webhooks; cadeia de suprimentos de software; CI/CD; infraestrutura e IAM; criptografia avançada; resposta a incidentes; restauração de backups; consumo abusivo de recursos; cache/jobs/realtime; privacidade e LGPD; segurança de sistemas com IA; processamento de arquivos; regressões de segurança.

> O checklist totaliza 61 verificações, mas alguns controles são deliberadamente complementares e parcialmente sobrepostos. Um achado só deve ser descrito como confirmado quando houver evidência adequada.

## Instalação no Claude

1. Acesse a pasta [`sentinela-seguranca/`](sentinela-seguranca/) e baixe os arquivos-fonte. Para importar como Skill no Claude, compacte **somente essa pasta**, preservando o `SKILL.md` e a pasta `referencias/`.
2. Abra o Claude e localize a área de personalização e gerenciamento de Skills (a localização exata pode mudar conforme o plano e a interface).
3. Importe o arquivo ZIP como uma Skill personalizada.
4. Abra um projeto ou forneça os arquivos do sistema que deseja auditar.
5. Inicie com o prompt do exemplo abaixo.

**Alternativa:** explore os arquivos em [`sentinela-seguranca/`](sentinela-seguranca/) e use o conteúdo de [`SKILL.md`](sentinela-seguranca/SKILL.md) como referência para configurar um agente manualmente.

## Exemplo de uso

```text
Ative a Skill SENTINELA 3.0 e realize uma auditoria de segurança completa
neste projeto. Avalie as 61 verificações distribuídas nas três matrizes.

Analise o código, as rotas, os mecanismos de autenticação e autorização,
as políticas do banco de dados, o storage, as integrações e a publicação.

Priorize possíveis acessos indevidos entre usuários, falhas de lógica
de negócio, credenciais expostas e configurações inseguras.

Não altere arquivos inicialmente. Entregue um relatório com evidências,
classificação de risco, localização dos problemas, ações propostas e
controles que não puderam ser verificados. Solicite autorização antes
de testes ativos e de qualquer alteração.
```

Mais exemplos em [`exemplos/`](exemplos/).

## Como interpretar os resultados

A Skill deve distinguir:

- **Confirmado:** há evidência suficiente de uma falha ou de um controle implementado.
- **Suspeita:** existe um indício, mas faltam provas para confirmar.
- **Não verificado:** não houve acesso ou dados suficientes para concluir.
- **Não aplicável:** o controle não se aplica ao sistema analisado.

A revisão deve começar por análise passiva e estática. Testes ativos só devem ocorrer **em ambientes e ativos sob sua administração ou com autorização explícita**, respeitando escopo, limites e impacto operacional.

## Tecnologias contempladas

React, Next.js, Node.js, Express, NestJS, Supabase, PostgreSQL, Firebase, WordPress, GitHub, GitHub Actions, Netlify, Vercel e aplicações com integrações de IA, entre outras. A cobertura real varia conforme a configuração do projeto.

## Estrutura do repositório

```text
sentinela-security-auditor/
├── README.md
├── CHANGELOG.md
├── SECURITY.md
├── .gitignore
├── sentinela-seguranca/
│   ├── SKILL.md
│   └── referencias/
├── downloads/
│   └── (ZIP instalável: publicação pendente)
├── docs/
│   ├── PROMPT_COMPLETO_SENTINELA_CLAUDE_V3.md
│   └── (PDF: publicação pendente)
└── exemplos/
    └── exemplo-auditoria.md
```

## Aviso de segurança e responsabilidade

O SENTINELA é uma ferramenta de apoio à auditoria defensiva. Não substitui testes de intrusão profissionais, revisão humana ou validações independentes. Não publique chaves, tokens, senhas nem relatórios contendo informações sensíveis neste repositório.

## Licenciamento

**Licença de distribuição e reutilização a definir pelo mantenedor.** A disponibilidade pública para visualização e download no GitHub não equivale automaticamente a uma licença de redistribuição ou modificação. Antes de divulgar oficialmente como software livre, adicione uma licença compatível com a finalidade do projeto.