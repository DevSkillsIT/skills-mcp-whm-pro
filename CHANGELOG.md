# Changelog

Todas as mudanças notáveis neste projeto serão documentadas neste arquivo.

O formato é baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/),
e este projeto adere ao [Semantic Versioning](https://semver.org/lang/pt-BR/).

---

## [1.5.0] - 2025-12-10

### Adicionado
- 🔌 **HTTP Streamable Protocol** - Suporte completo ao MCP 2024-11-05
- 🛠️ 3 novas tools de domínio: `domain_addon_conversion_status`, `domain_check_authority`, `domain_update_userdomains`
- 📊 **DNS Cache System** - Redução de 25k+ tokens para ~2k em zonas grandes
- 🔍 **Nested Domain Detector** - Detecção automática de subdomínios aninhados
- 📈 **Response Optimizer** - Paginação, compressão e estimativa de tokens
- 🧪 **651 testes** passando (100%) com 58.89% de cobertura

### Modificado
- ✅ Templates atualizados para HTTP Streamable (Claude Desktop, VS Code, Cursor, Windsurf, Zed)
- ✅ Endpoint padrão: `http://mcp.servidor.one:3200/mcp`
- ✅ Autenticação via header `x-api-key` (mais seguro que env vars)
- ✅ Porta padrão: 3200 (consistente em todos os templates)
- ✅ Total de 48 tools (incremento de 3 tools)

### Corrigido
- 🐛 Timeout em consultas DNS de zonas grandes (skillsit.com.br)
- 🐛 Memory leaks em suite de testes (setup.js global)
- 🐛 Inconsistência de portas entre templates (3100 vs 3200)

### Documentação
- 📝 README atualizado com 48 tools e HTTP protocol
- 📝 TESTING atualizado com curl examples HTTP
- 📝 Documentação técnica em `/docs` (MELHORIAS-DNS, IMPLEMENTATION, etc)

### Técnico
- 🏗️ Arquitetura DNS modular: `dns-constants/`, `dns-helpers/`
- 🧰 Bibliotecas de suporte: cache, validators, parsers, optimizers
- 🔐 Safety guard com confirmação em operações destrutivas
- 📊 Métricas: 48 tools, 1357 linhas no handler, 4 helpers DNS

---

## [1.4.0] - 2025-12-07

### Adicionado
- **Suite de Domínios (SPEC-NOVAS-FEATURES-WHM-001)**: 22 novas tools `domain.*`/`dns.*` cobrindo usuário/owner, alias, subdomínio, resolução, autoridade local, MX, DS, ALIAS, conversões de addon e manutenção `/etc/userdomains`.
- **Paginacao obrigatória** em `domain.get_all_info` (`limit/offset/filter`) com metadados `has_more/next_offset`.
- **DNSSEC/NSEC3 assíncrono**: `domain.enable_nsec3` e `domain.disable_nsec3` retornam `operation_id`; `domain.get_nsec3_status` faz polling com timeout dinâmico `60s + 30s * dom` (máx 600s).
- **Segurança reforçada**: validação de domínio (RS01), validação de `document_root` (RS03), SafetyGuard via header `X-MCP-Safety-Token` (body tem precedência) e ACL propagado (`X-MCP-ACL-Token`/`Authorization`) para root/reseller/user.
- **Idempotência**: `dns.add_mx` evita duplicatas; `domain.create_alias`/`domain.create_subdomain` e operações MX retornam flag `idempotent` quando já existem.
- **Lock + transaction log**: `domain.update_userdomains` usa `lock-manager` e `transaction-log` para rollback seguro; NSEC3 registra operações assíncronas.
- **Testes**: suites automatizadas para Fase 2/3 (MX idempotente, DS/ALIAS fallback, NSEC3 timeouts) e propagação de ACL token.

### Alterado
- **Timeouts alinhados ao RNF01**: limite absoluto 600s; `withTimeout` aplicado aos endpoints WHM sensíveis (DS/ALIAS) para evitar travamentos.
- **Contagem total de tools** atualizada para **45** (10 whm.*, 19 domain.*, 9 dns.*, 4 file/log/system).
- **Documentação**: README/TESTING revisados com novos exemplos de NSEC3, DS/ALIAS, paginacao e cabeçalhos de segurança; changelog anterior corrigido.
- **SafetyGuard**: suporte explícito a header, com redacão de tokens nos logs.

### Corrigido
- **DNSSEC/ALIAS**: chamadas agora retornam erro claro quando o endpoint não existe ou DNSSEC não está habilitado (em vez de timeout silencioso).
- **ACL**: validação agora usa o token da requisição (`X-MCP-ACL-Token`/`Authorization`), impedindo uso involuntário do fallback root.
- **MX duplicado**: `dns.add_mx` verifica registros existentes antes de criar.

---

## [1.0.0] - 2025-12-07

### Adicionado

#### Gerenciamento de Contas WHM
- **whm.list_accounts** - Listar todas as contas cPanel com filtros por domínio ou usuário
- **whm.create_account** - Criar nova conta cPanel com validação de parâmetros
- **whm.get_account_summary** - Obter informações detalhadas de uma conta
- **whm.suspend_account** - Suspender conta com auditoria de razão
- **whm.unsuspend_account** - Reativar conta suspensa
- **whm.delete_account** - Deletar conta (requer confirmationToken)
- **whm.change_package** - Alterar pacote de hospedagem de uma conta
- **whm.modify_account** - Modificar configurações de conta (quota, etc.)

#### Gerenciamento de DNS
- **dns.list_zones** - Listar todas as zonas DNS do servidor
- **dns.get_zone** - Obter registros DNS completos de uma zona
- **dns.add_record** - Adicionar registro DNS (A, AAAA, CNAME, MX, TXT, SRV, CAA)
- **dns.delete_record** - Deletar registro DNS com validação
- **dns.update_record** - Atualizar registro DNS existente
- **dns.validate_zone** - Validar sintaxe de zona DNS
- **dns.optimistic_lock** - Sistema de bloqueio otimista para prevenir race conditions

#### Monitoramento e Sistema
- **whm.server_status** - Status geral do servidor (uptime, load, memória, disco)
- **whm.service_status** - Status de serviços específicos (httpd, mysql, exim)
- **system.get_load** - Métricas detalhadas de carga e recursos
- **log.read_last_lines** - Ler últimas linhas de logs do sistema

#### Gerenciamento de Arquivos
- **file.list** - Listar arquivos e diretórios de uma conta
- **file.read** - Ler conteúdo de arquivo (com limite de segurança)
- **file.write** - Escrever conteúdo em arquivo
- **file.delete** - Deletar arquivo (requer confirmationToken)

#### Utilitários
- **util.run_command** - Executar comandos shell pré-aprovados (whitelisted)
- **util.restart_service** - Reiniciar serviços do sistema (requer confirmationToken)

### Segurança

#### Safety Guard System
- Confirmação obrigatória para operações destrutivas
- Whitelist de comandos shell permitidos
- Sanitização automática de credenciais em logs
- Validação de path para prevenir directory traversal
- Rate limiting em operações de massa

#### Autenticação e Autorização
- API Key authentication via WHMCS
- Bearer Token support
- Sanitização de logs (auto-redact de senhas e tokens)
- Validação de permissões por operação

### Monitoramento

#### Métricas Prometheus
- **http_requests_total** - Total de requisições HTTP por status e método
- **http_request_duration_seconds** - Duração de requisições HTTP (histograma)
- **mcp_tool_calls_total** - Total de chamadas de tools MCP por nome
- **mcp_tool_errors_total** - Total de erros em tools MCP
- Endpoint de scraping: `GET /metrics`

#### Logging Estruturado
- Winston logger com níveis configuráveis
- Logs rotacionados automaticamente
- Formato JSON para integração com ELK/Grafana
- Sanitização automática de credenciais

### CLI Ferramentas

#### Comandos Disponíveis
- `skills-whm-mcp introspect` - Introspecção de tools MCP (formato JSON)
- `skills-whm-mcp describe-tools` - Descrição detalhada de todos os tools
- Suporte a output JSON e XML

### Configuração

#### Variáveis de Ambiente
- `WHM_API_URL` - URL da API WHM (obrigatório)
- `WHM_API_TOKEN` - Token de autenticação WHM (obrigatório)
- `MCP_PORT` - Porta do servidor MCP (padrão: 3100)
- `LOG_LEVEL` - Nível de logging (debug|info|warn|error)
- `ENABLE_METRICS` - Habilitar métricas Prometheus (true|false)

### Documentação

#### Arquivos de Documentação
- **README.md** - Guia completo de instalação e uso (682 linhas)
- **CONTRIBUTING.md** - Guia para contribuidores
- **CODE_OF_CONDUCT.md** - Código de conduta
- **TESTING.md** - Procedimentos de teste e validação
- **schemas/mcp-tools.json** - Schema completo de todos os tools
- **schemas/examples.json** - 32+ exemplos de uso real
- **schemas/whm-api-reference.json** - Referência de APIs WHM utilizadas

#### Templates de Integração
- **Visual Studio Code** - Configuração MCP para VS Code
- **Windsurf** - Configuração MCP para Windsurf IDE
- **Claude Desktop** - Configuração MCP para Claude Desktop App
- **JetBrains IDEs** - Configuração XML para IntelliJ, PyCharm, etc.
- **Cursor** - Configuração MCP para Cursor AI IDE
- **Zed** - Configuração MCP para Zed Editor
- **Continue.dev** - Configuração MCP para Continue extension

### Testes

#### Cobertura de Testes
- Testes unitários para serviços WHM
- Testes de integração para tools MCP
- Cobertura mínima de 25% (branches, functions, lines, statements)
- CI/CD com Jest e relatórios de cobertura

### Dependências

#### Produção
- **@modelcontextprotocol/sdk** ^0.5.0 - SDK oficial MCP
- **express** ^4.18.0 - Framework HTTP
- **ssh2** ^1.14.0 - Cliente SSH para operações remotas
- **axios** ^1.6.0 - Cliente HTTP para WHM API
- **dotenv** ^16.3.0 - Gerenciamento de variáveis de ambiente
- **winston** ^3.11.0 - Logging estruturado
- **prom-client** ^15.1.0 - Métricas Prometheus
- **zod** ^3.22.0 - Validação de schemas

#### Desenvolvimento
- **jest** ^29.7.0 - Framework de testes
- **supertest** ^6.3.0 - Testes HTTP
- **nodemon** ^3.0.0 - Auto-reload para desenvolvimento

### Infraestrutura

#### Deployment
- Gerenciamento via PM2
- Logs centralizados em `/opt/mcp-servers/_shared/logs/`
- Suporte a múltiplas instâncias
- Health checks via `GET /health`

---

## Links

- [Documentação Completa](https://github.com/DevSkillsIT/skills-mcp-whm-pro#readme)
- [Issues e Bug Reports](https://github.com/DevSkillsIT/skills-mcp-whm-pro/issues)
- [Guia de Contribuição](https://github.com/DevSkillsIT/skills-mcp-whm-pro/blob/main/CONTRIBUTING.md)

---

**Skills IT - Soluções em Tecnologia**  
contato@skillsit.com.br  
https://www.skillsit.com.br
