# Guia de Deploy - Vidraçaria Paraná

Este guia explica como fazer o deploy da aplicação em um VPS usando Traefik ou diretamente com Node.js.

## 📋 Pré-requisitos

- VPS com Ubuntu/Debian (ou similar)
- Domínio apontando para o IP do VPS
- Acesso SSH ao servidor
- Docker e Docker Compose instalados (para opção Traefik)

## ⚠️ IMPORTANTE: Verificar Conflitos

**Antes de começar**, verifique se você já tem Traefik ou outro reverse proxy rodando:

```bash
# Verificar se Traefik está rodando
docker ps | grep traefik

# Verificar portas 80 e 443 em uso
sudo netstat -tulpn | grep -E ':(80|443)'

# Verificar redes Docker existentes
docker network ls
```

**Se você já tem Traefik rodando**, use o arquivo `docker-compose.traefik-existente.yml` (veja abaixo).

**Se você já tem Nginx rodando**, use o arquivo `docker-compose.nginx.yml` (veja abaixo).

## 🚀 Opção 1A: Deploy com Traefik NOVO (Sem conflitos)

**Use esta opção apenas se você NÃO tem Traefik rodando no VPS.**

Traefik é um reverse proxy moderno que gerencia SSL automaticamente com Let's Encrypt.

⚠️ **ATENÇÃO**: Esta configuração cria um NOVO Traefik. Se você já tem Traefik, use a **Opção 1B** abaixo.

### Passo 1: Preparar o Servidor

```bash
# Atualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Instalar Docker Compose
sudo apt install docker-compose-plugin -y

# Adicionar usuário ao grupo docker (opcional, para não usar sudo)
sudo usermod -aG docker $USER
newgrp docker
```

### Passo 2: Clonar/Copiar o Projeto

```bash
# Clonar repositório ou fazer upload dos arquivos
git clone <seu-repositorio> vidracaria-parana
cd vidracaria-parana

# Ou usar SCP para copiar arquivos do seu computador local
# scp -r . usuario@seu-vps:/home/usuario/vidracaria-parana
```

### Passo 3: Configurar Docker Compose

Edite o arquivo `docker-compose.yml`:

```yaml
# Substitua 'seu-dominio.com' pelo seu domínio real
- "traefik.http.routers.vidracaria.rule=Host(`seu-dominio.com`)"
- "traefik.http.routers.vidracaria-http.rule=Host(`seu-dominio.com`)"

# Substitua 'seu-email@exemplo.com' pelo seu email
- "--certificatesresolvers.letsencrypt.acme.email=seu-email@exemplo.com"
```

### Passo 4: Criar Diretórios de Dados

```bash
mkdir -p data/uploads data/client-uploads letsencrypt
chmod 600 letsencrypt  # Segurança para certificados SSL
```

### Passo 5: Inicializar o Banco de Dados (Opcional)

Se você já tem um banco de dados com dados, copie-o:

```bash
# Copiar banco existente
cp sqlite.db data/sqlite.db
```

### Passo 6: Construir e Iniciar

```bash
# Construir e iniciar containers
docker-compose up -d --build

# Ver logs
docker-compose logs -f

# Ver status
docker-compose ps
```

### Passo 7: Criar Usuário Admin

```bash
# Entrar no container
docker-compose exec app sh

# Executar script de criação de admin
node dist/scripts/create-admin.js
# Ou se precisar instalar dependências no container:
# pnpm install
# pnpm exec tsx server/scripts/create-admin.ts
```

### Passo 8: Verificar

- Acesse `https://seu-dominio.com` (HTTPS será automático)
- Dashboard do Traefik: `http://seu-vps-ip:8080` (apenas localmente)

### Comandos Úteis

```bash
# Parar serviços
docker-compose down

# Parar e remover volumes (CUIDADO: apaga dados)
docker-compose down -v

# Ver logs
docker-compose logs -f app

# Reiniciar após mudanças
docker-compose restart app

# Atualizar aplicação
git pull
docker-compose up -d --build
```

⚠️ **Conflito de portas?** Se as portas 80/443 já estiverem em uso, você precisa:
- Parar o serviço que está usando essas portas, OU
- Usar a Opção 1B (Traefik existente) ou Opção 1C (Nginx)

---

## 🚀 Opção 1B: Deploy com Traefik EXISTENTE (Recomendado se já tem Traefik)

**Use esta opção se você JÁ TEM Traefik rodando no VPS.**

### Passo 1: Identificar a Rede do Traefik

```bash
# Ver redes Docker
docker network ls

# Ver detalhes do container Traefik
docker inspect traefik | grep -A 5 Networks

# Ou verificar o docker-compose do Traefik existente
# Geralmente está em: /opt/traefik/docker-compose.yml ou similar
```

Anote o nome da rede (geralmente `traefik-network`, `traefik_default`, ou similar).

### Passo 2: Configurar docker-compose

```bash
# Copiar arquivo de configuração
cp docker-compose.traefik-existente.yml docker-compose.yml

# Editar docker-compose.yml
nano docker-compose.yml
```

**Ajustar:**
1. Substitua `traefik-network` pelo nome da sua rede Traefik (linha 32)
2. Substitua `seu-dominio.com` pelo seu domínio (linhas 23 e 29)
3. Verifique se os entrypoints estão corretos (geralmente `web` e `websecure`)

### Passo 3: Criar Diretórios

```bash
mkdir -p data/uploads data/client-uploads
```

### Passo 4: Iniciar Aplicação

```bash
# Construir e iniciar (sem o serviço Traefik)
docker-compose up -d --build

# Ver logs
docker-compose logs -f app
```

### Passo 5: Verificar no Traefik

Acesse o dashboard do Traefik (geralmente em `http://seu-vps-ip:8080`) e verifique se a aplicação aparece na lista de serviços.

---

## 🚀 Opção 1C: Deploy com Nginx (Sem Traefik)

**Use esta opção se você prefere Nginx ou já tem Nginx rodando.**

### Passo 1: Preparar Docker Compose

```bash
# Usar arquivo sem Traefik
cp docker-compose.nginx.yml docker-compose.yml

# Construir e iniciar aplicação
docker-compose up -d --build
```

### Passo 2: Configurar Nginx

```bash
# Copiar configuração de exemplo
sudo cp nginx-vidracaria.conf /etc/nginx/sites-available/vidracaria-parana

# Editar configuração
sudo nano /etc/nginx/sites-available/vidracaria-parana
```

**Substitua:**
- `seu-dominio.com` pelo seu domínio real

```bash
# Ativar site
sudo ln -s /etc/nginx/sites-available/vidracaria-parana /etc/nginx/sites-enabled/

# Testar configuração
sudo nginx -t

# Reiniciar Nginx
sudo systemctl restart nginx
```

### Passo 3: Configurar SSL

```bash
# Instalar Certbot
sudo apt install certbot python3-certbot-nginx -y

# Obter certificado SSL
sudo certbot --nginx -d seu-dominio.com -d www.seu-dominio.com

# Certbot atualizará automaticamente o arquivo do Nginx
```

### Passo 4: Verificar

Acesse `https://seu-dominio.com`

**Nota**: A aplicação Docker roda na porta 5000 internamente. O Nginx no host faz o proxy para o container.

---

## 🔧 Opção 2: Deploy Direto com Node.js e PM2

Esta opção não usa Docker, apenas Node.js e PM2 para gerenciar o processo.

### Passo 1: Preparar o Servidor

```bash
# Atualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Instalar pnpm
npm install -g pnpm

# Instalar PM2 (gerenciador de processos)
npm install -g pm2
```

### Passo 2: Clonar/Copiar o Projeto

```bash
cd /home/usuario
git clone <seu-repositorio> vidracaria-parana
cd vidracaria-parana
```

### Passo 3: Instalar Dependências e Build

```bash
# Instalar dependências
pnpm install --frozen-lockfile

# Build da aplicação
pnpm build

# Criar diretório de dados
mkdir -p data/uploads
```

### Passo 4: Configurar Variáveis de Ambiente

```bash
# Criar arquivo .env (se necessário)
nano .env
```

Adicione se necessário:
```
NODE_ENV=production
PORT=5000
```

### Passo 5: Iniciar com PM2

```bash
# Criar arquivo de configuração PM2
nano ecosystem.config.js
```

Conteúdo do `ecosystem.config.js`:

```javascript
module.exports = {
  apps: [{
    name: 'vidracaria-parana',
    script: 'dist/index.js',
    instances: 1,
    exec_mode: 'fork',
    env: {
      NODE_ENV: 'production',
      PORT: 5000
    },
    error_file: './logs/pm2-error.log',
    out_file: './logs/pm2-out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
    merge_logs: true,
    autorestart: true,
    watch: false,
    max_memory_restart: '500M'
  }]
};
```

```bash
# Criar diretório de logs
mkdir -p logs

# Iniciar aplicação
pm2 start ecosystem.config.js

# Salvar configuração PM2 para iniciar no boot
pm2 save
pm2 startup
```

### Passo 6: Configurar Nginx como Reverse Proxy

```bash
# Instalar Nginx
sudo apt install nginx -y

# Criar configuração
sudo nano /etc/nginx/sites-available/vidracaria-parana
```

Conteúdo da configuração:

```nginx
server {
    listen 80;
    server_name seu-dominio.com www.seu-dominio.com;

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Ativar site
sudo ln -s /etc/nginx/sites-available/vidracaria-parana /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### Passo 7: Configurar SSL com Certbot

```bash
# Instalar Certbot
sudo apt install certbot python3-certbot-nginx -y

# Obter certificado SSL
sudo certbot --nginx -d seu-dominio.com -d www.seu-dominio.com

# Renovação automática (já configurado automaticamente)
sudo certbot renew --dry-run
```

### Passo 8: Verificar

- Acesse `https://seu-dominio.com`

### Comandos Úteis PM2

```bash
# Ver status
pm2 status

# Ver logs
pm2 logs vidracaria-parana

# Reiniciar
pm2 restart vidracaria-parana

# Parar
pm2 stop vidracaria-parana

# Monitorar
pm2 monit
```

---

## 🔄 Atualizar Aplicação

### Com Traefik (Docker - novo):

```bash
cd /caminho/para/vidracaria-parana
git pull
docker-compose up -d --build
```

### Com Traefik (Docker - existente):

```bash
cd /caminho/para/vidracaria-parana
git pull
docker-compose up -d --build app
# Não precisa rebuild do Traefik, apenas da app
```

### Com Nginx (Docker):

```bash
cd /caminho/para/vidracaria-parana
git pull
docker-compose up -d --build
```

### Com PM2:

```bash
cd /home/usuario/vidracaria-parana
git pull
pnpm install --frozen-lockfile
pnpm build
pm2 restart vidracaria-parana
```

---

## 📦 Backup

**IMPORTANTE:** Sempre faça backup do banco de dados e uploads!

```bash
# Backup manual
tar -czf backup-$(date +%Y%m%d).tar.gz data/sqlite.db data/uploads

# Com Traefik (do host)
docker-compose exec app tar -czf /tmp/backup.tar.gz /app/sqlite.db /app/uploads
docker-compose cp app:/tmp/backup.tar.gz ./backup-$(date +%Y%m%d).tar.gz
```

---

## 🐛 Troubleshooting

### Problemas com Conflito de Portas

**Porta 80/443 já em uso:**
```bash
# Ver o que está usando a porta
sudo lsof -i :80
sudo lsof -i :443

# Se for Nginx, você pode:
# 1. Parar Nginx: sudo systemctl stop nginx
# 2. Ou usar docker-compose.nginx.yml e configurar Nginx para fazer proxy
```

**Traefik já existe:**
```bash
# Ver containers Traefik
docker ps | grep traefik

# Use docker-compose.traefik-existente.yml em vez de docker-compose.yml
```

### Problemas com SSL/Traefik

- Verifique se o domínio está apontando para o IP do VPS
- Verifique logs: `docker-compose logs traefik` (se usando Traefik novo)
- Verifique logs do Traefik existente: `docker logs traefik`
- Certifique-se de que as portas 80 e 443 estão abertas no firewall
- Verifique se a rede Docker está correta: `docker network inspect traefik-network`

### Problemas com Rede Docker

**Container não consegue se conectar ao Traefik:**
```bash
# Verificar se está na rede correta
docker network inspect traefik-network

# Adicionar container à rede manualmente
docker network connect traefik-network vidracaria-parana

# Verificar labels do Traefik
docker inspect vidracaria-parana | grep -A 20 Labels
```

### Problemas com Build

- Verifique se todas as dependências estão instaladas
- Para `better-sqlite3`, pode ser necessário compilar no mesmo ambiente
- Limpe build anterior: `docker-compose down && docker-compose build --no-cache`

### Problemas com Permissões

```bash
# Ajustar permissões de dados
sudo chown -R $USER:$USER data/
chmod -R 755 data/
```

### Aplicação não aparece no Traefik

```bash
# Verificar se Traefik está vendo o container
docker exec traefik cat /var/run/docker.sock

# Verificar logs do Traefik
docker logs traefik

# Verificar se labels estão corretos
docker inspect vidracaria-parana | grep traefik
```

---

## 📝 Notas Importantes

1. **Banco de Dados**: O SQLite é persistido em `data/sqlite.db`. Certifique-se de fazer backups regulares.
2. **Uploads**: Os arquivos enviados ficam em `data/uploads`. Mantenha espaço suficiente no disco.
3. **Segurança**: 
   - Mantenha o sistema atualizado
   - Use senhas fortes
   - Configure firewall (UFW)
   - Não exponha portas desnecessárias
4. **Performance**: Para produção com muito tráfego, considere migrar para PostgreSQL ou MySQL.

---

## 🔐 Configurar Firewall

```bash
# Instalar UFW
sudo apt install ufw -y

# Permitir SSH (IMPORTANTE: antes de ativar!)
sudo ufw allow 22/tcp

# Permitir HTTP e HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Ativar firewall
sudo ufw enable

# Ver status
sudo ufw status
```

---

## 📞 Suporte

Em caso de problemas, verifique:
- Logs da aplicação: `docker-compose logs` ou `pm2 logs`
- Logs do servidor web: `/var/log/nginx/error.log` ou `docker-compose logs traefik`
- Status dos serviços: `docker-compose ps` ou `pm2 status`

