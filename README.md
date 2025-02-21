# Projeto com Docker e Docker Compose

Este repositório contém uma aplicação Node.js conectada a um banco de dados MySQL, configurada usando Docker e Docker Compose. O projeto segue boas práticas, como o uso de variáveis de ambiente, volumes para persistência de dados, redes customizadas e múltiplos estágios no Dockerfile.

## Visão Geral do Projeto

Este projeto consiste em uma API Node.js conectada a um banco de dados MySQL. A aplicação foi desenvolvida com foco em boas práticas de containerização, utilizando Docker e Docker Compose para garantir facilidade de configuração, escalabilidade e segurança.

## Pré-requisitos

- Docker instalado: [https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)
- Docker Compose instalado: [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

## Estrutura do Projeto

- **Dockerfile**: Define a construção da imagem da aplicação Node.js.
- **docker-compose.yaml**: Configura os serviços da aplicação e do banco de dados.
- **.env**: Armazena variáveis de ambiente para configuração segura.

## Configuração

### 1. Clone o Repositório

Clone este repositório para sua máquina local:

```bash
git clone https://github.com/seu-usuario/seu-repositorio.git
cd seu-repositorio
```

### 2. Crie o Arquivo `.env`

Crie um arquivo `.env` na raiz do projeto com base no exemplo abaixo. Certifique-se de substituir os valores conforme necessário.

```env
# MySQL Configuration
MYSQL_ROOT_PASSWORD=rootpassword
MYSQL_DATABASE=rocketseat-db
MYSQL_USER=admin
MYSQL_PASSWORD=adminpassword
MYSQL_PORT=3306

# API Configuration
API_PORT=3001
```

**Importante:** Adicione o arquivo `.env` ao `.gitignore` para evitar que informações sensíveis sejam expostas.

### Segurança

- O usuário `root` do MySQL é protegido com uma senha forte.
- Um usuário específico (`admin`) foi criado para a aplicação, com permissões limitadas.
- Variáveis de ambiente são usadas para gerenciar configurações sensíveis, evitando hardcoding no código.

### 3. Construa e Inicie os Containers

Execute o seguinte comando para construir as imagens e iniciar os containers:

```bash
docker-compose up --build
```

Para executar os containers em segundo plano:

```bash
docker-compose up -d
```

### 4. Parar os Containers

Para parar os containers:

```bash
docker-compose down
```

## Testando a Conexão

### API

A API estará disponível em `http://localhost:3001`. Você pode testar a conexão usando ferramentas como Postman ou cURL.

Exemplo de requisição usando cURL:

```bash
curl http://localhost:3001
```

### Banco de Dados

O banco de dados MySQL estará acessível na porta `3306`. Você pode conectar-se ao banco de dados usando clientes como MySQL Workbench ou o comando `mysql`:

```bash
mysql -h 127.0.0.1 -P 3306 -u admin -p
```

Insira a senha definida no arquivo `.env` quando solicitado.

## Persistência de Dados

Os dados do banco de dados são persistidos usando volumes Docker. Isso garante que os dados não sejam perdidos ao reiniciar os containers. O volume é definido no arquivo `docker-compose.yaml`:

```yaml
volumes:
  db:
```

## Contribuição

Contribuições são bem-vindas! Abra uma issue ou envie um pull request para melhorar este projeto.

## Estrutura dos Arquivos

### Dockerfile

```dockerfile
# Stage 1: Build
FROM node:18 AS build
WORKDIR /usr/src/app
RUN npm install -g pnpm
COPY package.json pnpm-lock.yaml ./
RUN pnpm install
COPY . .
RUN pnpm run build
RUN pnpm install --prod --frozen-lockfile && pnpm store prune

# Stage 2: Runtime
FROM node:18-alpine3.21
WORKDIR /usr/src/app
COPY --from=build /usr/src/app/package.json ./package.json
COPY --from=build /usr/src/app/dist ./dist
COPY --from=build /usr/src/app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/main"]
```

### docker-compose.yaml

```yaml
services:
  mysql:
    image: mysql:8
    container_name: mysql
    volumes: 
      - db:/var/lib/mysql
    ports: 
      - "${MYSQL_PORT}:3306"
    environment:
      MYSQL_ROOT_PASSWORD: "${MYSQL_ROOT_PASSWORD}"
      MYSQL_DATABASE: "${MYSQL_DATABASE}"
      MYSQL_USER: "${MYSQL_USER}"
      MYSQL_PASSWORD: "${MYSQL_PASSWORD}"
    networks:
      - first-network

  api-rocket:
    build:
      context: .
    container_name: api-rocket
    ports:
      - "${API_PORT}:3000"
    environment:
      DB_HOST: mysql
      DB_PORT: "${MYSQL_PORT}"
      DB_USER: "${MYSQL_USER}"
      DB_PASSWORD: "${MYSQL_PASSWORD}"
      DB_NAME: "${MYSQL_DATABASE}"
    depends_on:
      - mysql
    networks:
      - first-network

networks:
  first-network:
    driver: bridge

volumes:
  db:
```

### .env (Exemplo)

```env
# MySQL Configuration
MYSQL_ROOT_PASSWORD=rootpassword
MYSQL_DATABASE=rocketseat-db
MYSQL_USER=admin
MYSQL_PASSWORD=adminpassword
MYSQL_PORT=3306

# API Configuration
API_PORT=3001
```

## Conclusão

Este arquivo `README.md` inclui todas as etapas necessárias para configurar, executar e testar o projeto. Ele também fornece exemplos claros de comandos e explicações sobre o uso de variáveis de ambiente, volumes e redes personalizadas.