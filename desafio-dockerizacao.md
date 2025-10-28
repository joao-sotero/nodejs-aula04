# 🚀 Desafio: Dockerização da Aplicação Node.js

## 📋 Objetivo

Dockerizar a aplicação Node.js da **Aula 03** utilizando **Docker** e **Docker Compose** com **bind mount** para desenvolvimento em tempo real.

---

## 🎯 O que você precisa fazer:

1. ✅ Criar um **Dockerfile** para a aplicação Node.js
2. ✅ Criar um **.dockerignore** para otimizar o build
3. ✅ Criar um **docker-compose.yml** com bind mount
4. ✅ Configurar o ambiente da aplicação
5. ✅ Testar a aplicação funcionando em container

---

## 📁 Estrutura Esperada do Projeto

```
projeto-aula03/
├── src/
│   ├── controllers/
│   ├── models/
│   ├── routes/
│   └── app.js
├── package.json
├── package-lock.json
├── Dockerfile          ← ✨ Você vai criar
├── .dockerignore       ← ✨ Você vai criar
└── docker-compose.yml  ← ✨ Você vai criar
```

---

## 🐳 Passo 1: Criar o Dockerfile

Crie um arquivo chamado `Dockerfile` na raiz do projeto:

```dockerfile
# 🚨 COMPLETE O DOCKERFILE ABAIXO

FROM node:____-alpine

# Definir diretório de trabalho
WORKDIR ____

# Copiar arquivos de dependência
COPY ____ ____
COPY ____ ____

# Instalar dependências
RUN ____

# Copiar código da aplicação
COPY ____ ____

# Expor porta da aplicação
EXPOSE ____

# Comando para iniciar a aplicação
CMD ["____", "____"]
```

### 💡 Dicas para o Dockerfile:
- Use `node:20-alpine` (imagem leve)
- Diretório de trabalho: `/home/node/app`
- Primeiro copie apenas `package*.json` para otimizar cache
- Depois copie o resto do código com `COPY . .`
- A aplicação roda na porta `3000`
- Use `npm start` para iniciar

---

## 🚫 Passo 2: Criar o .dockerignore

Crie um arquivo `.dockerignore` para excluir arquivos desnecessários:

```
# 🚨 COMPLETE ABAIXO - O QUE DEVE SER IGNORADO?

node_modules
____
____
____
____
.git
README.md
```

### 💡 O que ignorar:
- `node_modules` (será instalado no container)
- Arquivos de ambiente (`.env`)
- Logs (`*.log`)
- Arquivos temporários (`*.tmp`)
- Cache do npm (`.npm`)

---

## 🐙 Passo 3: Criar o docker-compose.yml

Crie um `docker-compose.yml` com **bind mount** para desenvolvimento:

```yaml
services:
  # 🚨 COMPLETE A CONFIGURAÇÃO DA APLICAÇÃO
  app:
    build: ____              # Caminho para o Dockerfile
    container_name: ____     # Nome do container da aplicação
    environment:
      - NODE_ENV=____        # Ambiente de desenvolvimento
      - PORT=____            # Porta da aplicação
    ports:
      - "____:____"          # Porta da aplicação (3000:3000)
    volumes:
      - ____:____            # 🔥 BIND MOUNT: código local → container
      - ____                 # Evitar conflito com node_modules
    restart: ____            # Política de restart
```

### 💡 Dicas para o Compose:
- **App:** 
  - Build: `.` (diretório atual)
  - Container name: `node-app-dev`
  - Environment: `development`
  - Bind mount: `./:/home/node/app` 
  - Excluir node_modules: `/home/node/app/node_modules`
  - Porta: `3000:3000`
  - Restart: `unless-stopped`

---

## ⚡ Passo 4: Comandos para Executar

### 1. **Build e Start dos Containers**
```bash
# Fazer build e subir o ambiente
docker compose up --build -d

# Verificar se está rodando
docker compose ps
```

### 2. **Durante Desenvolvimento**
```bash
# Ver logs da aplicação em tempo real
docker compose logs -f app

# Executar comandos no container
docker compose exec app npm install nova-dependencia
docker compose exec app npm run test
```

### 3. **Testar Bind Mount**
```bash
# Modificar código no VS Code
# Salvar arquivo
# Ver mudanças refletidas automaticamente no container (sem rebuild)
```

### 4. **Gerenciar Ambiente**
```bash
# Parar containers
docker compose stop

# Reiniciar apenas a aplicação
docker compose restart app

# Parar e remover tudo
docker compose down

# Reset completo (remove volumes também)
docker compose down -v
```

---

## 🧪 Passo 4: Testar a Aplicação

### ✅ Checklist de Teste:

1. **Aplicação Node.js:** http://localhost:3000
2. **Modificar código** → Salvar → Ver mudança sem rebuild (hot reload)
3. **Logs funcionando:** `docker compose logs app`
4. **Container ativo:** `docker compose ps`
5. **Bind mount funcionando:** Editar arquivo local e ver reflexo no container

---

## 🎖️ Critérios de Avaliação

### ⭐ **Básico :**
- [ ] Dockerfile funcionando corretamente
- [ ] .dockerignore com itens essenciais
- [ ] docker-compose.yml com o serviço da aplicação
- [ ] Aplicação acessível via browser

### ⭐⭐ **Intermediário**
- [ ] Bind mount configurado corretamente
- [ ] Hot reload funcionando (mudanças sem rebuild)
- [ ] Variáveis de ambiente bem configuradas
- [ ] Otimização do build com cache de dependências

### ⭐⭐⭐ **Avançado:**
- [ ] Dockerfile otimizado (multi-stage build, usuário não-root)
- [ ] .dockerignore completo e bem estruturado
- [ ] Health checks na aplicação
- [ ] Documentação completa de como executar
- [ ] Configurações de desenvolvimento vs produção

---

## 🚨 Problemas Comuns e Soluções

### 🐛 **Aplicação não inicia**
```bash
# Verificar se a porta está correta
EXPOSE 3000              # No Dockerfile
ports: ["3000:3000"]     # No docker-compose.yml

# Verificar se o comando de start está correto
CMD ["npm", "start"]     # No Dockerfile
```

### 🐛 **Bind mount não funciona**
```bash
# Verificar caminho correto
volumes:
  - ./:/home/node/app              # ✅ Diretório atual → /home/node/app
  - /home/node/app/node_modules    # ✅ Preserva node_modules do container
```

### 🐛 **Mudanças não aparecem automaticamente**
```bash
# Usar ferramenta de desenvolvimento com watch
CMD ["npm", "run", "dev"]  # Com nodemon ou similar

# Ou usar node --watch (Node.js 18+)
CMD ["node", "--watch", "/home/node/app/src/app.js"]

# Verificar se nodemon está configurado no package.json
"scripts": {
  "dev": "nodemon src/app.js",
  "start": "node src/app.js"
}
```

### 🐛 **Permissões no container**
```bash
# Se tiver problemas de permissão, usar usuário node
USER node
WORKDIR /home/node/app

# Ou ajustar ownership no build
COPY --chown=node:node . .
```

---

## 📚 Recursos Extras

### 🔗 **Links Úteis:**
- [Dockerfile Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Node.js Docker Guide](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)

### 📖 **Comandos de Debug:**
```bash
# Entrar no container para debug
docker compose exec app sh

# Verificar se a aplicação está rodando na porta correta
docker compose exec app netstat -tulpn

# Verificar variáveis de ambiente
docker compose exec app env

# Testar se a aplicação responde internamente
docker compose exec app curl http://localhost:3000
```

---

## 🎉 Entrega

### 📤 **Como entregar:**
1. **Código completo** com os 3 arquivos criados
2. **Screenshot** da aplicação rodando no browser
3. **Comando** `docker compose ps` mostrando containers ativos
4. **Explicação** de como o bind mount facilita o desenvolvimento

### 📅 **Prazo:** Uma semana a partir da data da aula

---

## 🏆 Bonus Challenge

Para quem quer ir além:

1. **Multi-stage Dockerfile** (build stage + production stage)
2. **Health checks** na aplicação
3. **Usuário não-root** no container (segurança)
4. **Diferentes ambientes** (development, production)
5. **Docker build com argumentos** (ARG no Dockerfile)

---

**💪 Boa sorte e mãos à obra!** 

Lembre-se: **Docker** facilita o desenvolvimento, mas requer prática. Use a documentação da **Aula 04** como referência e não hesite em experimentar! 🚀