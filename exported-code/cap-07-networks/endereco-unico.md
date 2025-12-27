# Cap 07 - Conectando containers via um endereço único (exportação anotada)

## Arquivo: `cap 07 - Networks/05 - Conectando containers via um endereço único/docker-compose-node.yaml`
```yaml
version: '3'

services:
  app:
    build: .
    restart: always
    ports:
      - 3000:3000
    deploy:
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    extra_hosts:
       - "host.docker.internal:host-gateway"
```

### Anotações técnicas
- **`extra_hosts`**: cria alias para `host.docker.internal`, permitindo o container acessar serviços no host.
- **Mapeamento de porta**: expõe a aplicação diretamente para o host.

## Arquivo: `cap 07 - Networks/05 - Conectando containers via um endereço único/docker-compose-mysql.yaml`
```yaml
version: "3"

services:
  db:
    image: mysql:8.0.30-debian
    command: --default-authentication-plugin=mysql_native_password
    environment:
      - MYSQL_ROOT_PASSWORD=root
    ports:
      - 3306:3306
    deploy:
      resources:
        limits:
          cpus: "0.3"
          memory: 1G
```

### Anotações técnicas
- **Banco exposto no host**: mapeia 3306 do container para o host.

## Arquivo: `cap 07 - Networks/05 - Conectando containers via um endereço único/Dockerfile`
```Dockerfile
FROM node:20-slim

WORKDIR /usr/src/app

COPY index.js package.json ./

RUN npm install

EXPOSE 3000

CMD [ "node",  "/usr/src/app/index.js" ]
```

## Arquivo: `cap 07 - Networks/05 - Conectando containers via um endereço único/index.js`
```js
const mysql = require('mysql');
const http = require('http');

const connection = mysql.createConnection({
  host: 'host.docker.internal',
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
- **Host gateway**: `host.docker.internal` resolve para o host, permitindo acesso ao DB exposto.

## Arquivo: `cap 07 - Networks/05 - Conectando containers via um endereço único/package.json`
```json
{
  "description": "",
  "dependencies": {
    "mysql": "^2.18.1"
  }
}
```

### Conceitos e boas práticas
- **Acesso ao host**: útil em dev quando serviços rodam fora do Compose.
- **Portabilidade**: evite depender do host em produção; prefira redes internas.
