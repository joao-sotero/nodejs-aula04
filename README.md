# 🐳 Guia Básico de Docker e Docker Compose

Este guia apresenta os conceitos e comandos fundamentais para trabalhar com **Docker** e **Docker Compose**.  
O conteúdo cobre containers, imagens, volumes, redes, build de imagens e gerenciamento de múltiplos serviços.

---

## 📦 Containers

Containers são instâncias de imagens Docker em execução.  
Eles empacotam uma aplicação e suas dependências em um ambiente isolado.

### 🔍 Comandos Principais

| Comando | Descrição |
|----------|------------|
| `docker ps` | Lista os containers **em execução**. |
| `docker ps -a` | Lista **todos** os containers, inclusive os parados. |
| `docker stop <nome ou id>` | Interrompe a execução de um container. |
| `docker start <nome ou id>` | Inicia um container que foi parado. |
| `docker restart <nome ou id>` | Reinicia um container. |
| `docker rm <nome ou id>` | Remove um container parado. |

---

### 🧩 Flags Importantes no `docker run`

| Flag | Descrição |
|------|------------|
| `-v` | Monta um volume. Ex: `-v /dados:/app/data` |
| `-p` | Mapeia portas entre o host e o container. Ex: `-p 8080:80` |
| `--name` | Define um nome personalizado para o container. |
| `-d` | Executa o container em **modo detached** (em segundo plano). |
| `--rm` | Remove automaticamente o container ao ser parado. |
| `-e` | Define variáveis de ambiente. Ex: `-e NODE_ENV=production` |
| `--network` | Conecta o container a uma rede existente. |

#### 🧠 Exemplo:
```bash
docker run -d -p 8080:80 --name meu-container -e NODE_ENV=production nginx
```

---

## 🖼️ Imagens

As **imagens** são os modelos que o Docker usa para criar containers.

| Comando | Descrição |
|----------|------------|
| `docker image ls` | Lista todas as imagens locais. |
| `docker image rm <id ou nome>` | Remove uma imagem. |
| `docker image pull <imagem>` | Baixa uma imagem do Docker Hub. |

#### 🧠 Exemplo:
```bash
docker image pull node:20-alpine
```

---

## 💾 Volumes

**Volumes** permitem armazenar dados de forma persistente fora do container.

| Comando | Descrição |
|----------|------------|
| `docker volume ls` | Lista todos os volumes. |
| `docker volume create <nome>` | Cria um novo volume. |
| `docker volume rm <nome>` | Remove um volume. |

### 📂 Tipos de Volumes

#### 1. **Volume Nomeado (Named Volume)**
Volume gerenciado pelo Docker, ideal para persistir dados de aplicação.
```bash
docker volume create meus-dados
docker run -d -v meus-dados:/app/data nginx
```

#### 2. **Bind Mount - Diretório Local**
Conecta um diretório do host diretamente ao container.
```bash
# Windows (PowerShell)
docker run -d -v C:\dados:/app/data nginx

# Linux/Mac
docker run -d -v /home/usuario/dados:/app/data nginx
```

#### 3. **Bind Mount - Diretório WSL**
Para usuários Windows com WSL, você pode apontar para diretórios do WSL:
```bash
# Apontando para diretório do WSL Ubuntu
docker run -d -v /mnt/c/Users/joao/projeto:/app nginx

# Ou usando o caminho WSL direto
docker run -d -v \\wsl$\Ubuntu\home\usuario\dados:/app/data nginx

# Exemplo prático com PostgreSQL (como usado em aula)
docker run -d \
  --name postgres-wsl \
  -e POSTGRES_DB=meudb \
  -e POSTGRES_PASSWORD=1234 \
  -v /mnt/c/dados/postgresql:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres
```

#### 4. **Volume Temporário (tmpfs)**
Volume em memória, não persiste após parar o container.
```bash
docker run -d --tmpfs /app/temp nginx
```

### 🧠 Exemplos Práticos

#### Container com Volume Personalizado para Logs
```bash
# Criar volume personalizado
docker volume create app-logs

# Usar em container Node.js
docker run -d \
  --name minha-app \
  -v app-logs:/app/logs \
  -p 3000:3000 \
  node:20-alpine
```

#### Desenvolvimento com Bind Mount
```bash
# Para desenvolvimento, sincronizando código em tempo real
docker run -d \
  --name dev-container \
  -v $(pwd):/app \
  -w /app \
  -p 3000:3000 \
  node:20-alpine \
  npm run dev
```

---

## 🌐 Redes

O Docker utiliza redes virtuais para comunicação entre containers.

| Comando | Descrição |
|----------|------------|
| `docker network ls` | Lista todas as redes disponíveis. |
| `docker network create <nome>` | Cria uma nova rede. |
| `docker network rm <nome>` | Remove uma rede. |
| `docker network connect <rede> <container>` | Conecta um container existente a uma rede existente. |
| `docker network inspect <nome>` | Mostra detalhes da rede (containers, IPs, etc.). |

### 🔌 Tipos de Redes Docker

#### 1. **Bridge (Padrão)**
Rede isolada no host. Containers podem se comunicar entre si por nome.
```bash
# Criar rede bridge personalizada
docker network create --driver bridge minha-rede

# Usar em containers
docker run -d --name web --network minha-rede nginx
docker run -d --name api --network minha-rede node:20-alpine
```

#### 2. **Host**
Container usa diretamente a rede do host (sem isolamento).
```bash
# Container compartilha a rede do host
docker run -d --network host nginx
# Acessa diretamente via localhost na porta 80
```

#### 3. **None**
Container sem acesso à rede (completamente isolado).
```bash
docker run -d --network none alpine sleep 300
```

#### 4. **External (Externa)**
Referencia uma rede criada externamente ao Docker Compose.
```bash
# Criar rede externa primeiro
docker network create aula04

# Usar no docker-compose.yml (como em nossa aula)
networks:
  aula04:
    external: true
```

### 🧠 Exemplos Práticos

#### Comunicação entre Containers
```bash
# Criar rede personalizada
docker network create app-network

# Container 1: API
docker run -d \
  --name api-backend \
  --network app-network \
  -e DATABASE_URL=postgres://user:pass@database:5432/mydb \
  minha-api:latest

# Container 2: Banco de Dados
docker run -d \
  --name database \
  --network app-network \
  -e POSTGRES_DB=mydb \
  -e POSTGRES_USER=user \
  -e POSTGRES_PASSWORD=pass \
  postgres:15
```

#### Conectar Container Existente a Nova Rede
```bash
# Conectar um container já criado a uma rede existente
docker network connect app-network meu-container

# Desconectar da rede
docker network disconnect app-network meu-container
```

#### Exemplo como na Aula (Rede Externa)
```bash
# Primeiro, criar a rede externamente
docker network create aula04

# Depois usar no compose ou containers individuais
docker run -d --name postgres-aula --network aula04 postgres
docker run -d --name pgadmin-aula --network aula04 dpage/pgadmin4
```

---

## 🔎 Inspect

O comando `inspect` é usado para visualizar **detalhes técnicos em formato JSON**, como IPs, volumes, redes, variáveis de ambiente, entre outros.

| Comando | Descrição |
|----------|------------|
| `docker inspect <container ou imagem>` | Mostra detalhes de containers ou imagens. |
| `docker network inspect <nome da rede>` | Mostra informações detalhadas sobre uma rede (containers conectados, IPs, gateway, etc.). |

#### 🧠 Exemplo:
```bash
docker inspect meu-container
docker network inspect minha-rede
```

Esses comandos retornam um JSON com as informações internas do recurso inspecionado.

---

## 💻 Exec

O comando `exec` permite **executar comandos dentro de um container em execução**.

### 🧩 Modos de Uso

| Modo | Descrição |
|------|------------|
| `docker exec -it <container> bash` | Acessa o terminal interativo dentro do container. |
| `docker exec <container> <comando>` | Executa um comando diretamente **sem precisar de terminal interativo**. |

#### 🧠 Exemplos:
```bash
# Entrar no container
docker exec -it meu-container bash

# Executar um comando direto sem abrir terminal
docker exec meu-container ls /app
docker exec meu-container npm run test
```

---

## 🏗️ Build e Dockerfile

O comando `build` é usado para **criar uma imagem personalizada** a partir de um `Dockerfile`.

| Flag | Descrição |
|------|------------|
| `-t` | Define o nome e a tag da imagem (ex: `-t meuapp:1.0`). |
| `-f` | Especifica um nome de arquivo Dockerfile alternativo. |

#### 🧠 Exemplo:
```bash
docker build -t meuapp:1.0 -f Dockerfile .
```

### 🧱 Dockerfile

O **Dockerfile** descreve as etapas de construção de uma imagem.

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]
```

### 🚫 .dockerignore

Define arquivos e pastas que devem ser **ignorados durante o build**.

```
node_modules
.git
.env
```

---

## ⚙️ Docker Compose

O **Docker Compose** permite gerenciar múltiplos containers com um único arquivo `docker-compose.yml`.

### 🔍 Comandos Principais

| Comando | Descrição |
|----------|------------|
| `docker compose up` | Sobe os containers definidos no arquivo Compose. |
| `docker compose down` | Para e remove containers, redes e volumes criados. |
| `docker compose restart` | Reinicia os serviços. |
| `docker compose start` | Inicia serviços já criados. |
| `docker compose stop` | Para os serviços sem removê-los. |
| `docker compose logs <serviço>` | Mostra os logs de um serviço específico. |
| `docker compose exec <serviço> <comando>` | Executa comando dentro do container do serviço. |

### 🧩 Flags Importantes

| Flag | Descrição |
|------|------------|
| `-d` | Executa em **modo detached** (em segundo plano). |
| `-f` | Especifica o caminho do arquivo Compose. |
| `--build` | Força o rebuild das imagens antes de subir. |

#### 🧠 Exemplo:
```bash
docker compose -f ./meu-compose.yml up -d --build
```

### 📖 Anatomia do docker-compose.yml da Aula

Vamos analisar **linha por linha** o arquivo usado em nossa aula:

```yaml
services:                                    # Define todos os serviços (containers)
  pgadmin:                                   # Nome do primeiro serviço
    image: dpage/pgadmin4                    # Imagem Docker oficial do pgAdmin4
    container_name: pgadmin_container-teste  # Nome personalizado do container
    environment:                             # Variáveis de ambiente do container
      PGADMIN_DEFAULT_EMAIL: user@domain.com # Email padrão para login no pgAdmin
      PGADMIN_DEFAULT_PASSWORD: 1234         # Senha padrão para login no pgAdmin
    ports:                                   # Mapeamento de portas
      - "5050:80"                           # Porta 5050 do host → porta 80 do container
    networks:                               # Redes que este container usará
      - aula04                              # Conecta à rede externa 'aula04'

  postgres:                                 # Nome do segundo serviço
    image: postgres                         # Imagem oficial do PostgreSQL (latest)
    container_name: postgresDB-teste        # Nome personalizado do container
    ports:                                  # Mapeamento de portas
      - "5432:5432"                        # Porta padrão do PostgreSQL
    environment:                            # Configurações do banco
      POSTGRES_DB: aula04                   # Nome do banco que será criado
      POSTGRES_PASSWORD: 1234               # Senha do usuário 'postgres'
    volumes:                                # Montagem de volumes
      - /pgdata:/var/lib/postgresql/data    # Bind mount: diretório host → dados do PG
    networks:                               # Redes que este container usará
      - aula04                              # Conecta à rede externa 'aula04'

networks:                                   # Definição de redes
  aula04:                                   # Nome da rede
    external: true                          # Indica que a rede já existe externamente
```

### 🔧 Estrutura Detalhada do Compose

#### **services:** (obrigatório)
Define todos os containers que serão gerenciados.

#### **Propriedades de cada serviço:**

| Propriedade | Descrição | Exemplo |
|-------------|-----------|---------|
| `image` | Imagem Docker a ser usada | `postgres:15` |
| `build` | Caminho para Dockerfile (alternativa ao image) | `build: ./app` |
| `container_name` | Nome personalizado do container | `meu-postgres` |
| `ports` | Mapeamento porta_host:porta_container | `"3000:3000"` |
| `environment` | Variáveis de ambiente | `NODE_ENV: production` |
| `env_file` | Arquivo .env com variáveis de ambiente | `.env` |
| `volumes` | Volumes e bind mounts | `- ./data:/app/data` |
| `networks` | Redes que o container participará | `- frontend` |
| `depends_on` | Dependências entre serviços | `- database` |
| `restart` | Política de restart | `unless-stopped` |

### 🔐 Variáveis de Ambiente com Arquivo .env

O Docker Compose permite carregar variáveis de ambiente através de arquivos `.env`, facilitando o gerenciamento de configurações sensíveis.

#### **Estrutura do arquivo .env**
```bash
# Arquivo .env na raiz do projeto
NODE_ENV=development
DB_HOST=postgres
DB_PORT=5432
DB_NAME=meuapp
DB_USER=usuario
DB_PASSWORD=senha123
POSTGRES_PASSWORD=senha123
PGADMIN_EMAIL=admin@exemplo.com
PGADMIN_PASSWORD=admin123
```

#### **Usando no docker-compose.yml**
```yaml
services:
  app:
    image: node:20-alpine
    env_file:
      - .env                    # Carrega todas as variáveis do .env
    environment:
      - NODE_ENV=${NODE_ENV}    # Ou usar interpolação específica
    ports:
      - "3000:3000"

  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=${DB_NAME}
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    ports:
      - "${DB_PORT}:5432"       # Usar variável para porta também
```

#### **Boas Práticas com .env**
```bash
# 1. Criar .env.example (template público)
NODE_ENV=development
DB_HOST=localhost
DB_PORT=5432
DB_NAME=nome_do_banco
DB_USER=seu_usuario
DB_PASSWORD=sua_senha

# 2. Adicionar .env ao .gitignore
echo ".env" >> .gitignore

# 3. Usar diferentes arquivos por ambiente
docker compose --env-file .env.development up
docker compose --env-file .env.production up -d
```

#### **Exemplo Prático Completo**
**.env:**
```bash
# Configurações da Aplicação
NODE_ENV=development
APP_PORT=3000

# PostgreSQL
POSTGRES_DB=aula04
POSTGRES_USER=postgres
POSTGRES_PASSWORD=1234
DB_PORT=5432

# pgAdmin
PGADMIN_EMAIL=user@domain.com
PGADMIN_PASSWORD=1234
PGADMIN_PORT=5050
```

**docker-compose.yml:**
```yaml
services:
  app:
    build: .
    env_file: .env
    ports:
      - "${APP_PORT}:3000"
    environment:
      - DATABASE_URL=postgres://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:${DB_PORT}/${POSTGRES_DB}

  postgres:
    image: postgres
    env_file: .env
    ports:
      - "${DB_PORT}:5432"

  pgadmin:
    image: dpage/pgadmin4
    env_file: .env
    environment:
      - PGADMIN_DEFAULT_EMAIL=${PGADMIN_EMAIL}
      - PGADMIN_DEFAULT_PASSWORD=${PGADMIN_PASSWORD}
    ports:
      - "${PGLADMIN_PORT}:80"
```

### 🧠 Exemplos Avançados

#### Compose Completo com Build Personalizado
```yaml
services:
  app:
    build:                              # Build personalizado
      context: ./app                    # Diretório com Dockerfile
      dockerfile: Dockerfile.prod       # Dockerfile específico
    container_name: minha-app
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://postgres:1234@db:5432/myapp
    ports:
      - "3000:3000"
    volumes:
      - ./logs:/app/logs               # Bind mount para logs
      - app_data:/app/data             # Volume nomeado
    depends_on:                        # Aguarda o banco subir primeiro
      - db
    restart: unless-stopped            # Reinicia automaticamente
    networks:
      - app-network

  db:
    image: postgres:15
    container_name: postgres-prod
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: 1234
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - app-network

volumes:                              # Definir volumes nomeados
  postgres_data:                      # Volume para dados do PostgreSQL
  app_data:                          # Volume para dados da aplicação

networks:                            # Definir redes personalizadas
  app-network:                       # Rede bridge personalizada
    driver: bridge
```

#### Usando o Compose da Aula
```bash
# Primeiro, criar a rede externa
docker network create aula04

# Subir os serviços
docker compose up -d

# Verificar status
docker compose ps

# Ver logs do PostgreSQL
docker compose logs postgres

# Entrar no container do PostgreSQL
docker compose exec postgres psql -U postgres -d aula04

# Parar sem remover
docker compose stop

# Parar e remover tudo
docker compose down
```

### 🛡️ Segurança com Variáveis de Ambiente

#### **⚠️ O que NUNCA fazer:**
```yaml
# ❌ NUNCA hardcode senhas no docker-compose.yml
services:
  postgres:
    environment:
      - POSTGRES_PASSWORD=minhasenha123  # ❌ Senha exposta no código
```

#### **✅ Boas Práticas:**
```bash
# 1. Use .env para desenvolvimento
# .env
POSTGRES_PASSWORD=senha_dev_123

# 2. Use Docker Secrets para produção
# docker-compose.prod.yml
services:
  postgres:
    secrets:
      - postgres_password
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/postgres_password

secrets:
  postgres_password:
    file: ./secrets/postgres_password.txt

# 3. Use variáveis do sistema em produção
export POSTGRES_PASSWORD="senha_super_segura"
docker compose up -d
```

#### **🔍 Validação de Variáveis**
```yaml
# docker-compose.yml
services:
  app:
    image: node:20-alpine
    environment:
      - NODE_ENV=${NODE_ENV:-development}           # Valor padrão
      - DB_HOST=${DB_HOST:?Database host required}  # Obrigatória
      - API_KEY=${API_KEY:?API key is required}     # Erro se não definida
```

#### **📂 Estrutura Recomendada**
```
projeto/
├── .env.example          # Template público (no git)
├── .env                 # Configurações locais (no .gitignore)
├── .env.development     # Ambiente de desenvolvimento
├── .env.staging         # Ambiente de homologação  
├── .env.production      # Ambiente de produção
├── docker-compose.yml   # Configuração base
├── docker-compose.dev.yml   # Override para desenvolvimento
└── docker-compose.prod.yml  # Override para produção
```

---

## 🧰 Exemplos Completos

### 1. Exemplo Básico (Web + Banco)
```yaml
version: "3.9"
services:
  web:
    image: nginx
    ports:
      - "8080:80"
    networks:
      - webnet
  
  db:
    image: mysql:8
    environment:
      - MYSQL_ROOT_PASSWORD=12345
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - webnet

volumes:
  dbdata:

networks:
  webnet:
    driver: bridge
```

### 2. Exemplo da Aula (PostgreSQL + pgAdmin)
```yaml
services:
  pgadmin:
    image: dpage/pgadmin4
    container_name: pgadmin_container-teste
    environment:
      PGADMIN_DEFAULT_EMAIL: user@domain.com
      PGADMIN_DEFAULT_PASSWORD: 1234
    ports:
      - "5050:80"
    networks:
      - aula04

  postgres:
    image: postgres
    container_name: postgresDB-teste
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: aula04
      POSTGRES_PASSWORD: 1234
    volumes:
      - /pgdata:/var/lib/postgresql/data
    networks:
      - aula04

networks:
  aula04:
    external: true
```

### 3. Exemplo com Aplicação Node.js
```yaml
services:
  app:
    build: .
    container_name: node-app
    environment:
      - NODE_ENV=development
      - DB_HOST=postgres
    ports:
      - "3000:3000"
    volumes:
      - .:/app                          # Sync código para desenvolvimento
      - /app/node_modules               # Evita conflito com node_modules
    depends_on:
      - postgres
    networks:
      - app-network

  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: nodeapp
      POSTGRES_USER: developer
      POSTGRES_PASSWORD: dev123
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - app-network

volumes:
  postgres_data:

networks:
  app-network:
    driver: bridge
```

### Comandos para Gerenciar os Ambientes

#### Ambiente da Aula:
```bash
# Criar rede externa primeiro
docker network create aula04

# Subir ambiente
docker compose up -d

# Verificar containers
docker compose ps

# Acessar pgAdmin: http://localhost:5050
# Acessar PostgreSQL: localhost:5432
```

#### Ambiente de Desenvolvimento:
```bash
# Subir com rebuild
docker compose up -d --build

# Ver logs em tempo real
docker compose logs -f app

# Executar comandos no container
docker compose exec app npm install
docker compose exec app npm run test
```

#### Parar Ambientes:
```bash
# Parar sem remover
docker compose stop

# Parar e remover tudo
docker compose down

# Parar e remover + volumes
docker compose down -v
```

---

## 🧼 Limpeza

| Comando | Descrição |
|----------|------------|
| `docker system prune` | Remove containers, imagens e redes não utilizadas. |
| `docker volume prune` | Remove volumes não utilizados. |

---

## 🎯 Dica Final

> Sempre leia e entenda cada comando antes de executá-lo.  
> O Docker é uma ferramenta poderosa, mas pode apagar dados ou ambientes importantes se usado sem atenção.

---

📘 **Autor:** João Sotero  
🧑‍💻 **Curso:** Fundamentos de Docker e Docker Compose
