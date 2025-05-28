

# 🌐 CRUD com Node.js + MongoDB na AWS – Projeto XPTO

Este repositório documenta como configurar e executar uma aplicação CRUD utilizando **Node.js**, **Express**, **MongoDB** e um **frontend HTML** simples, tudo hospedado em uma instância **EC2 Ubuntu na AWS**.

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
  * Porta `80` (HTTP – Frontend, opcional se usar nginx)

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

### 6. Frontend – Página HTML

Crie um arquivo chamado `index.html`:

```bash
nano index.html
```

Cole o conteúdo do frontend (aquele HTML com CSS melhorado que fizemos anteriormente).

---

### 7. Servir o Frontend

Opção 1 – Usar o `serve` (Node.js):

```bash
npm install -g serve
serve .
```

Por padrão vai subir na porta `3000` (ou outra disponível).

Opção 2 – Usar NGINX (Recomendado para produção):

```bash
sudo apt install nginx
sudo mv index.html /var/www/html/index.html
sudo systemctl restart nginx
```

Acesse via navegador: `http://IP_DA_INSTANCIA`

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

* Rode o frontend (se usar serve):

```bash
cd ~/crud-mongo
serve .
```

Ou, se estiver usando NGINX, ele sobe automaticamente com o sistema.

---

## 🧪 Teste da aplicação

1. Acesse o frontend no navegador:

```
http://IP_DA_INSTANCIA
```

2. Insira itens no formulário.

3. Verifique se consegue editar e excluir itens.

4. API REST também disponível diretamente:

* Listar itens:

```
GET http://IP_DA_INSTANCIA:3000/items
```

* Adicionar item:

```
POST http://IP_DA_INSTANCIA:3000/items
```

---

## 🚀 Estrutura da aplicação

* **MongoDB** rodando em container Docker
* **Node.js + Express** rodando na própria instância EC2
* **Frontend HTML/CSS/JS** servido via `serve` ou `nginx`

---

## ✅ Conclusão

Esse projeto demonstra como subir uma aplicação completa (frontend + backend + banco de dados) em uma única instância EC2 na AWS, utilizando tecnologias simples e eficientes como Docker, Node.js, MongoDB e NGINX.

---

Se quiser, posso gerar o arquivo `README.md` pronto pra você baixar. Quer? 🔥
