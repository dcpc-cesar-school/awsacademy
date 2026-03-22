
# Atividade Cloud (AWS)

## 📋 Visão Geral da Arquitetura

Criaremos/utilizaremos os seguintes serviços:

- **S3** - Armazenamento de arquivos (objetos)
- **EC2** - Servidor web (instância a ser integrada com o RDS)
- **RDS** - Banco de dados PostgreSQL (conectado ao EC2)
- **Aplicação Node.js** - Código para o servidor web, com conexão ao banco de dados e consumo de objetos.

---

## 0. Acesse o Console da AWS

- Você deve ter recebido um e-mail da AWS Academy com um convite para o Lab.
- Se for seu primeiro acesso, crie uma conta, ignore os campos que solicitam para associar um e-mail.
- Se já tinha conta, escolha a opção para logar com a conta existente.
- Ao acessar a AWS Academy, acesse o card do Laboratório.
- No menu da esquerda, clique em "Módulos".
- Nas seções procure um link "Iniciar laboratório de aprendizagem...".
- Se for sua primeira vez, role os termos de uso até o fim e clique em "I agree".
- Quando aparecer o menu superior do Lab, clique em "Start Lab".
- Aguarde até o ícone da AWS passar de amarelo para verde no lado esquerdo superior da página.
- Quando estiver verde, basta clicar no link "AWS".

> Todas as próximas instruções devem ser executadas no console de gerenciamento da AWS.

---

## 1. Armazenamento de objetos: S3

Vamos começar pelo S3 pois ele é independente dos outros serviços.

### 1.1. Criação de um Bucket no S3

1. Acesse o Console AWS.
2. Na barra de pesquisa superior, digite **S3** e clique no serviço S3.
3. Clique no botão laranja **Criar bucket** (canto superior direito).
4. Configure o bucket:
   - **Nome do bucket:** Digite um nome único globalmente, como `meu-app-tutorial-2025-seunome` (substitua "seunome").

   > ⚠️ Copie o nome exato do seu bucket - você precisará dele mais tarde!

   - **Região AWS:** Selecione `us-east-1` (Leste dos EUA - Norte da Virgínia) ou a região que você está usando no laboratório.
   - **Configurações de bloqueio do acesso público:** Mantenha todas as 4 caixas marcadas (já vêm marcadas por padrão). Isso garante que o bucket seja privado e só acessível via código com permissões IAM.
   - Deixe todas as outras configurações como padrão.
5. Role até o final da página e clique em **Criar bucket**.
6. Faça upload de um arquivo de teste:
   - Na lista de buckets, clique no nome do bucket que você criou.
   - Clique no botão **Carregar**.
   - Clique em **Adicionar arquivos**.
   - Selecione um arquivo de texto simples do seu computador (ou crie um `teste.txt`).
   - Clique no botão laranja **Carregar** no final da página.
   - Certifique-se que o upload foi concluído e que o bucket está funcional. Depois clique em **Fechar**.

---

## 2. Segurança: Security Groups (nas configurações de VPC)

Antes de criar o servidor EC2, vamos configurar as regras de firewall que controlarão o acesso aos serviços.

### 2.1. Criação do Grupo de Segurança para acesso ao servidor web (na EC2)

1. Na barra de pesquisa superior, digite **VPC** e clique no serviço VPC.
2. No menu lateral esquerdo, role para baixo até a seção **Segurança**.
3. Clique em **Grupos de segurança**.
4. Clique no botão laranja **Criar grupo de segurança**.
5. Configure o grupo:
   - **Nome do grupo de segurança:** `gs-ec2-web`
   - **Descrição:** Permite acesso SSH e HTTP ao servidor web
   - **VPC:** Deixe selecionada a VPC padrão (deve aparecer algo como "vpc-xxxxx | default")
6. Adicione as regras de entrada:
   - Role para baixo até a seção **Regras de entrada**.
   - Clique em **Adicionar regra**.

   > ⚠️ Não altere a regra atual, você deve adicionar uma nova regra.

   - **Regra 1 - SSH:**
     - Tipo: Selecione `SSH` no menu dropdown
     - Origem: Selecione `Qualquer local IPv4`

     > ⚠️ ATENÇÃO: Esta configuração permite acesso de qualquer IP. É adequada para laboratório, mas NUNCA use em produção.

   - Clique novamente em **Adicionar regra**.
   - **Regra 2 - Aplicação Node.js:**
     - Tipo: Selecione `TCP personalizado`
     - Intervalo de portas: Digite `3000`
     - Origem: Selecione `Qualquer local IPv4`
7. Role até o final e clique em **Criar grupo de segurança**.

### 2.2. Criação do Grupo de Segurança para permitir acesso ao banco de dados (no RDS)

1. Ainda na tela de Grupos de segurança, clique em **Criar grupo de segurança**.
2. Configure o grupo:
   - **Nome do grupo de segurança:** `gs-rds-access`
   - **Descrição:** Permite acesso ao PostgreSQL 5432
   - **VPC:** Deixe selecionada a VPC padrão
3. Adicione a regra de entrada:
   - Role para baixo até **Regras de entrada**.
   - Clique em **Adicionar regra**.
   - Tipo: Selecione `PostgreSQL` (a porta 5432 será preenchida automaticamente).
   - Origem: Selecione `Qualquer local IPv4` (ou digite `0.0.0.0/0`).
   - Descrição: Acesso ao banco de dados PostgreSQL.
4. Role até o final e clique em **Criar grupo de segurança**.

---

## 3. Aplicação: Servidor Web (Amazon EC2)

Agora criaremos o servidor que hospedará nossa aplicação Node.js.

### 3.1. Lançamento da Instância EC2

1. Na barra de pesquisa superior, digite **EC2** e clique no serviço EC2.
2. No painel do lado esquerdo, clique em **Instâncias**.
3. Clique no botão laranja **Executar instâncias** (canto superior direito).

### 3.2. Configuração da Instância

- **Nome e tags:**
  - Nome: `web-server-node`

- **Imagens de aplicação e de sistema operacional (AMI):**
  - Selecione `Amazon Linux 2023 AMI`
  - Deve aparecer uma etiqueta verde **Elegível para o nível gratuito**.

- **Tipo de instância:**
  - Selecione `t3.small`
  - Deve ter a etiqueta **Elegível para o nível gratuito**.

- **Par de chaves (login):**
  - Clique em **Criar novo par de chaves**.
  - Nome do par de chaves: `web-key`
  - Tipo de par de chaves: Deixe `RSA`
  - Formato de arquivo de chave privada: Deixe `.pem`
  - Clique em **Criar par de chaves**.
  - O arquivo será baixado automaticamente - guarde-o em local seguro.

- **Configurações de rede:**
  - Clique em **Editar** (ao lado direito).
  - Firewall (grupos de segurança): Selecione **Selecionar grupo de segurança existente**.
  - No dropdown, marque apenas o `gs-ec2-web` (desmarque qualquer outro).

- **Configurar armazenamento:**
  - Deixe o padrão: 8 GiB tipo gp3.

- **Detalhes avançados:**
  - Clique para expandir a seção **Detalhes avançados**.
  - Role para baixo até encontrar **Perfil de instância do IAM**.
  - Selecione `LabInstanceProfile` (ou `LabRole`, dependendo da sua conta AWS Academy).
  - Se não aparecer nenhuma opção, pule este passo.

- **Resumo:**
  - Número de instâncias: Deve estar `1`.
  - Clique no botão laranja **Executar instância**.
  - Aguarde a mensagem de sucesso e clique em **Visualizar todas as instâncias**.
  - Aguarde até que o Status da instância mude para **Em execução** e as Verificações de status mostrem **2/2 verificações aprovadas** (isso pode levar 2-3 minutos).

> ⚠️ Anote o endereço IP público da sua instância - aparece na coluna **Endereço IPv4 público**.

---

## 4. Instalação do Node.js na Instância EC2

Agora vamos conectar ao servidor e instalar manualmente o Node.js e suas dependências.

### 4.1. Conexão com a Instância EC2

1. Na página Instâncias, selecione sua instância `web-server-node`.
2. Clique no botão **Conectar** no topo da página.
3. Na tela que abrir, clique na aba **EC2 Instance Connect**.
4. Deixe o Nome de usuário como `ec2-user`.
5. Clique no botão laranja **Conectar**.

Uma nova aba do navegador será aberta com um terminal preto - você está dentro do servidor!

### 4.2. Instalação do Node.js e Ferramentas

No terminal que abriu, copie e cole os comandos abaixo um por um:

> ⚠️ Não precisa colar o que aparece na linha iniciada por `#`, trata-se apenas de comentário.
```bash
# Atualizar o sistema
sudo yum update -y

# Instalar Git
sudo yum install -y git

# Baixar e instalar o NVM (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Carregar o NVM no terminal atual
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# Instalar Node.js versão 20
nvm install 20

# Verificar se o Node foi instalado corretamente
node --version
# Deve aparecer algo como v20.x.x

# Verificar se o npm foi instalado
npm --version
# Deve aparecer algo como 10.x.x
```

> ✅ Pronto! O Node.js está instalado e funcionando.

---

## 5. Banco de Dados: Amazon RDS (PostgreSQL)

Agora que temos o servidor EC2 criado, podemos criar o banco de dados.

### 5.1. Criação da Instância RDS

1. Na barra de pesquisa superior, digite **RDS** e clique no serviço Aurora e RDS.
2. No painel lateral esquerdo, clique em **Bancos de dados**.
3. Clique no botão laranja **Criar banco de dados**.

### 5.2. Configuração do Banco de Dados

- **Método de criação:** Selecione **Criação padrão**.
- **Opções de mecanismo:** Clique em **PostgreSQL** (não a versão Aurora).
  - Versão do mecanismo: Deixe a versão mais recente (provavelmente será a 17).
- **Modelos:** Selecione **Sandbox**.
- **Configurações:**
  - Identificador da instância de BD: `web-db-instance`
  - Nome de usuário principal: `postgres`
  - Gerenciamento de credenciais: Deixe **Autogerenciado**.
  - Senha principal: Digite uma senha forte (ex: `MinhaSenh@123`).
  - Confirmar senha principal: Digite a mesma senha novamente.

  > ⚠️ IMPORTANTE: Anote esta senha! Você precisará dela para conectar ao banco.

- **Configuração da instância:** Deixe `db.t3.micro` ou `db.t4g.micro`.
- **Armazenamento:**
  - Deixe as configurações padrão (20 GiB, SSD de uso geral).
  - Desmarque **Habilitar escalabilidade automática do armazenamento**.

- **Conectividade:**
  - Recurso de computação: Selecione **Conectar-se a um recurso de computação do EC2**.
  - Instância do EC2: Selecione `web-server-node`.
  - Tipo de rede: Deixe IPv4.
  - VPC: Deixe a VPC padrão.
  - Acesso público: Será **Não** por padrão.
  - Grupo de segurança da VPC: Selecione **Escolher existente**, remova o `default` e adicione o `gs-rds-access`.

- **Configuração adicional** (segunda seção, no fim da página):
  - Nome do banco de dados inicial: `webappdb`

  > ⚠️ Muito importante! Isso cria o banco automaticamente.

  - Habilitar backups automáticos: **Desmarque**.
  - Habilitar monitoramento aprimorado: **Desmarque**.
  - Habilitar proteção contra exclusão: **Desmarque**.

4. Role até o final e clique em **Criar banco de dados**.
5. Aguarde 5-10 minutos até o Status mudar para **Disponível**.

### 5.3. Obter o Endpoint do Banco de Dados

1. Quando o status estiver **Disponível**, clique no nome `web-db-instance`.
2. Na seção **Conectividade e segurança**, localize o **Endpoint**.
   - Exemplo: `web-db-instance.xxxxxxxxx.us-east-1.rds.amazonaws.com`

> ⚠️ Copie/anote este endpoint - você precisará dele na aplicação!

---

## 6. Criação da Aplicação Node.js

### 6.1. Conectar ao EC2

1. Acesse EC2 > Instâncias.
2. Selecione `web-server-node`.
3. Clique em **Conectar** > aba **EC2 Instance Connect** > **Conectar**.

### 6.2. Criar a Estrutura do Projeto
```bash
# Recarregar o NVM (caso tenha perdido a conexão)
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# Criar pasta do projeto
sudo mkdir webapp
sudo chown ssm-user:ssm-user webapp || sudo chown ec2-user:ec2-user webapp

cd webapp
```

> ⚠️ O comando `chown` testa duas variações de nome de usuário. Se necessário, verifique o usuário com `whoami` e ajuste o comando.

### 6.3. Criar o arquivo package.json
```bash
sudo nano package.json
```

Cole o seguinte conteúdo:
```json
{
  "name": "aws-tutorial-server",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "@aws-sdk/client-s3": "^3.501.0",
    "express": "^4.18.2",
    "pg": "^8.11.3"
  }
}
```

Para salvar: `Ctrl + X` → `Y` → `Enter`.

### 6.4. Criar o arquivo server.js

Abra o bloco de notas, copie e cole o código abaixo, **substituindo os 3 campos indicados** pelos seus valores antes de colar no servidor:

- `SEU_ENDPOINT_RDS` → endpoint do RDS
- `SUA_SENHA_FORTE` → senha definida no RDS
- `SEU_NOME_DO_BUCKET_S3` → nome do bucket S3
```javascript
const express = require('express');
const { Pool } = require('pg');
const { S3Client, ListObjectsV2Command } = require('@aws-sdk/client-s3');

const app = express();
const PORT = 3000;

// ==========================================
// CONFIGURAÇÃO DO BANCO DE DADOS (RDS)
// ==========================================
const pool = new Pool({
  host: 'SEU_ENDPOINT_RDS',  // Ex: web-db-instance.xxxxxxxxx.us-east-1.rds.amazonaws.com
  port: 5432,
  database: 'webappdb',
  user: 'postgres',
  password: 'SUA_SENHA_FORTE',
  ssl: {
    rejectUnauthorized: false
  }
});

// ==========================================
// CONFIGURAÇÃO DO S3
// ==========================================
const BUCKET_NAME = 'SEU_NOME_DO_BUCKET_S3';
const s3Client = new S3Client({ region: 'us-east-1' });

// ==========================================
// ENDPOINT RAIZ - STATUS
// ==========================================
app.get('/', (req, res) => {
  res.send(`
    <h1>🚀 Servidor AWS Tutorial</h1>
    <p>Servidor rodando com sucesso!</p>
    <h3>Endpoints disponíveis:</h3>
    <ul>
      <li><a href="/db/test">/db/test</a> - Testar conexão com RDS</li>
      <li><a href="/db/insert">/db/insert</a> - Inserir usuário no banco</li>
      <li><a href="/db/list">/db/list</a> - Listar usuários do banco</li>
      <li><a href="/s3/list">/s3/list</a> - Listar arquivos do S3</li>
    </ul>
  `);
});

// ==========================================
// ENDPOINTS DO BANCO DE DADOS
// ==========================================

app.get('/db/test', async (req, res) => {
  try {
    const result = await pool.query('SELECT NOW()');
    res.json({
      success: true,
      message: 'Conexão com RDS PostgreSQL estabelecida!',
      timestamp: result.rows[0].now
    });
  } catch (error) {
    res.status(500).json({ success: false, message: 'Erro ao conectar ao RDS', error: error.message });
  }
});

app.get('/db/setup', async (req, res) => {
  try {
    await pool.query(`
      CREATE TABLE IF NOT EXISTS usuarios (
        id SERIAL PRIMARY KEY,
        nome VARCHAR(100),
        email VARCHAR(100),
        criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
      )
    `);
    res.json({ success: true, message: 'Tabela "usuarios" criada/verificada com sucesso!' });
  } catch (error) {
    res.status(500).json({ success: false, message: 'Erro ao criar tabela', error: error.message });
  }
});

app.get('/db/insert', async (req, res) => {
  try {
    await pool.query(`
      CREATE TABLE IF NOT EXISTS usuarios (
        id SERIAL PRIMARY KEY,
        nome VARCHAR(100),
        email VARCHAR(100),
        criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
      )
    `);

    const nome = `Usuario_${Date.now()}`;
    const email = `user${Date.now()}@exemplo.com`;

    const result = await pool.query(
      'INSERT INTO usuarios (nome, email) VALUES ($1, $2) RETURNING *',
      [nome, email]
    );

    res.json({ success: true, message: 'Usuário inserido com sucesso!', usuario: result.rows[0] });
  } catch (error) {
    res.status(500).json({ success: false, message: 'Erro ao inserir usuário', error: error.message });
  }
});

app.get('/db/list', async (req, res) => {
  try {
    const result = await pool.query('SELECT * FROM usuarios ORDER BY id DESC');
    res.json({ success: true, total: result.rows.length, usuarios: result.rows });
  } catch (error) {
    res.status(500).json({ success: false, message: 'Erro ao listar usuários', error: error.message });
  }
});

// ==========================================
// ENDPOINTS DO S3
// ==========================================

app.get('/s3/list', async (req, res) => {
  try {
    const command = new ListObjectsV2Command({ Bucket: BUCKET_NAME, MaxKeys: 10 });
    const data = await s3Client.send(command);

    const files = data.Contents ? data.Contents.map(item => ({
      nome: item.Key,
      tamanho: `${(item.Size / 1024).toFixed(2)} KB`,
      modificado: item.LastModified
    })) : [];

    res.json({ success: true, bucket: BUCKET_NAME, total_arquivos: files.length, arquivos: files });
  } catch (error) {
    res.status(500).json({ success: false, message: 'Erro ao listar arquivos do S3', error: error.message });
  }
});

// ==========================================
// INICIAR SERVIDOR
// ==========================================
app.listen(PORT, '0.0.0.0', () => {
  console.log(`✅ Servidor rodando na porta ${PORT}`);
  console.log(`🌐 Acesse: http://SEU_IP_PUBLICO:${PORT}`);
  console.log('\n📋 Endpoints disponíveis:');
  console.log(`   GET /          - Status do servidor`);
  console.log(`   GET /db/test   - Testar conexão RDS`);
  console.log(`   GET /db/setup  - Criar tabela`);
  console.log(`   GET /db/insert - Inserir usuário`);
  console.log(`   GET /db/list   - Listar usuários`);
  console.log(`   GET /s3/list   - Listar arquivos S3`);
});
```

Após substituir os 3 campos no bloco de notas, copie tudo e execute no terminal:
```bash
sudo nano server.js
```

Cole o conteúdo, depois salve: `Ctrl + X` → `Y` → `Enter`.

### 6.5. Instalar Dependências e Iniciar o Servidor
```bash
# Instalar as dependências do Node.js
npm install

# Iniciar o servidor
node server.js
```

Você deve ver:
```
✅ Servidor rodando na porta 3000
🌐 Acesse: http://SEU_IP_PUBLICO:3000
```

🎉 **O servidor está rodando!**

---

## 7. Testando a Aplicação

> ⚠️ Não será possível acessar esse endereço na wi-fi da instituição por conta da política de segurança da rede. Use a rede celular ou outra rede externa.

### 7.1. Testes no Navegador

Substitua `SEU_IP_PUBLICO` pelo IP público da sua instância EC2:

| # | Endpoint | Descrição |
|---|----------|-----------|
| 1 | `http://SEU_IP_PUBLICO:3000` | Página de status com links |
| 2 | `http://SEU_IP_PUBLICO:3000/db/test` | Conexão com RDS (retorna timestamp) |
| 3 | `http://SEU_IP_PUBLICO:3000/db/setup` | Cria a tabela `usuarios` |
| 4 | `http://SEU_IP_PUBLICO:3000/db/insert` | Insere um novo usuário |
| 5 | `http://SEU_IP_PUBLICO:3000/db/list` | Lista todos os usuários |
| 6 | `http://SEU_IP_PUBLICO:3000/s3/list` | Lista arquivos do bucket S3 |

### 7.2. Solução de Problemas

- **Erro de conexão RDS:** Verifique se o endpoint está correto, a senha está correta e o status do RDS está "Disponível".
- **Erro no S3:** Verifique se o nome do bucket está correto e se o perfil IAM (`LabInstanceProfile`) está anexado ao EC2.
- **Página não carrega:** Verifique o IP público, o Security Group `gs-ec2-web` (porta 3000) e se o servidor Node.js está rodando.

---

## 8. Acesso ao Banco de Dados via psql

### 8.1. Instalação do Cliente PostgreSQL

Abra uma nova conexão EC2 Instance Connect (para não parar o servidor) e execute:
```bash
sudo yum install -y postgresql15
```

### 8.2. Conexão ao Banco
```bash
psql -h SEU_ENDPOINT_RDS -U postgres -d webappdb
```

> ⚠️ Substitua `SEU_ENDPOINT_RDS` pelo endpoint real. Quando solicitado, digite a senha do RDS.

### 8.3. Comandos Úteis no psql
```sql
-- Listar tabelas
\dt
-- Ver estrutura da tabela
\d usuarios
-- Consultar usuários
SELECT * FROM usuarios;
-- Contar usuários
SELECT COUNT(*) FROM usuarios;
-- Sair do psql
\q
```
---

## ✅ Conclusão

Você criou com sucesso uma infraestrutura web completa na AWS, integrando:

- ✅ **EC2** - Servidor com Node.js
- ✅ **RDS** - Banco de dados PostgreSQL
- ✅ **S3** - Armazenamento de objetos
- ✅ **VPC** - Rede e segurança
