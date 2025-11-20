# 🔐 Painel Administrativo - Vidraçaria Paraná

## 📋 Visão Geral

O painel administrativo permite gerenciar todo o conteúdo do site de forma visual e intuitiva, sem necessidade de mexer no código.

---

## 🚀 Acesso ao Painel

### URL de Acesso
```
http://localhost:3000/admin/login
```

### Credenciais Iniciais
- **E-mail:** `admin@vidracariaparana.com.br`
- **Senha:** `admin123`

⚠️ **IMPORTANTE:** Altere a senha após o primeiro login!

---

## 📱 Funcionalidades Implementadas

### ✅ 1. Sistema de Autenticação
- Login seguro com JWT
- Cookies httpOnly para segurança
- Sessão de 7 dias
- Logout seguro

### ✅ 2. Dashboard
- Estatísticas em tempo real
- Contadores de categorias, produtos e imagens
- Ações rápidas para tarefas comuns
- Interface responsiva

### ✅ 3. Gerenciamento de Galeria
- **Upload de imagens**
  - Múltiplas imagens simultaneamente
  - Drag & drop (arrastar e soltar)
  - Limite de 5MB por imagem
  - Formatos: JPEG, PNG, WebP

- **Organização**
  - Selecionar produto
  - Visualizar todas as imagens
  - Deletar imagens
  - Editar títulos e descrições (em breve)

### ✅ 4. APIs REST Completas
- `/api/auth/*` - Autenticação
- `/api/categories/*` - Categorias
- `/api/products/*` - Produtos
- `/api/upload/*` - Upload de imagens

---

## 👥 Níveis de Acesso

### 🔴 Administrador
- Acesso total ao sistema
- Criar, editar e deletar tudo
- Gerenciar usuários (em breve)
- Configurações do site

### 🟡 Vendedor
- Adicionar produtos
- Fazer upload de imagens
- Editar produtos existentes
- **NÃO pode deletar** categorias ou produtos principais

---

## 📂 Estrutura do Banco de Dados

### Tabelas Criadas

#### `users` - Usuários do Sistema
- id, email, password (hash bcrypt)
- name, role (admin/vendedor)
- active, createdAt, lastLogin

#### `categories` - Categorias de Produtos
- id, name, slug
- description, order
- createdAt

#### `subcategories` - Subcategorias (Linhas)
- id, categoryId, name, slug
- description, order
- createdAt

#### `products` - Produtos
- id, categoryId, subcategoryId
- title, slug, description
- specs (JSON), coverImage
- published, featured, order
- createdAt, updatedAt

#### `gallery_images` - Imagens da Galeria
- id, productId, imageUrl
- title, description, alt
- order, width, height, fileSize
- createdAt

---

## 🛠️ Como Usar

### 1. Adicionar Imagens a um Produto

1. Acesse `/admin/gallery`
2. Selecione o produto no dropdown
3. Clique em "Selecionar imagens" ou arraste arquivos
4. As imagens serão enviadas automaticamente
5. Visualize e gerencie as imagens adicionadas

### 2. Criar Nova Categoria (via API)

```bash
curl -X POST http://localhost:3000/api/categories \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Esquadrias",
    "slug": "esquadrias",
    "description": "Esquadrias de alumínio",
    "order": 1
  }'
```

### 3. Criar Novo Produto (via API)

```bash
curl -X POST http://localhost:3000/api/products \
  -H "Content-Type: application/json" \
  -d '{
    "categoryId": 1,
    "title": "Porta de Correr 4 Folhas",
    "slug": "porta-correr-4-folhas",
    "description": "Porta de correr em alumínio linha Gold",
    "specs": {
      "linha": "Gold",
      "tipo": "Correr",
      "folhas": 4,
      "acabamento": "Preto"
    }
  }'
```

---

## 🔒 Segurança

### Implementado
✅ Senhas com hash bcrypt
✅ JWT com expiração de 7 dias
✅ Cookies httpOnly (proteção XSS)
✅ Validação de tipos de arquivo
✅ Limite de tamanho de upload
✅ Middleware de autenticação
✅ Controle de permissões (admin/vendedor)

### Recomendações
⚠️ Alterar senha padrão
⚠️ Usar HTTPS em produção
⚠️ Configurar variável JWT_SECRET
⚠️ Fazer backup regular do banco

---

## 📝 Próximas Funcionalidades

### Em Desenvolvimento
🔄 Gerenciamento de categorias (UI)
🔄 Gerenciamento de produtos (UI)
🔄 Edição de textos e descrições
🔄 Reordenar imagens (drag & drop)
🔄 Gerenciamento de usuários
🔄 Alterar senha do perfil
🔄 Logs de atividades
🔄 Estatísticas de visualizações

---

## 🐛 Troubleshooting

### Erro ao fazer login
- Verifique se o banco de dados foi inicializado
- Execute: `npx tsx server/scripts/create-admin.ts`

### Erro ao fazer upload
- Verifique se a pasta `uploads/` existe
- Verifique permissões de escrita
- Confirme que a imagem é menor que 5MB

### Sessão expira muito rápido
- Ajuste o tempo de expiração em `server/middleware/auth.ts`
- Padrão: 7 dias

---

## 📞 Suporte

Para dúvidas ou problemas:
- 📧 E-mail: contato@vidracariaparanasp.com.br
- 📱 WhatsApp: (11) XXXXX-XXXX

---

**Desenvolvido para Vidraçaria Paraná** 🏗️

