# 🚀 Guia Rápido de Deploy

## Qual opção escolher?

### ✅ Use `docker-compose.yml` (Traefik novo)
- Se você **NÃO tem** Traefik rodando
- Se você **NÃO tem** Nginx usando portas 80/443
- Se você quer SSL automático sem configuração manual

### ✅ Use `docker-compose.traefik-existente.yml`
- Se você **JÁ TEM** Traefik rodando
- Se você quer adicionar esta aplicação ao Traefik existente
- Copie para `docker-compose.yml` e ajuste o nome da rede

### ✅ Use `docker-compose.nginx.yml`
- Se você **JÁ TEM** Nginx rodando
- Se você prefere gerenciar SSL manualmente com Certbot
- Configure Nginx usando `nginx-vidracaria.conf`

### ✅ Use PM2 (sem Docker)
- Se você não quer usar Docker
- Se você prefere controle total sobre o processo Node.js
- Veja seção "Opção 2" no DEPLOY.md

## ⚡ Comandos Rápidos

### Verificar conflitos ANTES de deploy:
```bash
# Verificar Traefik
docker ps | grep traefik

# Verificar portas
sudo netstat -tulpn | grep -E ':(80|443)'

# Verificar redes Docker
docker network ls
```

### Deploy rápido (Traefik novo):
```bash
# 1. Editar docker-compose.yml (domínio e email)
# 2. Criar diretórios
mkdir -p data/uploads data/client-uploads letsencrypt

# 3. Iniciar
docker-compose up -d --build
```

### Deploy rápido (Traefik existente):
```bash
# 1. Identificar rede do Traefik
docker network ls

# 2. Copiar e editar
cp docker-compose.traefik-existente.yml docker-compose.yml
nano docker-compose.yml  # Ajustar nome da rede e domínio

# 3. Iniciar
docker-compose up -d --build
```

### Deploy rápido (Nginx):
```bash
# 1. Copiar e editar
cp docker-compose.nginx.yml docker-compose.yml
docker-compose up -d --build

# 2. Configurar Nginx
sudo cp nginx-vidracaria.conf /etc/nginx/sites-available/vidracaria-parana
sudo nano /etc/nginx/sites-available/vidracaria-parana  # Editar domínio
sudo ln -s /etc/nginx/sites-available/vidracaria-parana /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl restart nginx

# 3. SSL
sudo certbot --nginx -d seu-dominio.com
```

## 📝 Checklist Antes de Deploy

- [ ] Domínio apontando para IP do VPS
- [ ] Portas 80/443 disponíveis (ou usar Traefik/Nginx existente)
- [ ] Docker instalado (se usar Docker)
- [ ] Arquivo docker-compose.yml configurado com domínio correto
- [ ] Diretórios de dados criados
- [ ] Firewall configurado (portas 80, 443, 22 abertas)

## 🔍 Troubleshooting Rápido

**Porta já em uso?**
→ Use `docker-compose.traefik-existente.yml` ou `docker-compose.nginx.yml`

**Traefik não encontra a aplicação?**
→ Verifique nome da rede: `docker network inspect traefik-network`

**SSL não funciona?**
→ Verifique se domínio aponta para o VPS: `dig seu-dominio.com`

**Aplicação não inicia?**
→ Ver logs: `docker-compose logs -f app`






