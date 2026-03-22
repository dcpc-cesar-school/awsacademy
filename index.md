<h1 id="º-ciclo---avaliação-cloud-aws">2º Ciclo - Avaliação Cloud (AWS)</h1>
<h2 id="📋-visão-geral-da-arquitetura">📋 Visão Geral da Arquitetura</h2>
<p>Criaremos/utilizaremos os seguintes serviços:</p>
<ul>
<li><strong>S3</strong> - Armazenamento de arquivos (objetos)</li>
<li><strong>EC2</strong> - Servidor web (instância a ser integrada com o RDS)</li>
<li><strong>RDS</strong> - Banco de dados PostgreSQL (conectado ao EC2)</li>
<li><strong>Aplicação Node.js</strong> - Código para o servidor web, com conexão ao banco de dados e consumo de objetos.</li>
</ul>
<hr>
<h2 id="acesse-o-console-da-aws">0. Acesse o Console da AWS</h2>
<ul>
<li>Você deve ter recebido um e-mail da AWS Academy com um convite para o Lab.</li>
<li>Se for seu primeiro acesso, crie uma conta, ignore os campos que solicitam para associar um e-mail.</li>
<li>Se já tinha conta, escolha a opção para logar com a conta existente.</li>
<li>Ao acessar a AWS Academy, acesse o card do Laboratório.</li>
<li>No menu da esquerda, clique em “Módulos”.</li>
<li>Nas seções procure um link “Iniciar laboratório de aprendizagem…”.</li>
<li>Se for sua primeira vez, role os termos de uso até o fim e clique em “I agree”.</li>
<li>Quando aparecer o menu superior do Lab, clique em “Start Lab”.</li>
<li>Aguarde até o ícone da AWS passar de amarelo para verde no lado esquerdo superior da página.</li>
<li>Quando estiver verde, basta clicar no link “AWS”.</li>
</ul>
<blockquote>
<p>Todas as próximas instruções devem ser executadas no console de gerenciamento da AWS.</p>
</blockquote>
<hr>
<h2 id="armazenamento-de-objetos-s3">1. Armazenamento de objetos: S3</h2>
<p>Vamos começar pelo S3 pois ele é independente dos outros serviços.</p>
<h3 id="criação-de-um-bucket-no-s3">1.1. Criação de um Bucket no S3</h3>
<ol>
<li>
<p>Acesse o Console AWS.</p>
</li>
<li>
<p>Na barra de pesquisa superior, digite <strong>S3</strong> e clique no serviço S3.</p>
</li>
<li>
<p>Clique no botão laranja <strong>Criar bucket</strong> (canto superior direito).</p>
</li>
<li>
<p>Configure o bucket:</p>
<ul>
<li><strong>Nome do bucket:</strong> Digite um nome único globalmente, como <code>meu-app-tutorial-2025-seunome</code> (substitua “seunome”).</li>
</ul>
<blockquote>
<p>⚠️ Copie o nome exato do seu bucket - você precisará dele mais tarde!</p>
</blockquote>
<ul>
<li><strong>Região AWS:</strong> Selecione <code>us-east-1</code> (Leste dos EUA - Norte da Virgínia) ou a região que você está usando no laboratório.</li>
<li><strong>Configurações de bloqueio do acesso público:</strong> Mantenha todas as 4 caixas marcadas (já vêm marcadas por padrão). Isso garante que o bucket seja privado e só acessível via código com permissões IAM.</li>
<li>Deixe todas as outras configurações como padrão.</li>
</ul>
</li>
<li>
<p>Role até o final da página e clique em <strong>Criar bucket</strong>.</p>
</li>
<li>
<p>Faça upload de um arquivo de teste:</p>
<ul>
<li>Na lista de buckets, clique no nome do bucket que você criou.</li>
<li>Clique no botão <strong>Carregar</strong>.</li>
<li>Clique em <strong>Adicionar arquivos</strong>.</li>
<li>Selecione um arquivo de texto simples do seu computador (ou crie um <code>teste.txt</code>).</li>
<li>Clique no botão laranja <strong>Carregar</strong> no final da página.</li>
<li>Certifique-se que o upload foi concluído e que o bucket está funcional. Depois clique em <strong>Fechar</strong>.</li>
</ul>
</li>
</ol>
<hr>
<h2 id="segurança-security-groups-nas-configurações-de-vpc">2. Segurança: Security Groups (nas configurações de VPC)</h2>
<p>Antes de criar o servidor EC2, vamos configurar as regras de firewall que controlarão o acesso aos serviços.</p>
<h3 id="criação-do-grupo-de-segurança-para-acesso-ao-servidor-web-na-ec2">2.1. Criação do Grupo de Segurança para acesso ao servidor web (na EC2)</h3>
<ol>
<li>
<p>Na barra de pesquisa superior, digite <strong>VPC</strong> e clique no serviço VPC.</p>
</li>
<li>
<p>No menu lateral esquerdo, role para baixo até a seção <strong>Segurança</strong>.</p>
</li>
<li>
<p>Clique em <strong>Grupos de segurança</strong>.</p>
</li>
<li>
<p>Clique no botão laranja <strong>Criar grupo de segurança</strong>.</p>
</li>
<li>
<p>Configure o grupo:</p>
<ul>
<li><strong>Nome do grupo de segurança:</strong> <code>gs-ec2-web</code></li>
<li><strong>Descrição:</strong> Permite acesso SSH e HTTP ao servidor web</li>
<li><strong>VPC:</strong> Deixe selecionada a VPC padrão (deve aparecer algo como “vpc-xxxxx | default”)</li>
</ul>
</li>
<li>
<p>Adicione as regras de entrada:</p>
<ul>
<li>Role para baixo até a seção <strong>Regras de entrada</strong>.</li>
<li>Clique em <strong>Adicionar regra</strong>.</li>
</ul>
<blockquote>
<p>⚠️ Não altere a regra atual, você deve adicionar uma nova regra.</p>
</blockquote>
<ul>
<li>
<p><strong>Regra 1 - SSH:</strong></p>
<ul>
<li>Tipo: Selecione <code>SSH</code> no menu dropdown</li>
<li>Origem: Selecione <code>Qualquer local IPv4</code></li>
</ul>
<blockquote>
<p>⚠️ ATENÇÃO: Esta configuração permite acesso de qualquer IP. É adequada para laboratório, mas NUNCA use em produção.</p>
</blockquote>
</li>
<li>
<p>Clique novamente em <strong>Adicionar regra</strong>.</p>
</li>
<li>
<p><strong>Regra 2 - Aplicação Node.js:</strong></p>
<ul>
<li>Tipo: Selecione <code>TCP personalizado</code></li>
<li>Intervalo de portas: Digite <code>3000</code></li>
<li>Origem: Selecione <code>Qualquer local IPv4</code></li>
</ul>
</li>
</ul>
</li>
<li>
<p>Role até o final e clique em <strong>Criar grupo de segurança</strong>.</p>
</li>
</ol>
<h3 id="criação-do-grupo-de-segurança-para-permitir-acesso-ao-banco-de-dados-no-rds">2.2. Criação do Grupo de Segurança para permitir acesso ao banco de dados (no RDS)</h3>
<ol>
<li>Ainda na tela de Grupos de segurança, clique em <strong>Criar grupo de segurança</strong>.</li>
<li>Configure o grupo:
<ul>
<li><strong>Nome do grupo de segurança:</strong> <code>gs-rds-access</code></li>
<li><strong>Descrição:</strong> Permite acesso ao PostgreSQL 5432</li>
<li><strong>VPC:</strong> Deixe selecionada a VPC padrão</li>
</ul>
</li>
<li>Adicione a regra de entrada:
<ul>
<li>Role para baixo até <strong>Regras de entrada</strong>.</li>
<li>Clique em <strong>Adicionar regra</strong>.</li>
<li>Tipo: Selecione <code>PostgreSQL</code> (a porta 5432 será preenchida automaticamente).</li>
<li>Origem: Selecione <code>Qualquer local IPv4</code> (ou digite <code>0.0.0.0/0</code>).</li>
<li>Descrição: Acesso ao banco de dados PostgreSQL.</li>
</ul>
</li>
<li>Role até o final e clique em <strong>Criar grupo de segurança</strong>.</li>
</ol>
<hr>
<h2 id="aplicação-servidor-web-amazon-ec2">3. Aplicação: Servidor Web (Amazon EC2)</h2>
<p>Agora criaremos o servidor que hospedará nossa aplicação Node.js.</p>
<h3 id="lançamento-da-instância-ec2">3.1. Lançamento da Instância EC2</h3>
<ol>
<li>Na barra de pesquisa superior, digite <strong>EC2</strong> e clique no serviço EC2.</li>
<li>No painel do lado esquerdo, clique em <strong>Instâncias</strong>.</li>
<li>Clique no botão laranja <strong>Executar instâncias</strong> (canto superior direito).</li>
</ol>
<h3 id="configuração-da-instância">3.2. Configuração da Instância</h3>
<ul>
<li>
<p><strong>Nome e tags:</strong></p>
<ul>
<li>Nome: <code>web-server-node</code></li>
</ul>
</li>
<li>
<p><strong>Imagens de aplicação e de sistema operacional (AMI):</strong></p>
<ul>
<li>Selecione <code>Amazon Linux 2023 AMI</code></li>
<li>Deve aparecer uma etiqueta verde <strong>Elegível para o nível gratuito</strong>.</li>
</ul>
</li>
<li>
<p><strong>Tipo de instância:</strong></p>
<ul>
<li>Selecione <code>t3.small</code></li>
<li>Deve ter a etiqueta <strong>Elegível para o nível gratuito</strong>.</li>
</ul>
</li>
<li>
<p><strong>Par de chaves (login):</strong></p>
<ul>
<li>Clique em <strong>Criar novo par de chaves</strong>.</li>
<li>Nome do par de chaves: <code>web-key</code></li>
<li>Tipo de par de chaves: Deixe <code>RSA</code></li>
<li>Formato de arquivo de chave privada: Deixe <code>.pem</code></li>
<li>Clique em <strong>Criar par de chaves</strong>.</li>
<li>O arquivo será baixado automaticamente - guarde-o em local seguro.</li>
</ul>
</li>
<li>
<p><strong>Configurações de rede:</strong></p>
<ul>
<li>Clique em <strong>Editar</strong> (ao lado direito).</li>
<li>Firewall (grupos de segurança): Selecione <strong>Selecionar grupo de segurança existente</strong>.</li>
<li>No dropdown, marque apenas o <code>gs-ec2-web</code> (desmarque qualquer outro).</li>
</ul>
</li>
<li>
<p><strong>Configurar armazenamento:</strong></p>
<ul>
<li>Deixe o padrão: 8 GiB tipo gp3.</li>
</ul>
</li>
<li>
<p><strong>Detalhes avançados:</strong></p>
<ul>
<li>Clique para expandir a seção <strong>Detalhes avançados</strong>.</li>
<li>Role para baixo até encontrar <strong>Perfil de instância do IAM</strong>.</li>
<li>Selecione <code>LabInstanceProfile</code> (ou <code>LabRole</code>, dependendo da sua conta AWS Academy).</li>
<li>Se não aparecer nenhuma opção, pule este passo.</li>
</ul>
</li>
<li>
<p><strong>Resumo:</strong></p>
<ul>
<li>Número de instâncias: Deve estar <code>1</code>.</li>
<li>Clique no botão laranja <strong>Executar instância</strong>.</li>
<li>Aguarde a mensagem de sucesso e clique em <strong>Visualizar todas as instâncias</strong>.</li>
<li>Aguarde até que o Status da instância mude para <strong>Em execução</strong> e as Verificações de status mostrem <strong>2/2 verificações aprovadas</strong> (isso pode levar 2-3 minutos).</li>
</ul>
</li>
</ul>
<blockquote>
<p>⚠️ Anote o endereço IP público da sua instância - aparece na coluna <strong>Endereço IPv4 público</strong>.</p>
</blockquote>
<hr>
<h2 id="instalação-do-node.js-na-instância-ec2">4. Instalação do Node.js na Instância EC2</h2>
<p>Agora vamos conectar ao servidor e instalar manualmente o Node.js e suas dependências.</p>
<h3 id="conexão-com-a-instância-ec2">4.1. Conexão com a Instância EC2</h3>
<ol>
<li>Na página Instâncias, selecione sua instância <code>web-server-node</code>.</li>
<li>Clique no botão <strong>Conectar</strong> no topo da página.</li>
<li>Na tela que abrir, clique na aba <strong>EC2 Instance Connect</strong>.</li>
<li>Deixe o Nome de usuário como <code>ec2-user</code>.</li>
<li>Clique no botão laranja <strong>Conectar</strong>.</li>
</ol>
<p>Uma nova aba do navegador será aberta com um terminal preto - você está dentro do servidor!</p>
<h3 id="instalação-do-node.js-e-ferramentas">4.2. Instalação do Node.js e Ferramentas</h3>
<p>No terminal que abriu, copie e cole os comandos abaixo um por um:</p>
<blockquote>
<p>⚠️ Não precisa colar o que aparece na linha iniciada por <code>#</code>, trata-se apenas de comentário.</p>
</blockquote>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># Atualizar o sistema</span>
<span class="token function">sudo</span> yum update -y

<span class="token comment"># Instalar Git</span>
<span class="token function">sudo</span> yum <span class="token function">install</span> -y <span class="token function">git</span>

<span class="token comment"># Baixar e instalar o NVM (Node Version Manager)</span>
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh <span class="token operator">|</span> <span class="token function">bash</span>

<span class="token comment"># Carregar o NVM no terminal atual</span>
<span class="token function">export</span> NVM_DIR<span class="token operator">=</span><span class="token string">"<span class="token variable">$HOME</span>/.nvm"</span>
<span class="token punctuation">[</span> -s <span class="token string">"<span class="token variable">$NVM_DIR</span>/nvm.sh"</span> <span class="token punctuation">]</span> <span class="token operator">&amp;&amp;</span> \. <span class="token string">"<span class="token variable">$NVM_DIR</span>/nvm.sh"</span>

<span class="token comment"># Instalar Node.js versão 20</span>
nvm <span class="token function">install</span> 20

<span class="token comment"># Verificar se o Node foi instalado corretamente</span>
node --version
<span class="token comment"># Deve aparecer algo como v20.x.x</span>

<span class="token comment"># Verificar se o npm foi instalado</span>
<span class="token function">npm</span> --version
<span class="token comment"># Deve aparecer algo como 10.x.x</span>
</code></pre>
<blockquote>
<p>✅ Pronto! O Node.js está instalado e funcionando.</p>
</blockquote>
<hr>
<h2 id="banco-de-dados-amazon-rds-postgresql">5. Banco de Dados: Amazon RDS (PostgreSQL)</h2>
<p>Agora que temos o servidor EC2 criado, podemos criar o banco de dados.</p>
<h3 id="criação-da-instância-rds">5.1. Criação da Instância RDS</h3>
<ol>
<li>Na barra de pesquisa superior, digite <strong>RDS</strong> e clique no serviço Aurora e RDS.</li>
<li>No painel lateral esquerdo, clique em <strong>Bancos de dados</strong>.</li>
<li>Clique no botão laranja <strong>Criar banco de dados</strong>.</li>
</ol>
<h3 id="configuração-do-banco-de-dados">5.2. Configuração do Banco de Dados</h3>
<ul>
<li>
<p><strong>Método de criação:</strong> Selecione <strong>Criação padrão</strong>.</p>
</li>
<li>
<p><strong>Opções de mecanismo:</strong> Clique em <strong>PostgreSQL</strong> (não a versão Aurora).</p>
<ul>
<li>Versão do mecanismo: Deixe a versão mais recente (provavelmente será a 17).</li>
</ul>
</li>
<li>
<p><strong>Modelos:</strong> Selecione <strong>Sandbox</strong>.</p>
</li>
<li>
<p><strong>Configurações:</strong></p>
<ul>
<li>Identificador da instância de BD: <code>web-db-instance</code></li>
<li>Nome de usuário principal: <code>postgres</code></li>
<li>Gerenciamento de credenciais: Deixe <strong>Autogerenciado</strong>.</li>
<li>Senha principal: Digite uma senha forte (ex: <code>MinhaSenh@123</code>).</li>
<li>Confirmar senha principal: Digite a mesma senha novamente.</li>
</ul>
<blockquote>
<p>⚠️ IMPORTANTE: Anote esta senha! Você precisará dela para conectar ao banco.</p>
</blockquote>
</li>
<li>
<p><strong>Configuração da instância:</strong> Deixe <code>db.t3.micro</code> ou <code>db.t4g.micro</code>.</p>
</li>
<li>
<p><strong>Armazenamento:</strong></p>
<ul>
<li>Deixe as configurações padrão (20 GiB, SSD de uso geral).</li>
<li>Desmarque <strong>Habilitar escalabilidade automática do armazenamento</strong>.</li>
</ul>
</li>
<li>
<p><strong>Conectividade:</strong></p>
<ul>
<li>Recurso de computação: Selecione <strong>Conectar-se a um recurso de computação do EC2</strong>.</li>
<li>Instância do EC2: Selecione <code>web-server-node</code>.</li>
<li>Tipo de rede: Deixe IPv4.</li>
<li>VPC: Deixe a VPC padrão.</li>
<li>Acesso público: Será <strong>Não</strong> por padrão.</li>
<li>Grupo de segurança da VPC: Selecione <strong>Escolher existente</strong>, remova o <code>default</code> e adicione o <code>gs-rds-access</code>.</li>
</ul>
</li>
<li>
<p><strong>Configuração adicional</strong> (segunda seção, no fim da página):</p>
<ul>
<li>Nome do banco de dados inicial: <code>webappdb</code></li>
</ul>
<blockquote>
<p>⚠️ Muito importante! Isso cria o banco automaticamente.</p>
</blockquote>
<ul>
<li>Habilitar backups automáticos: <strong>Desmarque</strong>.</li>
<li>Habilitar monitoramento aprimorado: <strong>Desmarque</strong>.</li>
<li>Habilitar proteção contra exclusão: <strong>Desmarque</strong>.</li>
</ul>
</li>
</ul>
<ol start="4">
<li>Role até o final e clique em <strong>Criar banco de dados</strong>.</li>
<li>Aguarde 5-10 minutos até o Status mudar para <strong>Disponível</strong>.</li>
</ol>
<h3 id="obter-o-endpoint-do-banco-de-dados">5.3. Obter o Endpoint do Banco de Dados</h3>
<ol>
<li>Quando o status estiver <strong>Disponível</strong>, clique no nome <code>web-db-instance</code>.</li>
<li>Na seção <strong>Conectividade e segurança</strong>, localize o <strong>Endpoint</strong>.
<ul>
<li>Exemplo: <code>web-db-instance.xxxxxxxxx.us-east-1.rds.amazonaws.com</code></li>
</ul>
</li>
</ol>
<blockquote>
<p>⚠️ Copie/anote este endpoint - você precisará dele na aplicação!</p>
</blockquote>
<hr>
<h2 id="criação-da-aplicação-node.js">6. Criação da Aplicação Node.js</h2>
<h3 id="conectar-ao-ec2">6.1. Conectar ao EC2</h3>
<ol>
<li>Acesse EC2 &gt; Instâncias.</li>
<li>Selecione <code>web-server-node</code>.</li>
<li>Clique em <strong>Conectar</strong> &gt; aba <strong>EC2 Instance Connect</strong> &gt; <strong>Conectar</strong>.</li>
</ol>
<h3 id="criar-a-estrutura-do-projeto">6.2. Criar a Estrutura do Projeto</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># Recarregar o NVM (caso tenha perdido a conexão)</span>
<span class="token function">export</span> NVM_DIR<span class="token operator">=</span><span class="token string">"<span class="token variable">$HOME</span>/.nvm"</span>
<span class="token punctuation">[</span> -s <span class="token string">"<span class="token variable">$NVM_DIR</span>/nvm.sh"</span> <span class="token punctuation">]</span> <span class="token operator">&amp;&amp;</span> \. <span class="token string">"<span class="token variable">$NVM_DIR</span>/nvm.sh"</span>

<span class="token comment"># Criar pasta do projeto</span>
<span class="token function">sudo</span> <span class="token function">mkdir</span> webapp
<span class="token function">sudo</span> <span class="token function">chown</span> ssm-user:ssm-user webapp <span class="token operator">||</span> <span class="token function">sudo</span> <span class="token function">chown</span> ec2-user:ec2-user webapp

<span class="token function">cd</span> webapp
</code></pre>
<blockquote>
<p>⚠️ O comando <code>chown</code> testa duas variações de nome de usuário. Se necessário, verifique o usuário com <code>whoami</code> e ajuste o comando.</p>
</blockquote>
<h3 id="criar-o-arquivo-package.json">6.3. Criar o arquivo package.json</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> <span class="token function">nano</span> package.json
</code></pre>
<p>Cole o seguinte conteúdo:</p>
<pre class=" language-json"><code class="prism  language-json"><span class="token punctuation">{</span>
  <span class="token string">"name"</span><span class="token punctuation">:</span> <span class="token string">"aws-tutorial-server"</span><span class="token punctuation">,</span>
  <span class="token string">"version"</span><span class="token punctuation">:</span> <span class="token string">"1.0.0"</span><span class="token punctuation">,</span>
  <span class="token string">"main"</span><span class="token punctuation">:</span> <span class="token string">"server.js"</span><span class="token punctuation">,</span>
  <span class="token string">"scripts"</span><span class="token punctuation">:</span> <span class="token punctuation">{</span>
    <span class="token string">"start"</span><span class="token punctuation">:</span> <span class="token string">"node server.js"</span>
  <span class="token punctuation">}</span><span class="token punctuation">,</span>
  <span class="token string">"dependencies"</span><span class="token punctuation">:</span> <span class="token punctuation">{</span>
    <span class="token string">"@aws-sdk/client-s3"</span><span class="token punctuation">:</span> <span class="token string">"^3.501.0"</span><span class="token punctuation">,</span>
    <span class="token string">"express"</span><span class="token punctuation">:</span> <span class="token string">"^4.18.2"</span><span class="token punctuation">,</span>
    <span class="token string">"pg"</span><span class="token punctuation">:</span> <span class="token string">"^8.11.3"</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre>
<p>Para salvar: <code>Ctrl + X</code> → <code>Y</code> → <code>Enter</code>.</p>
<h3 id="criar-o-arquivo-server.js">6.4. Criar o arquivo server.js</h3>
<p>Abra o bloco de notas, copie e cole o código abaixo, <strong>substituindo os 3 campos indicados</strong> pelos seus valores antes de colar no servidor:</p>
<ul>
<li><code>SEU_ENDPOINT_RDS</code> → endpoint do RDS</li>
<li><code>SUA_SENHA_FORTE</code> → senha definida no RDS</li>
<li><code>SEU_NOME_DO_BUCKET_S3</code> → nome do bucket S3</li>
</ul>
<pre class=" language-javascript"><code class="prism  language-javascript"><span class="token keyword">const</span> express <span class="token operator">=</span> <span class="token function">require</span><span class="token punctuation">(</span><span class="token string">'express'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token keyword">const</span> <span class="token punctuation">{</span> Pool <span class="token punctuation">}</span> <span class="token operator">=</span> <span class="token function">require</span><span class="token punctuation">(</span><span class="token string">'pg'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token keyword">const</span> <span class="token punctuation">{</span> S3Client<span class="token punctuation">,</span> ListObjectsV2Command <span class="token punctuation">}</span> <span class="token operator">=</span> <span class="token function">require</span><span class="token punctuation">(</span><span class="token string">'@aws-sdk/client-s3'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token keyword">const</span> app <span class="token operator">=</span> <span class="token function">express</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token keyword">const</span> PORT <span class="token operator">=</span> <span class="token number">3000</span><span class="token punctuation">;</span>

<span class="token comment">// ==========================================</span>
<span class="token comment">// CONFIGURAÇÃO DO BANCO DE DADOS (RDS)</span>
<span class="token comment">// ==========================================</span>
<span class="token keyword">const</span> pool <span class="token operator">=</span> <span class="token keyword">new</span> <span class="token class-name">Pool</span><span class="token punctuation">(</span><span class="token punctuation">{</span>
  host<span class="token punctuation">:</span> <span class="token string">'SEU_ENDPOINT_RDS'</span><span class="token punctuation">,</span>  <span class="token comment">// Ex: web-db-instance.xxxxxxxxx.us-east-1.rds.amazonaws.com</span>
  port<span class="token punctuation">:</span> <span class="token number">5432</span><span class="token punctuation">,</span>
  database<span class="token punctuation">:</span> <span class="token string">'webappdb'</span><span class="token punctuation">,</span>
  user<span class="token punctuation">:</span> <span class="token string">'postgres'</span><span class="token punctuation">,</span>
  password<span class="token punctuation">:</span> <span class="token string">'SUA_SENHA_FORTE'</span><span class="token punctuation">,</span>
  ssl<span class="token punctuation">:</span> <span class="token punctuation">{</span>
    rejectUnauthorized<span class="token punctuation">:</span> <span class="token boolean">false</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token comment">// ==========================================</span>
<span class="token comment">// CONFIGURAÇÃO DO S3</span>
<span class="token comment">// ==========================================</span>
<span class="token keyword">const</span> BUCKET_NAME <span class="token operator">=</span> <span class="token string">'SEU_NOME_DO_BUCKET_S3'</span><span class="token punctuation">;</span>
<span class="token keyword">const</span> s3Client <span class="token operator">=</span> <span class="token keyword">new</span> <span class="token class-name">S3Client</span><span class="token punctuation">(</span><span class="token punctuation">{</span> region<span class="token punctuation">:</span> <span class="token string">'us-east-1'</span> <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token comment">// ==========================================</span>
<span class="token comment">// ENDPOINT RAIZ - STATUS</span>
<span class="token comment">// ==========================================</span>
app<span class="token punctuation">.</span><span class="token keyword">get</span><span class="token punctuation">(</span><span class="token string">'/'</span><span class="token punctuation">,</span> <span class="token punctuation">(</span>req<span class="token punctuation">,</span> res<span class="token punctuation">)</span> <span class="token operator">=&gt;</span> <span class="token punctuation">{</span>
  res<span class="token punctuation">.</span><span class="token function">send</span><span class="token punctuation">(</span><span class="token template-string"><span class="token string">`
    &lt;h1&gt;🚀 Servidor AWS Tutorial&lt;/h1&gt;
    &lt;p&gt;Servidor rodando com sucesso!&lt;/p&gt;
    &lt;h3&gt;Endpoints disponíveis:&lt;/h3&gt;
    &lt;ul&gt;
      &lt;li&gt;&lt;a href="/db/test"&gt;/db/test&lt;/a&gt; - Testar conexão com RDS&lt;/li&gt;
      &lt;li&gt;&lt;a href="/db/insert"&gt;/db/insert&lt;/a&gt; - Inserir usuário no banco&lt;/li&gt;
      &lt;li&gt;&lt;a href="/db/list"&gt;/db/list&lt;/a&gt; - Listar usuários do banco&lt;/li&gt;
      &lt;li&gt;&lt;a href="/s3/list"&gt;/s3/list&lt;/a&gt; - Listar arquivos do S3&lt;/li&gt;
    &lt;/ul&gt;
  `</span></span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token comment">// ==========================================</span>
<span class="token comment">// ENDPOINTS DO BANCO DE DADOS</span>
<span class="token comment">// ==========================================</span>

app<span class="token punctuation">.</span><span class="token keyword">get</span><span class="token punctuation">(</span><span class="token string">'/db/test'</span><span class="token punctuation">,</span> <span class="token keyword">async</span> <span class="token punctuation">(</span>req<span class="token punctuation">,</span> res<span class="token punctuation">)</span> <span class="token operator">=&gt;</span> <span class="token punctuation">{</span>
  <span class="token keyword">try</span> <span class="token punctuation">{</span>
    <span class="token keyword">const</span> result <span class="token operator">=</span> <span class="token keyword">await</span> pool<span class="token punctuation">.</span><span class="token function">query</span><span class="token punctuation">(</span><span class="token string">'SELECT NOW()'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    res<span class="token punctuation">.</span><span class="token function">json</span><span class="token punctuation">(</span><span class="token punctuation">{</span>
      success<span class="token punctuation">:</span> <span class="token boolean">true</span><span class="token punctuation">,</span>
      message<span class="token punctuation">:</span> <span class="token string">'Conexão com RDS PostgreSQL estabelecida!'</span><span class="token punctuation">,</span>
      timestamp<span class="token punctuation">:</span> result<span class="token punctuation">.</span>rows<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span><span class="token punctuation">.</span>now
    <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span> <span class="token keyword">catch</span> <span class="token punctuation">(</span><span class="token class-name">error</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    res<span class="token punctuation">.</span><span class="token function">status</span><span class="token punctuation">(</span><span class="token number">500</span><span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token function">json</span><span class="token punctuation">(</span><span class="token punctuation">{</span> success<span class="token punctuation">:</span> <span class="token boolean">false</span><span class="token punctuation">,</span> message<span class="token punctuation">:</span> <span class="token string">'Erro ao conectar ao RDS'</span><span class="token punctuation">,</span> error<span class="token punctuation">:</span> error<span class="token punctuation">.</span>message <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

app<span class="token punctuation">.</span><span class="token keyword">get</span><span class="token punctuation">(</span><span class="token string">'/db/setup'</span><span class="token punctuation">,</span> <span class="token keyword">async</span> <span class="token punctuation">(</span>req<span class="token punctuation">,</span> res<span class="token punctuation">)</span> <span class="token operator">=&gt;</span> <span class="token punctuation">{</span>
  <span class="token keyword">try</span> <span class="token punctuation">{</span>
    <span class="token keyword">await</span> pool<span class="token punctuation">.</span><span class="token function">query</span><span class="token punctuation">(</span><span class="token template-string"><span class="token string">`
      CREATE TABLE IF NOT EXISTS usuarios (
        id SERIAL PRIMARY KEY,
        nome VARCHAR(100),
        email VARCHAR(100),
        criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
      )
    `</span></span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    res<span class="token punctuation">.</span><span class="token function">json</span><span class="token punctuation">(</span><span class="token punctuation">{</span> success<span class="token punctuation">:</span> <span class="token boolean">true</span><span class="token punctuation">,</span> message<span class="token punctuation">:</span> <span class="token string">'Tabela "usuarios" criada/verificada com sucesso!'</span> <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span> <span class="token keyword">catch</span> <span class="token punctuation">(</span><span class="token class-name">error</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    res<span class="token punctuation">.</span><span class="token function">status</span><span class="token punctuation">(</span><span class="token number">500</span><span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token function">json</span><span class="token punctuation">(</span><span class="token punctuation">{</span> success<span class="token punctuation">:</span> <span class="token boolean">false</span><span class="token punctuation">,</span> message<span class="token punctuation">:</span> <span class="token string">'Erro ao criar tabela'</span><span class="token punctuation">,</span> error<span class="token punctuation">:</span> error<span class="token punctuation">.</span>message <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

app<span class="token punctuation">.</span><span class="token keyword">get</span><span class="token punctuation">(</span><span class="token string">'/db/insert'</span><span class="token punctuation">,</span> <span class="token keyword">async</span> <span class="token punctuation">(</span>req<span class="token punctuation">,</span> res<span class="token punctuation">)</span> <span class="token operator">=&gt;</span> <span class="token punctuation">{</span>
  <span class="token keyword">try</span> <span class="token punctuation">{</span>
    <span class="token keyword">await</span> pool<span class="token punctuation">.</span><span class="token function">query</span><span class="token punctuation">(</span><span class="token template-string"><span class="token string">`
      CREATE TABLE IF NOT EXISTS usuarios (
        id SERIAL PRIMARY KEY,
        nome VARCHAR(100),
        email VARCHAR(100),
        criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
      )
    `</span></span><span class="token punctuation">)</span><span class="token punctuation">;</span>

    <span class="token keyword">const</span> nome <span class="token operator">=</span> <span class="token template-string"><span class="token string">`Usuario_</span><span class="token interpolation"><span class="token interpolation-punctuation punctuation">${</span>Date<span class="token punctuation">.</span><span class="token function">now</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token interpolation-punctuation punctuation">}</span></span><span class="token string">`</span></span><span class="token punctuation">;</span>
    <span class="token keyword">const</span> email <span class="token operator">=</span> <span class="token template-string"><span class="token string">`user</span><span class="token interpolation"><span class="token interpolation-punctuation punctuation">${</span>Date<span class="token punctuation">.</span><span class="token function">now</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token interpolation-punctuation punctuation">}</span></span><span class="token string">@exemplo.com`</span></span><span class="token punctuation">;</span>

    <span class="token keyword">const</span> result <span class="token operator">=</span> <span class="token keyword">await</span> pool<span class="token punctuation">.</span><span class="token function">query</span><span class="token punctuation">(</span>
      <span class="token string">'INSERT INTO usuarios (nome, email) VALUES ($1, $2) RETURNING *'</span><span class="token punctuation">,</span>
      <span class="token punctuation">[</span>nome<span class="token punctuation">,</span> email<span class="token punctuation">]</span>
    <span class="token punctuation">)</span><span class="token punctuation">;</span>

    res<span class="token punctuation">.</span><span class="token function">json</span><span class="token punctuation">(</span><span class="token punctuation">{</span> success<span class="token punctuation">:</span> <span class="token boolean">true</span><span class="token punctuation">,</span> message<span class="token punctuation">:</span> <span class="token string">'Usuário inserido com sucesso!'</span><span class="token punctuation">,</span> usuario<span class="token punctuation">:</span> result<span class="token punctuation">.</span>rows<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span> <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span> <span class="token keyword">catch</span> <span class="token punctuation">(</span><span class="token class-name">error</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    res<span class="token punctuation">.</span><span class="token function">status</span><span class="token punctuation">(</span><span class="token number">500</span><span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token function">json</span><span class="token punctuation">(</span><span class="token punctuation">{</span> success<span class="token punctuation">:</span> <span class="token boolean">false</span><span class="token punctuation">,</span> message<span class="token punctuation">:</span> <span class="token string">'Erro ao inserir usuário'</span><span class="token punctuation">,</span> error<span class="token punctuation">:</span> error<span class="token punctuation">.</span>message <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

app<span class="token punctuation">.</span><span class="token keyword">get</span><span class="token punctuation">(</span><span class="token string">'/db/list'</span><span class="token punctuation">,</span> <span class="token keyword">async</span> <span class="token punctuation">(</span>req<span class="token punctuation">,</span> res<span class="token punctuation">)</span> <span class="token operator">=&gt;</span> <span class="token punctuation">{</span>
  <span class="token keyword">try</span> <span class="token punctuation">{</span>
    <span class="token keyword">const</span> result <span class="token operator">=</span> <span class="token keyword">await</span> pool<span class="token punctuation">.</span><span class="token function">query</span><span class="token punctuation">(</span><span class="token string">'SELECT * FROM usuarios ORDER BY id DESC'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    res<span class="token punctuation">.</span><span class="token function">json</span><span class="token punctuation">(</span><span class="token punctuation">{</span> success<span class="token punctuation">:</span> <span class="token boolean">true</span><span class="token punctuation">,</span> total<span class="token punctuation">:</span> result<span class="token punctuation">.</span>rows<span class="token punctuation">.</span>length<span class="token punctuation">,</span> usuarios<span class="token punctuation">:</span> result<span class="token punctuation">.</span>rows <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span> <span class="token keyword">catch</span> <span class="token punctuation">(</span><span class="token class-name">error</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    res<span class="token punctuation">.</span><span class="token function">status</span><span class="token punctuation">(</span><span class="token number">500</span><span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token function">json</span><span class="token punctuation">(</span><span class="token punctuation">{</span> success<span class="token punctuation">:</span> <span class="token boolean">false</span><span class="token punctuation">,</span> message<span class="token punctuation">:</span> <span class="token string">'Erro ao listar usuários'</span><span class="token punctuation">,</span> error<span class="token punctuation">:</span> error<span class="token punctuation">.</span>message <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token comment">// ==========================================</span>
<span class="token comment">// ENDPOINTS DO S3</span>
<span class="token comment">// ==========================================</span>

app<span class="token punctuation">.</span><span class="token keyword">get</span><span class="token punctuation">(</span><span class="token string">'/s3/list'</span><span class="token punctuation">,</span> <span class="token keyword">async</span> <span class="token punctuation">(</span>req<span class="token punctuation">,</span> res<span class="token punctuation">)</span> <span class="token operator">=&gt;</span> <span class="token punctuation">{</span>
  <span class="token keyword">try</span> <span class="token punctuation">{</span>
    <span class="token keyword">const</span> command <span class="token operator">=</span> <span class="token keyword">new</span> <span class="token class-name">ListObjectsV2Command</span><span class="token punctuation">(</span><span class="token punctuation">{</span> Bucket<span class="token punctuation">:</span> BUCKET_NAME<span class="token punctuation">,</span> MaxKeys<span class="token punctuation">:</span> <span class="token number">10</span> <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword">const</span> data <span class="token operator">=</span> <span class="token keyword">await</span> s3Client<span class="token punctuation">.</span><span class="token function">send</span><span class="token punctuation">(</span>command<span class="token punctuation">)</span><span class="token punctuation">;</span>

    <span class="token keyword">const</span> files <span class="token operator">=</span> data<span class="token punctuation">.</span>Contents <span class="token operator">?</span> data<span class="token punctuation">.</span>Contents<span class="token punctuation">.</span><span class="token function">map</span><span class="token punctuation">(</span>item <span class="token operator">=&gt;</span> <span class="token punctuation">(</span><span class="token punctuation">{</span>
      nome<span class="token punctuation">:</span> item<span class="token punctuation">.</span>Key<span class="token punctuation">,</span>
      tamanho<span class="token punctuation">:</span> <span class="token template-string"><span class="token string">`</span><span class="token interpolation"><span class="token interpolation-punctuation punctuation">${</span><span class="token punctuation">(</span>item<span class="token punctuation">.</span>Size <span class="token operator">/</span> <span class="token number">1024</span><span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token function">toFixed</span><span class="token punctuation">(</span><span class="token number">2</span><span class="token punctuation">)</span><span class="token interpolation-punctuation punctuation">}</span></span><span class="token string"> KB`</span></span><span class="token punctuation">,</span>
      modificado<span class="token punctuation">:</span> item<span class="token punctuation">.</span>LastModified
    <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">:</span> <span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">;</span>

    res<span class="token punctuation">.</span><span class="token function">json</span><span class="token punctuation">(</span><span class="token punctuation">{</span> success<span class="token punctuation">:</span> <span class="token boolean">true</span><span class="token punctuation">,</span> bucket<span class="token punctuation">:</span> BUCKET_NAME<span class="token punctuation">,</span> total_arquivos<span class="token punctuation">:</span> files<span class="token punctuation">.</span>length<span class="token punctuation">,</span> arquivos<span class="token punctuation">:</span> files <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span> <span class="token keyword">catch</span> <span class="token punctuation">(</span><span class="token class-name">error</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    res<span class="token punctuation">.</span><span class="token function">status</span><span class="token punctuation">(</span><span class="token number">500</span><span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token function">json</span><span class="token punctuation">(</span><span class="token punctuation">{</span> success<span class="token punctuation">:</span> <span class="token boolean">false</span><span class="token punctuation">,</span> message<span class="token punctuation">:</span> <span class="token string">'Erro ao listar arquivos do S3'</span><span class="token punctuation">,</span> error<span class="token punctuation">:</span> error<span class="token punctuation">.</span>message <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token comment">// ==========================================</span>
<span class="token comment">// INICIAR SERVIDOR</span>
<span class="token comment">// ==========================================</span>
app<span class="token punctuation">.</span><span class="token function">listen</span><span class="token punctuation">(</span>PORT<span class="token punctuation">,</span> <span class="token string">'0.0.0.0'</span><span class="token punctuation">,</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">=&gt;</span> <span class="token punctuation">{</span>
  console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span><span class="token template-string"><span class="token string">`✅ Servidor rodando na porta </span><span class="token interpolation"><span class="token interpolation-punctuation punctuation">${</span>PORT<span class="token interpolation-punctuation punctuation">}</span></span><span class="token string">`</span></span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span><span class="token template-string"><span class="token string">`🌐 Acesse: http://SEU_IP_PUBLICO:</span><span class="token interpolation"><span class="token interpolation-punctuation punctuation">${</span>PORT<span class="token interpolation-punctuation punctuation">}</span></span><span class="token string">`</span></span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span><span class="token string">'\n📋 Endpoints disponíveis:'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span><span class="token template-string"><span class="token string">`   GET /          - Status do servidor`</span></span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span><span class="token template-string"><span class="token string">`   GET /db/test   - Testar conexão RDS`</span></span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span><span class="token template-string"><span class="token string">`   GET /db/setup  - Criar tabela`</span></span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span><span class="token template-string"><span class="token string">`   GET /db/insert - Inserir usuário`</span></span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span><span class="token template-string"><span class="token string">`   GET /db/list   - Listar usuários`</span></span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span><span class="token template-string"><span class="token string">`   GET /s3/list   - Listar arquivos S3`</span></span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<p>Após substituir os 3 campos no bloco de notas, copie tudo e execute no terminal:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> <span class="token function">nano</span> server.js
</code></pre>
<p>Cole o conteúdo, depois salve: <code>Ctrl + X</code> → <code>Y</code> → <code>Enter</code>.</p>
<h3 id="instalar-dependências-e-iniciar-o-servidor">6.5. Instalar Dependências e Iniciar o Servidor</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># Instalar as dependências do Node.js</span>
<span class="token function">npm</span> <span class="token function">install</span>

<span class="token comment"># Iniciar o servidor</span>
node server.js
</code></pre>
<p>Você deve ver:</p>
<pre><code>✅ Servidor rodando na porta 3000
🌐 Acesse: http://SEU_IP_PUBLICO:3000
</code></pre>
<p>🎉 <strong>O servidor está rodando!</strong></p>
<hr>
<h2 id="testando-a-aplicação">7. Testando a Aplicação</h2>
<blockquote>
<p>⚠️ Não será possível acessar esse endereço na wi-fi da instituição por conta da política de segurança da rede. Use a rede celular ou outra rede externa.</p>
</blockquote>
<h3 id="testes-no-navegador">7.1. Testes no Navegador</h3>
<p>Substitua <code>SEU_IP_PUBLICO</code> pelo IP público da sua instância EC2:</p>

<table>
<thead>
<tr>
<th>#</th>
<th>Endpoint</th>
<th>Descrição</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td><code>http://SEU_IP_PUBLICO:3000</code></td>
<td>Página de status com links</td>
</tr>
<tr>
<td>2</td>
<td><code>http://SEU_IP_PUBLICO:3000/db/test</code></td>
<td>Conexão com RDS (retorna timestamp)</td>
</tr>
<tr>
<td>3</td>
<td><code>http://SEU_IP_PUBLICO:3000/db/setup</code></td>
<td>Cria a tabela <code>usuarios</code></td>
</tr>
<tr>
<td>4</td>
<td><code>http://SEU_IP_PUBLICO:3000/db/insert</code></td>
<td>Insere um novo usuário</td>
</tr>
<tr>
<td>5</td>
<td><code>http://SEU_IP_PUBLICO:3000/db/list</code></td>
<td>Lista todos os usuários</td>
</tr>
<tr>
<td>6</td>
<td><code>http://SEU_IP_PUBLICO:3000/s3/list</code></td>
<td>Lista arquivos do bucket S3</td>
</tr>
</tbody>
</table><h3 id="solução-de-problemas">7.2. Solução de Problemas</h3>
<ul>
<li><strong>Erro de conexão RDS:</strong> Verifique se o endpoint está correto, a senha está correta e o status do RDS está “Disponível”.</li>
<li><strong>Erro no S3:</strong> Verifique se o nome do bucket está correto e se o perfil IAM (<code>LabInstanceProfile</code>) está anexado ao EC2.</li>
<li><strong>Página não carrega:</strong> Verifique o IP público, o Security Group <code>gs-ec2-web</code> (porta 3000) e se o servidor Node.js está rodando.</li>
</ul>
<hr>
<h2 id="acesso-ao-banco-de-dados-via-psql">8. Acesso ao Banco de Dados via psql</h2>
<h3 id="instalação-do-cliente-postgresql">8.1. Instalação do Cliente PostgreSQL</h3>
<p>Abra uma nova conexão EC2 Instance Connect (para não parar o servidor) e execute:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> yum <span class="token function">install</span> -y postgresql15
</code></pre>
<h3 id="conexão-ao-banco">8.2. Conexão ao Banco</h3>
<pre class=" language-bash"><code class="prism  language-bash">psql -h SEU_ENDPOINT_RDS -U postgres -d webappdb
</code></pre>
<blockquote>
<p>⚠️ Substitua <code>SEU_ENDPOINT_RDS</code> pelo endpoint real. Quando solicitado, digite a senha do RDS.</p>
</blockquote>
<h3 id="comandos-úteis-no-psql">8.3. Comandos Úteis no psql</h3>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token comment">-- Listar tabelas</span>
\dt
<span class="token comment">-- Ver estrutura da tabela</span>
\<span class="token number">d</span> usuarios
<span class="token comment">-- Consultar usuários</span>
<span class="token keyword">SELECT</span> <span class="token operator">*</span> <span class="token keyword">FROM</span> usuarios<span class="token punctuation">;</span>
<span class="token comment">-- Contar usuários</span>
<span class="token keyword">SELECT</span> <span class="token function">COUNT</span><span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span> <span class="token keyword">FROM</span> usuarios<span class="token punctuation">;</span>
<span class="token comment">-- Sair do psql</span>
\q
</code></pre>
<hr>
<h2 id="✅-conclusão">✅ Conclusão</h2>
<p>Você criou com sucesso uma infraestrutura web completa na AWS, integrando:</p>
<ul>
<li>✅ <strong>EC2</strong> - Servidor com Node.js</li>
<li>✅ <strong>RDS</strong> - Banco de dados PostgreSQL</li>
<li>✅ <strong>S3</strong> - Armazenamento de objetos</li>
<li>✅ <strong>VPC</strong> - Rede e segurança</li>
</ul>

