# Cap 07 - Rede host (exportação anotada)

## Arquivo: `cap 07 - Networks/03 - Rede host/docker-compose.yaml`
```yaml
version: "3"

services:
  # localhost:3000
  app:
    build: .
    restart: always
    depends_on:
      db:
        condition: service_healthy
    deploy:
      resources:
        limits:
          cpus: "0.1"
          memory: 50M

  # localhost:3306
  db:
    image: mysql:8.0.30-debian
    command: --default-authentication-plugin=mysql_native_password
    environment:
      - MYSQL_ROOT_PASSWORD=root
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 20s
      timeout: 5s
      start_period: 5s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: "0.3"
          memory: 1G
```

### Anotações técnicas
- **Rede host (conceito)**: embora o arquivo não configure explicitamente `network_mode: host`, a anotação indica que os serviços pretendem usar `localhost` (host network). Em host network, as portas do container são as mesmas do host.
- **Dependência com healthcheck**: mesma estratégia de inicialização segura do DB.

## Arquivo: `cap 07 - Networks/03 - Rede host/Dockerfile`
```Dockerfile
FROM node:20-slim

WORKDIR /usr/src/app

COPY index.js package.json ./

RUN npm install

EXPOSE 3000

CMD [ "node",  "/usr/src/app/index.js" ]
```

## Arquivo: `cap 07 - Networks/03 - Rede host/Dockerfile.nginx`
```Dockerfile
FROM nginx:1.19.10-alpine  

COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
```

## Arquivo: `cap 07 - Networks/03 - Rede host/nginx.conf`
```nginx
events {
  worker_connections 10;
}

http {
  server {
    listen 80;

    location / {
      proxy_pass http://localhost:3000;
    }
  }
}
```

### Anotações técnicas
- **Proxy para `localhost`**: em modo host, o Nginx se conecta ao Node via loopback do host.

## Arquivo: `cap 07 - Networks/03 - Rede host/index.js`
```js
const mysql = require('mysql');
const http = require('http');

const connection = mysql.createConnection({
  host: 'localhost',
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
- **Conexão via `localhost`**: com host network, o MySQL está exposto na porta 3306 do host.

## Arquivo: `cap 07 - Networks/03 - Rede host/package.json`
```json
{
  "description": "",
  "dependencies": {
    "mysql": "^2.18.1"
  }
}
```

### Conceitos e boas práticas
- **Host network**: reduz isolamento e pode causar conflitos de porta; use com cautela.
- **Preferência por bridge**: em geral, redes bridge oferecem melhor isolamento e portabilidade.
