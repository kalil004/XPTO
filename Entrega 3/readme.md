# 🌐 CRUD com Node.js + MongoDB na AWS – Projeto XPTO

Este repositório documenta como configurar e executar uma aplicação CRUD utilizando **Node.js**, **Express**, **MongoDB** e um **frontend HTML simples**, tudo hospedado em uma instância **EC2 Ubuntu na AWS**.

---

## 🛠️ Passo a Passo

### 1. Criar uma instância EC2 Ubuntu

No console da AWS:

* Tipo de instância: `t2.micro` (Free Tier elegível)
* AMI: `Ubuntu Server 22.04 LTS`
* Grupo de Segurança:

  * Porta `22` (SSH) – Acesso remoto
  * Porta `3000` (HTTP API – Backend)
  * Porta `27017` (MongoDB) – Acesso interno (opcional)
  * Porta `80` (HTTP – Frontend, se usar NGINX)

> ⚠️ **Garanta que a instância possui um IP público ou Elastic IP.**

---

### 2. Acessar a instância via SSH

```bash
ssh -i "sua-chave.pem" ubuntu@IP_DA_INSTANCIA
```

---

### 3. Instalar Docker e Node.js

#### Docker + Docker Compose:

```bash
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable docker
sudo usermod -aG docker $USER
```

> ⚠️ Saia da sessão (`exit`) e conecte novamente para aplicar a permissão do Docker.

#### Node.js:

```bash
sudo apt install -y curl
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```

---

### 4. Subir o MongoDB com Docker

```bash
docker run -d --name mongodb -p 27017:27017 mongo
```

---

### 5. Backend – API Node.js + Express

#### Criar pasta e iniciar projeto:

```bash
mkdir crud-mongo
cd crud-mongo
npm init -y
```

#### Instalar dependências:

```bash
npm install express mongoose cors
```

#### Criar arquivo `index.js`:

```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');

const app = express();
app.use(cors());
app.use(express.json());

mongoose.connect('mongodb://localhost:27017/crud', {
    useNewUrlParser: true,
    useUnifiedTopology: true
});

const Item = mongoose.model('Item', {
    name: String,
    description: String
});

// Rotas
app.get('/items', async (req, res) => {
    const items = await Item.find();
    res.json(items);
});

app.post('/items', async (req, res) => {
    const item = new Item(req.body);
    await item.save();
    res.json(item);
});

app.put('/items/:id', async (req, res) => {
    const item = await Item.findByIdAndUpdate(req.params.id, req.body, { new: true });
    res.json(item);
});

app.delete('/items/:id', async (req, res) => {
    await Item.findByIdAndDelete(req.params.id);
    res.json({ message: 'Item deletado' });
});

app.listen(3000, () => {
    console.log('Servidor rodando na porta 3000');
});
```

#### Rodar o backend:

```bash
node index.js
```

---

## 🎨 Frontend – Como deve ser estruturado

### ✅ O que é necessário para o frontend funcionar:

* Um arquivo `index.html` simples, que faz requisições à API na porta `3000` da instância.
* O frontend faz chamadas REST para os endpoints:

  * `GET /items` – Listar itens
  * `POST /items` – Adicionar itens
  * `PUT /items/:id` – Atualizar itens
  * `DELETE /items/:id` – Deletar itens

### ✅ Estrutura mínima do frontend:

* Um formulário para adicionar itens (campos de nome e descrição).
* Uma listagem dinâmica que:

  * Mostra os itens cadastrados.
  * Permite editar e excluir.
* Toda interação com o backend acontece usando **fetch API** ou similar, via JavaScript.

### ✅ Exemplo de como configurar o frontend para apontar para a API:

```javascript
const apiUrl = 'http://IP_DA_INSTANCIA:3000/items';
```

> 🔥 **Importante:** Substituir `IP_DA_INSTANCIA` pelo IP público da sua instância.

---

### 🚀 Como servir o frontend:

#### Opção 1 – Usar `serve` (rápido e simples):

Instale o `serve` globalmente:

```bash
npm install -g serve
```

Suba o frontend:

```bash
serve .
```

Por padrão vai rodar na porta `3000` (ou a próxima disponível, se estiver ocupada).

#### Opção 2 – Usar NGINX (recomendado para produção):

```bash
sudo apt install nginx
sudo mv index.html /var/www/html/index.html
sudo systemctl restart nginx
```

Acesse no navegador:

```
http://IP_DA_INSTANCIA
```

---

## 🔥 Inicialização rápida dos serviços

Sempre que a instância for reiniciada:

* Suba o MongoDB:

```bash
docker start mongodb
```

* Rode o backend:

```bash
cd ~/crud-mongo
node index.js
```

* Rode o frontend (se usar `serve`):

```bash
cd ~/crud-mongo
serve .
```

Se estiver usando NGINX, ele sobe automaticamente com o sistema.

---

## 🧪 Teste da aplicação

1. Acesse o frontend no navegador:

```
http://IP_DA_INSTANCIA
```

2. Use o formulário para criar itens.

3. Edite e exclua itens diretamente pela interface.

4. Se quiser testar direto na API:

* Listar itens:

```
GET http://IP_DA_INSTANCIA:3000/items
```

* Adicionar item:

```
POST http://IP_DA_INSTANCIA:3000/items
```

---

## ✅ Conclusão

Esse projeto demonstra como rodar uma aplicação completa — **banco de dados (MongoDB)**, **backend (Node.js + Express)** e **frontend (HTML + JS)** — em uma única instância EC2 na AWS.
