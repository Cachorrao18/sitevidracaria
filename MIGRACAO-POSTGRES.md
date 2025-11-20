# 🐘 Migração para PostgreSQL

A aplicação foi migrada de SQLite para PostgreSQL. Siga estes passos:

## 📋 Passos para Migração

### 1. Instalar Dependências

```bash
pnpm install
```

### 2. Gerar Migrações

```bash
pnpm db:generate
```

Isso criará os arquivos de migração na pasta `drizzle/`.

### 3. Executar Migrações

**Localmente (desenvolvimento):**
```bash
# Configurar DATABASE_URL no .env ou variável de ambiente
export DATABASE_URL="postgresql://postgres:postgres@localhost:5432/vidracaria_parana"

# Executar migrações
pnpm db:migrate
```

**Com Docker:**
```bash
# Após subir os containers, executar migrações dentro do container
docker-compose exec app pnpm db:migrate
```

### 4. Migrar Dados do SQLite (se necessário)

Se você tem dados no SQLite que precisa migrar:

```bash
# Exportar dados do SQLite
sqlite3 sqlite.db .dump > backup.sql

# Converter e importar manualmente para PostgreSQL
# Ou usar ferramenta de migração
```

## 🔧 Configuração

### Variáveis de Ambiente

```env
DATABASE_URL=postgresql://usuario:senha@host:5432/database
# ou
POSTGRES_URL=postgresql://usuario:senha@host:5432/database
```

### Docker Compose

O PostgreSQL já está configurado nos arquivos `docker-compose.yml`:
- Banco: `vidracaria_parana`
- Usuário: `postgres`
- Senha: `postgres` (⚠️ **ALTERE EM PRODUÇÃO!**)

## 🔐 Segurança em Produção

**IMPORTANTE**: Altere as credenciais padrão do PostgreSQL:

1. Edite `docker-compose.yml`:
```yaml
environment:
  POSTGRES_USER: seu_usuario
  POSTGRES_PASSWORD: sua_senha_forte
  POSTGRES_DB: vidracaria_parana
```

2. Atualize `DATABASE_URL` na aplicação:
```env
DATABASE_URL=postgresql://seu_usuario:sua_senha_forte@postgres:5432/vidracaria_parana
```

## 📊 Comandos Úteis

```bash
# Gerar migrações a partir do schema
pnpm db:generate

# Executar migrações
pnpm db:migrate

# Aplicar mudanças direto (sem migrações) - apenas dev
pnpm db:push

# Abrir Drizzle Studio (interface visual)
pnpm db:studio
```

## 🐛 Troubleshooting

### Erro de conexão
- Verifique se PostgreSQL está rodando
- Verifique se `DATABASE_URL` está correto
- Verifique firewall/portas

### Erro de migração
- Certifique-se de que as migrações foram geradas: `pnpm db:generate`
- Verifique permissões do banco de dados
- Verifique se o banco existe

### Reset completo (CUIDADO!)
```bash
# Deletar volume do PostgreSQL
docker-compose down -v

# Recriar
docker-compose up -d
pnpm db:migrate
```






