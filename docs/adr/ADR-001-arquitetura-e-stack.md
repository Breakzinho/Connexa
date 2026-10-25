# ADR-001 — Arquitetura e stack do Connexa

- **Situação:** proposta
- **Data:** 2026-10-25
- **Participantes:** equipe Breakzinho

## Contexto

O Connexa é uma plataforma de formação e acompanhamento de grupos de estudo.
A equipe precisa transformar histórias do Azure Boards em um incremento
verificável durante a Sprint 1, com duração de duas semanas, mantendo a
rastreabilidade entre tarefas, testes, commits e pull requests.

O repositório da equipe parte do template .NET + React da disciplina, que já
fornece módulos de domínio, persistência, exemplos de testes, integração
contínua e instruções para o agente de IA. A solução deve permitir dividir o
trabalho por funcionalidade, preservar as fronteiras entre módulos e evitar
complexidade operacional desnecessária para esta etapa do projeto.

A autenticação real está fora do escopo da Sprint 1. O estudante atual será
simulado pelo módulo de identidade, acessado pelos demais módulos por meio
da interface `IUsuarioAtual`.

## Decisão

Adotar a arquitetura de **monolito modular com fatias verticais no back-end e
uma SPA no front-end**, mantendo a solução .NET + React fornecida pelo template.
Organizar as funcionalidades por domínio, com um projeto por módulo no
back-end e uma pasta por funcionalidade no front-end.

### Stack adotada

| Parte | Tecnologias |
|---|---|
| Back-end | C#, .NET 10 e ASP.NET Core Minimal APIs |
| Persistência | Entity Framework Core com Npgsql e PostgreSQL 17 |
| Contrato da API | OpenAPI nativo do ASP.NET Core e Swagger UI |
| Front-end | React 19, TypeScript e Vite |
| Rotas, consultas e formulários | React Router, TanStack Query, React Hook Form e Zod |
| Testes do back-end | MSTest, `WebApplicationFactory` e NetArchTest |
| Testes e verificação do front-end | Vitest, Testing Library, ESLint e compilador TypeScript |
| Ambiente de desenvolvimento | GitHub Codespaces ou máquina local com Docker Compose para o PostgreSQL |
| Integração contínua | GitHub Actions |

Cada módulo do back-end será responsável pelos próprios dados, com contexto
do EF Core, migrações e schema próprios no PostgreSQL. Não haverá acesso
direto às tabelas de outro módulo nem chaves estrangeiras entre módulos.
Tipos em `Internal/` permanecerão internos; integrações usarão as APIs
públicas dos módulos e eventos em processo para reações a acontecimentos.
O projeto `Connexa.Comum` conterá apenas infraestrutura, sem regras de negócio.

No front-end, cada funcionalidade exporá sua API pelo `index.ts`. As chamadas
HTTP passarão pelo cliente compartilhado `requisitar`, preservando o tratamento
padronizado de erros. As versões específicas das dependências permanecerão
nas configurações fornecidas pelo template.

## Alternativas consideradas

| Alternativa | Por que não foi escolhida |
|---|---|
| Templates Java ou Node.js da disciplina | O repositório já foi criado a partir da solução .NET. Mantê-la permite aproveitar a estrutura, os exemplos e as verificações existentes, sem trocar a base durante a preparação da sprint. |
| Microsserviços | Exigiriam implantação, comunicação e observabilidade distribuídas, aumentando o trabalho operacional para o prazo da Sprint 1. |
| Monolito organizado apenas por camadas técnicas globais | A separação por módulos de domínio favorece tarefas por funcionalidade e fronteiras que podem ser verificadas pelo compilador e pelos testes de arquitetura. |

## Consequências

- **Positivas:** a equipe aproveita a infraestrutura do template e concentra o
  esforço nas histórias selecionadas. As fatias verticais permitem entregar
  comportamentos completos, e as fronteiras por módulo reduzem o acoplamento.
  Desenvolvimento e testes usam PostgreSQL, mantendo o mesmo mecanismo de
  persistência nos ambientes local, Codespaces e CI.
- **Negativas:** os módulos compartilham o processo da API e sua implantação;
  não podem ser implantados ou escalados individualmente. A equipe precisa
  manter as fronteiras mesmo com um banco compartilhado, coordenar os
  contratos entre API e SPA e preparar .NET, Node.js e PostgreSQL no ambiente
  local. A sessão simulada não atende aos requisitos de autenticação de produção.
- **Como verificar:** executar `./verificar` ou, no Windows, `.\verificar.ps1`.
  O comando verifica compilação, testes de aceitação, fronteiras entre módulos,
  schemas, lint, tipos, testes de componentes e build do front-end. Na revisão
  do pull request, conferir também a propriedade dos dados por módulo, o uso
  de APIs públicas e eventos e a ausência de autenticação real na Sprint 1.
  Integrar no `main` somente com revisão aprovada e CI verde.
