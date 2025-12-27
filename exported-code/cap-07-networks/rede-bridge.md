# Cap 07 - Rede bridge (exportação anotada)

## Arquivo: `cap 07 - Networks/02 - Rede bridge/docker-compose.yaml`
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
    # networks:
    #   - my-network
  
  nginx:
    build:
      context: .
      dockerfile: Dockerfile.nginx
    ports:
      - 8080:80
    depends_on:
      - app
    # networks:
    #   - my-network
  
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
    # networks:
    #   - my-network
```

### Anotações técnicas
- **Rede bridge padrão**: sem rede customizada, o Docker cria uma rede bridge automática para o Compose.
- **Exemplo de rede dedicada**: as linhas comentadas mostram como conectar serviços a uma rede específica.
- **Composição em três camadas**: app, proxy, banco.

## Arquivo: `cap 07 - Networks/02 - Rede bridge/Dockerfile`
```Dockerfile
FROM node:20-slim

WORKDIR /usr/src/app

COPY index.js package.json ./

RUN npm install

EXPOSE 3000

CMD [ "node",  "/usr/src/app/index.js" ]
```

## Arquivo: `cap 07 - Networks/02 - Rede bridge/Dockerfile.nginx`
```Dockerfile
FROM nginx:1.19.10-alpine  

COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
```

## Arquivo: `cap 07 - Networks/02 - Rede bridge/nginx.conf`
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
- **Service discovery via DNS**: `app` resolve para o container do serviço Node na rede bridge.

## Arquivo: `cap 07 - Networks/02 - Rede bridge/index.js`
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
- **Host `db`**: aponta para o serviço MySQL dentro da rede bridge.

## Arquivo: `cap 07 - Networks/02 - Rede bridge/package.json`
```json
{
  "description": "",
  "dependencies": {
    "mysql": "^2.18.1"
  }
}
```

### Conceitos e boas práticas
- **Rede isolada**: bridge fornece isolamento por padrão.
- **Nomeação de serviços**: evita dependência de IPs dinâmicos.
- **Healthchecks**: reduzem instabilidades na inicialização.
