# Cap 05 - Primeiro exemplo com Docker Compose (exportação anotada)

## Arquivo: `cap 05 - Docker Compose/03 - Primeiro exemplo com Docker Compose/docker-compose.yaml`
```yaml
version: '3'

services:
  app:
    build: .
    restart: always
    depends_on:
      db:
        condition: service_healthy
    # ports:
    #   - 8080:3000
    deploy:
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
  
  nginx:
    build:
      context: .
      dockerfile: Dockerfile.nginx
    ports:
      - 8080:80
    depends_on:
      - app
  
  db:
    image: mysql:8.0.30-debian
    command: --default-authentication-plugin=mysql_native_password
    environment:
      - MYSQL_ROOT_PASSWORD=root
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 1m30s
      timeout: 10s
      start_period: 10s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: "0.3"
          memory: 1G
```

### Anotações técnicas
- **Três serviços**: `app` (Node), `nginx` (proxy), `db` (MySQL). Arquitetura em camadas clássica.
- **`depends_on` com healthcheck**: garante que o app só suba quando o banco estiver saudável.
- **Limites de recursos**: `deploy.resources` ilustra *resource governance* (útil em Swarm).
- **Portas**: Nginx publica `8080:80`, expondo o proxy para o host. A porta do app é interna.

## Arquivo: `cap 05 - Docker Compose/03 - Primeiro exemplo com Docker Compose/Dockerfile`
```Dockerfile
FROM node:20-slim

WORKDIR /usr/src/app

COPY index.js package.json ./

RUN npm install

EXPOSE 3000

CMD [ "node",  "/usr/src/app/index.js" ]
```

### Anotações técnicas
- **Build do app**: instala dependências e expõe porta 3000 para comunicação interna no Compose.

## Arquivo: `cap 05 - Docker Compose/03 - Primeiro exemplo com Docker Compose/Dockerfile.nginx`
```Dockerfile
FROM nginx:1.19.10-alpine  

COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
```

### Anotações técnicas
- **Imagem enxuta** e configuração do Nginx via `nginx.conf` customizado.

## Arquivo: `cap 05 - Docker Compose/03 - Primeiro exemplo com Docker Compose/nginx.conf`
```nginx
events {
  worker_connections 10;
}

http {
  server {
    listen 80;

    location / {
      proxy_pass http://app:3000;
    }
  }
}
```

### Anotações técnicas
- **Service discovery interno**: `proxy_pass http://app:3000` usa o nome do serviço do Compose como DNS interno.

## Arquivo: `cap 05 - Docker Compose/03 - Primeiro exemplo com Docker Compose/index.js`
```js
const mysql = require('mysql');
const http = require('http');

const connection = mysql.createConnection({
  host: 'db',
  user: 'root',
  password: 'root',
  database: 'mysql'
});

connection.connect((err) => {
  if (err) {
    console.error('Erro ao conectar ao banco de dados:', err);
    throw err;
  }
  console.log('Conectado ao banco de dados MySQL');
});

http.createServer(function (req, res) {
  res.write('Hello World!');
  res.end();
}).listen(3000);
```

### Anotações técnicas
- **Conexão com MySQL via DNS do Compose**: `host: 'db'` resolve para o serviço `db`.
- **Server HTTP nativo**: demonstra uma resposta simples para validação da pilha.

## Arquivo: `cap 05 - Docker Compose/03 - Primeiro exemplo com Docker Compose/package.json`
```json
{
  "description": "",
  "dependencies": {
    "mysql": "^2.18.1"
  }
}
```

### Anotações técnicas
- **Dependência do driver**: o pacote `mysql` permite conectar ao serviço MySQL.

### Conceitos e boas práticas
- **Separação de responsabilidades**: banco, app e proxy em containers distintos.
- **Service discovery**: nomes de serviços no Compose substituem IPs fixos.
- **Healthcheck**: evita corrida na inicialização e reduz falhas de conexão.
