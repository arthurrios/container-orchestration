# Project with Docker and Docker Compose

This repository contains a Node.js application connected to a MySQL database, configured using Docker and Docker Compose. The project follows best practices, such as the use of environment variables, volumes for data persistence, custom networks, and multi-stage builds in the Dockerfile.

## Project Overview

This project consists of a Node.js API connected to a MySQL database. The application was developed with a focus on containerization best practices, using Docker and Docker Compose to ensure ease of configuration, scalability, and security.

## Prerequisites

- Docker installed: [https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)
- Docker Compose installed: [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

## Project Structure

- **Dockerfile**: Defines the build process for the Node.js application image.
- **docker-compose.yaml**: Configures the application and database services.
- **.env**: Stores environment variables for secure configuration.

## Configuration

### 1. Clone the Repository

Clone this repository to your local machine:

```bash
git clone https://github.com/your-username/your-repository.git
cd your-repository
```

### 2. Create the `.env` File

Create a `.env` file in the root of the project based on the example below. Make sure to replace the values as needed.

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

**Important:** Add the `.env` file to `.gitignore` to prevent sensitive information from being exposed.

### Security

- The MySQL `root` user is protected with a strong password.
- A specific user (`admin`) was created for the application, with limited permissions.
- Environment variables are used to manage sensitive configurations, avoiding hardcoding in the code.

### 3. Build and Start the Containers

Run the following command to build the images and start the containers:

```bash
docker-compose up --build
```

To run the containers in the background:

```bash
docker-compose up -d
```

### 4. Stop the Containers

To stop the containers:

```bash
docker-compose down
```

## Testing the Connection

### API

The API will be available at `http://localhost:3001`. You can test the connection using tools like Postman or cURL.

Example request using cURL:

```bash
curl http://localhost:3001
```

### Database

The MySQL database will be accessible on port `3306`. You can connect to the database using clients like MySQL Workbench or the `mysql` command:

```bash
mysql -h 127.0.0.1 -P 3306 -u admin -p
```

Enter the password defined in the `.env` file when prompted.

## Data Persistence

Database data is persisted using Docker volumes. This ensures that data is not lost when restarting the containers. The volume is defined in the `docker-compose.yaml` file:

```yaml
volumes:
  db:
```

## Contribution

Contributions are welcome! Open an issue or submit a pull request to improve this project.

## File Structure

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

### .env (Example)

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

## Conclusion

This `README.md` file includes all the necessary steps to configure, run, and test the project. It also provides clear examples of commands and explanations regarding the use of environment variables, volumes, and custom networks.
