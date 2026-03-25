# 🍕 AGT Delivery — Backend API

> API REST para aplicativo de delivery **Pedix**, desenvolvida com Node.js + Express + PostgreSQL.

![Node.js](https://img.shields.io/badge/Node.js-18%2B-green?logo=node.js)
![Express](https://img.shields.io/badge/Express-5.x-lightgrey?logo=express)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15%2B-blue?logo=postgresql)
![Render](https://img.shields.io/badge/Deploy-Render-46E3B7?logo=render)
![License](https://img.shields.io/badge/License-ISC-yellow)

---

## 📋 Índice

- [Sobre o Projeto](#sobre-o-projeto)
- [Arquitetura](#arquitetura)
- [Funcionalidades](#funcionalidades)
- [Pré-requisitos](#pré-requisitos)
- [Instalação Local](#instalação-local)
- [Variáveis de Ambiente](#variáveis-de-ambiente)
- [Banco de Dados](#banco-de-dados)
- [Endpoints da API](#endpoints-da-api)
- [Deploy no Render](#deploy-no-render)
- [Segurança](#segurança)

---

## 📌 Sobre o Projeto

Backend do aplicativo **Pedix**, um sistema de pedidos de delivery que conecta clientes a restaurantes/empresas locais. A API gerencia usuários, empresas, cardápios, pedidos e favoritos, com autenticação via JWT.

---

## 🏗️ Arquitetura

O projeto segue o padrão de camadas **Controller → Service → Repository**:

```
src/
├── controllers/       # Recebe requisições HTTP e envia respostas
│   ├── controller.banner.js
│   ├── controller.categoria.js
│   ├── controller.empresa.js
│   ├── controller.pedido.js
│   └── controller.usuario.js
│
├── services/          # Regras de negócio
│   ├── service.banner.js
│   ├── service.categoria.js
│   ├── service.empresa.js
│   ├── service.pedido.js
│   └── service.usuario.js
│
├── repositories/      # Acesso ao banco de dados
│   ├── repository.banner.js
│   ├── repository.categoria.js
│   ├── repository.empresa.js
│   ├── repository.pedido.js
│   └── repository.usuario.js
│
├── database/
│   └── postgres.js    # Conexão com PostgreSQL (via pg)
│
├── token.js           # Geração e validação de JWT
├── routes.js          # Definição de rotas
└── index.js           # Entry point da aplicação
```

---

## ✅ Funcionalidades

- **Autenticação** — Login e registro de usuários com senha criptografada (bcrypt) e token JWT
- **Empresas** — Listagem, destaques, busca por nome/categoria/banner e cardápio completo
- **Favoritos** — Adicionar e remover empresas favoritas por usuário
- **Pedidos** — Criar pedido com múltiplos itens, listar e atualizar status
- **Perfil** — Visualizar e atualizar endereço do usuário logado
- **Banners e Categorias** — Listagem de conteúdo promocional

---

## 🛠️ Pré-requisitos

- [Node.js](https://nodejs.org/) v18 ou superior
- [PostgreSQL](https://www.postgresql.org/) v14 ou superior
- npm v9+

---

## 🚀 Instalação Local

```bash
# 1. Clone o repositório
git clone https://github.com/GutoAgt/Pw-AGTdelivery-backend.git
cd Pw-AGTdelivery-backend

# 2. Instale as dependências
npm install

# 3. Crie o arquivo .env na raiz do projeto
cp .env.example .env
# Edite o .env com suas configurações

# 4. Execute as migrations (crie as tabelas no PostgreSQL)
psql -U seu_usuario -d seu_banco -f src/database/schema.sql

# 5. Inicie o servidor
npm start
```

O servidor estará disponível em `http://localhost:3001`.

---

## 🔑 Variáveis de Ambiente

Crie um arquivo `.env` na raiz do projeto com as seguintes variáveis:

```env
# Porta do servidor
PORT=3001

# Segredo para geração de tokens JWT (use uma string longa e aleatória)
JWT_SECRET=sua_chave_secreta_muito_segura_aqui

# String de conexão com o PostgreSQL
DATABASE_URL=postgresql://usuario:senha@host:5432/nome_do_banco

# Ambiente
NODE_ENV=production
```

> ⚠️ **Nunca** faça commit do arquivo `.env`. Ele já está no `.gitignore`.

---

## 🗄️ Banco de Dados

### Schema principal (PostgreSQL)

```sql
-- Categorias de empresas
CREATE TABLE categoria (
  id_categoria SERIAL PRIMARY KEY,
  categoria VARCHAR(50),
  icone VARCHAR(1000),
  ordem INTEGER
);

-- Empresas / Restaurantes
CREATE TABLE empresa (
  id_empresa SERIAL PRIMARY KEY,
  nome VARCHAR(200),
  foto VARCHAR(1000),
  icone VARCHAR(1000),
  id_categoria INTEGER REFERENCES categoria(id_categoria),
  vl_taxa_entrega DECIMAL(12,2),
  endereco VARCHAR(100),
  complemento VARCHAR(50),
  bairro VARCHAR(50),
  cidade VARCHAR(50),
  uf VARCHAR(2),
  cep VARCHAR(10),
  hora_abertura TEXT DEFAULT '08:00',
  hora_fechamento TEXT DEFAULT '22:00',
  cpf_cnpj VARCHAR(30),
  status_funcionamento TEXT
);

-- Usuários
CREATE TABLE usuario (
  id_usuario SERIAL PRIMARY KEY,
  nome VARCHAR(100),
  email VARCHAR(100) UNIQUE,
  senha VARCHAR(200),
  endereco TEXT,
  complemento VARCHAR(50),
  bairro VARCHAR(50),
  cidade VARCHAR(50),
  uf VARCHAR(2),
  cep VARCHAR(10),
  dt_cadastro TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  tipo_usuario TEXT DEFAULT 'cliente'
);

-- Status possíveis de um pedido
CREATE TABLE pedido_status (
  status CHAR(1) PRIMARY KEY,
  descricao VARCHAR(50),
  cor VARCHAR(20)
);

-- Pedidos
CREATE TABLE pedido (
  id_pedido SERIAL PRIMARY KEY,
  id_usuario INTEGER REFERENCES usuario(id_usuario),
  id_empresa INTEGER REFERENCES empresa(id_empresa),
  vl_subtotal DECIMAL(12,2),
  vl_taxa_entrega DECIMAL(12,2),
  vl_total DECIMAL(12,2),
  dt_pedido TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  status CHAR(1) REFERENCES pedido_status(status),
  status_pagamento TEXT
);

-- Itens de um pedido
CREATE TABLE pedido_item (
  id_item SERIAL PRIMARY KEY,
  id_pedido INTEGER REFERENCES pedido(id_pedido),
  id_produto INTEGER REFERENCES produto(id_produto),
  obs VARCHAR(500),
  qtd INTEGER,
  vl_unitario DECIMAL(12,2),
  vl_total DECIMAL(12,2)
);

-- Categorias de produtos dentro de uma empresa
CREATE TABLE produto_categoria (
  id_categoria SERIAL PRIMARY KEY,
  id_empresa INTEGER REFERENCES empresa(id_empresa),
  categoria VARCHAR(100),
  ordem INTEGER
);

-- Produtos
CREATE TABLE produto (
  id_produto SERIAL PRIMARY KEY,
  id_empresa INTEGER REFERENCES empresa(id_empresa),
  id_categoria INTEGER REFERENCES produto_categoria(id_categoria),
  nome VARCHAR(100),
  descricao VARCHAR(500),
  icone VARCHAR(1000),
  vl_produto DECIMAL(12,2),
  ind_ativo CHAR(1)
);

-- Banners promocionais
CREATE TABLE banner (
  id_banner SERIAL PRIMARY KEY,
  icone VARCHAR(1000),
  id_empresa INTEGER REFERENCES empresa(id_empresa),
  ordem INTEGER
);

-- Destaques da home
CREATE TABLE destaque (
  id_destaque SERIAL PRIMARY KEY,
  id_empresa INTEGER REFERENCES empresa(id_empresa),
  ordem INTEGER
);

-- Favoritos por usuário
CREATE TABLE usuario_favorito (
  id_favorito SERIAL PRIMARY KEY,
  id_usuario INTEGER REFERENCES usuario(id_usuario),
  id_empresa INTEGER REFERENCES empresa(id_empresa)
);

-- Dados iniciais de status
INSERT INTO pedido_status (status, descricao, cor) VALUES
  ('P', 'Pendente',    '#FFA500'),
  ('A', 'Aceito',      '#2196F3'),
  ('E', 'Em entrega',  '#9C27B0'),
  ('F', 'Finalizado',  '#4CAF50'),
  ('C', 'Cancelado',   '#F44336');
```

---

## 📡 Endpoints da API

Todas as rotas (exceto login e registro) requerem o header:
```
Authorization: Bearer <token_jwt>
```

### Autenticação
| Método | Rota | Descrição |
|--------|------|-----------|
| `POST` | `/usuarios` | Registrar novo usuário |
| `POST` | `/usuarios/login` | Login (retorna token JWT) |

### Usuários
| Método | Rota | Descrição |
|--------|------|-----------|
| `GET` | `/usuarios/perfil` | Dados do perfil logado |
| `PATCH` | `/usuarios/perfil` | Atualizar endereço |
| `GET` | `/usuarios/favoritos` | Listar favoritos do usuário |

### Empresas
| Método | Rota | Descrição |
|--------|------|-----------|
| `GET` | `/empresas` | Listar empresas (filtros: `busca`, `id_categoria`, `id_banner`) |
| `GET` | `/empresas/destaques` | Empresas em destaque |
| `GET` | `/empresas/:id/cardapio` | Cardápio completo da empresa |
| `GET` | `/empresas/:id/produtos/:id_produto` | Produto específico |
| `POST` | `/empresas/:id/favoritos` | Favoritar empresa |
| `DELETE` | `/empresas/:id/favoritos` | Desfavoritar empresa |

### Pedidos
| Método | Rota | Descrição |
|--------|------|-----------|
| `GET` | `/pedidos` | Listar todos os pedidos |
| `GET` | `/pedidos/:id` | Detalhes de um pedido com itens |
| `POST` | `/pedidos` | Criar novo pedido |
| `PUT` | `/pedidos/:id` | Atualizar status do pedido |

### Conteúdo
| Método | Rota | Descrição |
|--------|------|-----------|
| `GET` | `/categorias` | Listar categorias |
| `GET` | `/banners` | Listar banners |

---

## ☁️ Deploy no Render

1. Faça push do projeto para o GitHub
2. Acesse [render.com](https://render.com) e crie um **New Web Service**
3. Conecte o repositório `GutoAgt/Pw-AGTdelivery-backend`
4. Configure:
   - **Build Command:** `npm install`
   - **Start Command:** `npm start`
   - **Environment:** `Node`
5. Crie também um **PostgreSQL Database** no Render e copie a `DATABASE_URL`
6. Em **Environment Variables**, adicione:
   - `DATABASE_URL` → string de conexão do banco Render
   - `JWT_SECRET` → sua chave secreta
   - `NODE_ENV` → `production`
7. Clique em **Deploy**

---

## 🔐 Segurança

- Senhas armazenadas com hash **bcrypt** (salt rounds: 10)
- Autenticação via **JWT** em todas as rotas protegidas
- CORS habilitado para comunicação com o app mobile
- Variáveis sensíveis isoladas em `.env` (nunca commitadas)

---

## 👨‍💻 Autor

**Augusto Jardim** — [@GutoAgt](https://github.com/GutoAgt)

---

## 📄 Licença

Este projeto está sob a licença **ISC**.
