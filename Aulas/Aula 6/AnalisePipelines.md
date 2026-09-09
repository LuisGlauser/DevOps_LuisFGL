# Pesquisa: Repositórios GitHub com Integração de Pipeline (CI/CD)

## Objetivo

Levantar e analisar repositórios públicos no GitHub que implementam ou contenham integração de **Pipeline** (CI/CD — Integração Contínua e Entrega/Implantação Contínua), destacando para cada um: **características**, **funcionalidades**, **gatilhos (triggers)** e **histórico**.

---

## 1. Harness Open Source

**Repositório:** [github.com/harness/harness](https://github.com/harness/harness)

### Características
- Plataforma de desenvolvimento *end-to-end* que reúne Controle de Código-Fonte (SCM), Pipelines de CI/CD, Ambientes de Desenvolvimento hospedados e Registro de Artefatos em um único produto open source.
- Escrito majoritariamente em **Go**.
- Um dos projetos mais populares da categoria "ci-cd" no GitHub, com dezenas de milhares de estrelas.
- Faz parte de um ecossistema maior mantido pela empresa Harness, com múltiplos repositórios satélites (CLI, SDKs, evals etc.).

### Funcionalidades
- **CI (Integração Contínua):** build, testes automatizados (unitários e de segurança) e empacotamento de imagens de contêiner.
- **CD (Entrega/Implantação Contínua):** implantação progressiva entre ambientes (dev → staging → produção), com estratégias de *canary* e *blue-green deployment*.
- **GitOps:** integração com Open Policy Agent (OPA) para validação e governança de deployments baseados em Git.
- **Feature Flags:** gerenciamento de lançamento gradual de funcionalidades e experimentação (Feature Management & Experimentation).
- **Cache de builds:** suporte a cache inteligente (ex.: Azure Blob Storage com autenticação OIDC) para acelerar builds Go e outros.
- Pipelines definidos via configuração declarativa (YAML), com contadores de versão independentes por branch.

### Gatilhos (Triggers)
- **Eventos Git:** push, pull request/merge, entre outros eventos de repositórios GitHub, GitLab, Bitbucket, Azure Repos e "Harness Code" (nativo).
- **Gatilhos por artefato:** disparo automático de pipeline quando uma nova versão de artefato é publicada em um registro (ex.: nova imagem Docker no Docker Hub).
- **Gatilhos customizados (Custom Trigger):** para provedores de repositório não suportados nativamente.
- **Execução manual** pela interface do Harness ("Run Pipeline").
- Suporta condições específicas de payload (ex.: apenas PRs para determinada branch) para tornar os gatilhos mais granulares.

### Histórico
- Nasceu como oferta comercial da empresa Harness e, posteriormente, deu origem ao projeto **Harness Open Source**, disponibilizando gratuitamente as capacidades centrais de CI/CD, SCM e ambientes de desenvolvimento.
- Recebe atualizações frequentes (releases em 2026 trouxeram melhorias de governança GitOps, cache de build e execução de pipelines em contêiner).
- Mantém repositórios auxiliares ativos (ex.: `harness/cli`, SDKs de feature flag), evidenciando desenvolvimento contínuo e comunidade ativa.

---

## 2. Moodle

**Repositório:** [github.com/moodle/moodle](https://github.com/moodle/moodle)

### Características
- **Moodle** é a maior plataforma de aprendizagem (LMS — Learning Management System) open source do mundo, escrita principalmente em **PHP**, licenciada sob **GPL-3.0**.
- Projeto muito antigo e de grande escala, com milhares de estrelas e milhares de forks no GitHub, refletindo uma comunidade global de desenvolvedores, escolas e universidades.
- Particularidade importante: o repositório `moodle/moodle` no GitHub é oficialmente um **espelho (mirror)** do repositório principal, que fica hospedado em `git.moodle.org`. O `CONTRIBUTING.txt` do projeto deixa isso explícito, orientando a não abrir Pull Requests diretamente pelo GitHub.
- Possui um ecossistema próprio de ferramentas de CI mantidas pela organização **moodlehq** (ex.: `moodle-plugin-ci`, `moodle-ci-runner`, `moodle-docker`, `moodle-behat-extension`), além de *workflows* reutilizáveis mantidos pela comunidade (ex.: `catalyst-moodle-workflows`).

### Funcionalidades
- **PHPUnit:** testes unitários automatizados do núcleo e dos plugins.
- **Behat:** framework de testes de aceitação orientado a comportamento (BDD), que simula interações reais de usuário (via navegador headless ou Selenium/Chrome/Firefox) para validar funcionalidades ponta a ponta.
- **moodle-ci-runner:** ferramenta que padroniza a execução de jobs de PHPUnit e Behat dentro de qualquer ambiente de CI, organizada em módulos (docker, git, php, logs, env), garantindo consistência entre diferentes provedores de pipeline.
- **Matriz de testes (test matrix):** execução cruzada contra múltiplas versões de PHP (ex.: 8.1, 8.2, 8.3, 8.4), diferentes bancos de dados (PostgreSQL, MariaDB, MySQLi) e diferentes branches estáveis do Moodle (ex.: `MOODLE_401_STABLE`, `MOODLE_500_STABLE`).
- **Git bisect automatizado:** o `moodle-ci-runner` permite definir um commit "bom" e um "ruim" (`GOOD_COMMIT`/`BAD_COMMIT`) para que o próprio pipeline identifique automaticamente qual commit introduziu uma regressão.
- **moodle-docker:** conjunto de containers Docker que reproduz um ambiente de teste completo (banco de dados, Selenium, SMTP fake, extensões PHP), usado tanto localmente quanto em CI para garantir reprodutibilidade.
- **moodle-plugin-ci:** ferramenta baseada em Composer, amplamente adotada por milhares de plugins do ecossistema Moodle, que empacota lint de código, verificação de mustache/grunt, PHPUnit e Behat em um único comando.
- Cobertura de código integrada via **Codecov** em ferramentas satélite como o próprio `moodle-ci-runner`.

### Gatilhos (Triggers)
- **No núcleo (core):** o fluxo de integração não é um simples "push dispara build". Correções e novas funcionalidades passam primeiro pelo **Moodle Tracker** (sistema de issues próprio) e por revisão de pares; só então a equipe de **Integradores** da Moodle HQ "puxa" o código para o repositório de integração, onde um servidor **Jenkins** hospedado em `ci.moodle.org` (usando o `moodle-ci-runner`) executa PHPUnit e Behat automaticamente a cada integração — um gatilho híbrido, combinando processo humano de revisão com automação de testes.
- **No ecossistema de plugins** (milhares de repositórios de terceiros): o gatilho é o modelo clássico do GitHub Actions, `on: [push, pull_request]`, acionando *workflows* reutilizáveis (ex.: `catalyst-moodle-workflows` ou a action `moodle-plugin-ci-action`) a cada commit ou pull request.
- **Testes do aplicativo móvel (Moodle App):** toda nova funcionalidade que entra no core dispara, via Jenkins, a suíte completa de testes do app contra a versão mais recente, funcionando como um gatilho de regressão cruzada entre projetos.
- Diversos exemplos de configuração mostram gatilhos condicionados a variáveis (ex.: habilitar/desabilitar `phpunit`, `grunt`, `release` por meio de *flags* no workflow), permitindo pipelines mais granulares por plugin.

### Histórico
- O projeto Moodle foi criado em **2002** por Martin Dougiamas e é mantido desde então por uma comunidade global, coordenada pela **Moodle HQ** (Austrália) e por moodle.org.
- O histórico de automação de testes evoluiu em camadas: primeiro scripts manuais de PHPUnit/Behat via linha de comando, depois integração com **Jenkins** (ainda em uso hoje em `ci.moodle.org`), e mais recentemente forte adoção de **GitHub Actions** no ecossistema de plugins via ferramentas como `moodle-plugin-ci` e *reusable workflows*.
- Arquivos de configuração de CI do ecossistema (como o `config.json` usado por workflows da comunidade) evidenciam a evolução constante da matriz de compatibilidade — por exemplo, o suporte a `MOODLE_500_STABLE` com PHP 8.2–8.4 e três bancos de dados, atualizado conforme novas versões do Moodle são lançadas.
- A cobertura de testes automatizados (PHPUnit e/ou Behat) é **obrigatória** para qualquer novo recurso aceito no core, o que reforça o papel do pipeline de CI como um "portão de qualidade" formal, e não apenas uma conveniência de desenvolvimento.

---

## 3. Multilang CI/CD Pipeline (DevOpsProjectsLab)

**Repositório:** [github.com/DevOpsProjectsLab/multilang-pipeline](https://github.com/DevOpsProjectsLab/multilang-pipeline)

### Características
- Projeto educacional/demonstrativo, criado no âmbito da organização **DevOpsProjectsLab**, com foco em boas práticas modernas de CI aplicadas a ambientes **multi-stack**.
- Combina duas linguagens no mesmo pipeline: **Python** (back-end/API) e **Node.js** (front-end).
- Repositório pequeno e recente (22 commits), ideal para fins de estudo e portfólio, em contraste com os dois exemplos anteriores (projetos de grande porte).
- Pipeline definida em arquivo único `ci.yml` dentro de `.github/workflows/`.

### Funcionalidades
- **GitHub Actions** como motor de CI.
- **Matrix strategy:** permite executar os jobs de Python e Node.js em **paralelo**, cada um em seu próprio ambiente isolado, otimizando performance e rastreabilidade dos resultados.
- **Testes automatizados:** Pytest para o back-end Python; Vitest + React Testing Library para o front-end Node.js.
- **Relatórios visuais:** uso da action `dorny/test-reporter` para exibir resultados de teste diretamente na interface do GitHub Actions.
- **Cobertura de código:** integração com **Codecov** para análise e acompanhamento da cobertura de testes.
- Estrutura de repositório organizada, separando claramente `back-end/` e `front-end/`, além dos workflows e assets de documentação.

### Gatilhos (Triggers)
- Workflows do GitHub Actions tipicamente disparados por eventos `push` e `pull_request` (padrão comum em projetos com este tipo de badge/estrutura, incluindo o de "CI Status" exibido no README).
- Execução paralela via matrix strategy pode ser entendida como "múltiplos sub-gatilhos" dentro de uma mesma execução: cada combinação da matriz (ex.: linguagem/versão) gera um job independente.

### Histórico
- Projeto recente, parte de uma organização (DevOpsProjectsLab) dedicada ao estudo e prática de conceitos DevOps modernos.
- A organização mantém outros repositórios com propósito semelhante (ex.: `docker-multistage`, que compara imagens Docker otimizadas vs. não otimizadas usando Trivy e Hadolint em uma pipeline CI/CD; e `service-monitoring`, com Prometheus e Docker Compose), evidenciando uma linha consistente de projetos práticos de DevOps/CI-CD.
- Por ser um repositório pequeno e ativo, seu histórico de commits reflete iterações incrementais típicas de projetos de aprendizado (configuração inicial do workflow, adição de badges, integração de ferramentas de cobertura e relatório).

---

## Quadro Comparativo

| Critério | Harness | Moodle | Multilang Pipeline |
|---|---|---|---|
| Porte do projeto | Grande (plataforma completa) | Muito grande (LMS global, +20 anos) | Pequeno (projeto didático) |
| Linguagem principal | Go | PHP | YAML (GitHub Actions) + Python/Node.js |
| Modelo de gatilho | Webhook de SCM, artefato, manual | Híbrido: revisão humana + Jenkins (core); push/PR via GitHub Actions (plugins) | Push/Pull Request (GitHub Actions) |
| Foco | CI + CD + GitOps + Feature Flags | Qualidade e regressão via PHPUnit/Behat em larga escala | CI simples multi-linguagem |
| Maturidade | Alta, empresa por trás | Altíssima, comunidade global desde 2002 | Recente, uso educacional |

---

## Conclusão

A análise dos três repositórios evidencia diferentes escalas e abordagens de integração de pipeline:

1. **Harness** representa uma solução robusta e comercial-open-source, cobrindo todo o ciclo de vida de entrega de software, com gatilhos flexíveis baseados em eventos Git e artefatos.
2. **Moodle** mostra como um projeto open source de grande escala e longa história combina processo humano (revisão de pares, integração controlada pela Moodle HQ) com automação robusta de testes (PHPUnit, Behat, Jenkins e, no ecossistema de plugins, GitHub Actions), tratando o pipeline de CI como um verdadeiro portão de qualidade obrigatório.
3. O projeto **Multilang Pipeline** ilustra, em escala reduzida, como aplicar conceitos de CI/CD com GitHub Actions em um cenário real de aplicação multi-stack, servindo como bom exemplo prático e didático de pipeline simples, mas funcional.

Em todos os casos, o elemento comum é a automação: mudanças no código-fonte (via commits, pull requests ou eventos programados) disparam automaticamente processos de build, teste e, em alguns casos, implantação — o núcleo conceitual de qualquer pipeline de CI/CD.
