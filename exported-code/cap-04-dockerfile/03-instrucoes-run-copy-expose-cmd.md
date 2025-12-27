# Cap 04 - Instruções RUN, COPY, EXPOSE e CMD (exportação anotada)

## Arquivo: `cap 04 - Dockerfile/03-As instruções RUN, COPY, EXPOSE E CMD/Dockerfile`
```Dockerfile
FROM node:20-slim

RUN apt update && apt install nginx -y

COPY nginx.conf /etc/nginx/nginx.conf

WORKDIR /usr/src/app

COPY index.js package.json ./

RUN npm install

EXPOSE 80

CMD [ "/bin/sh", "-c", "node /usr/src/app/index.js & nginx -g 'daemon off;'" ]

#ENTRYPOINT [ "/bin/sh", "-c", "node /usr/src/app/index.js & nginx -g 'daemon off;'" ]
```

### Anotações técnicas
- **Imagem base do Node**: `node:20-slim` fornece runtime JS com footprint menor.
- **Instalação de pacotes**: `apt install nginx -y` adiciona um servidor web dentro do mesmo container.
- **Configuração do Nginx**: `COPY nginx.conf` customiza o proxy reverso.
- **Contexto de trabalho**: `WORKDIR /usr/src/app` define a raiz para a aplicação.
- **Dependências**: `npm install` instala `express`, preparando o servidor HTTP.
- **Exposição de porta**: `EXPOSE 80` documenta a porta do Nginx, que expõe o app.
- **Dois processos no mesmo container**: `CMD` inicia Node e Nginx em background/foreground. Isso ilustra **anti‑pattern** de múltiplos processos em um único container (em produção, ideal é 1 processo por container, usando sidecars ou compose).

## Arquivo: `cap 04 - Dockerfile/03-As instruções RUN, COPY, EXPOSE E CMD/nginx.conf`
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
- **Nginx como reverse proxy**: encaminha tráfego da porta 80 para a aplicação Node na porta 3000.
- **Topologia**: Nginx atua na borda do container; Node permanece interno.

## Arquivo: `cap 04 - Dockerfile/03-As instruções RUN, COPY, EXPOSE E CMD/index.js`
```js
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`)
})
```

### Anotações técnicas
- **Framework Express**: rota HTTP simples para `GET /`.
- **Separação de camadas**: Node serve a lógica; Nginx serve como proxy (camada de edge).

## Arquivo: `cap 04 - Dockerfile/03-As instruções RUN, COPY, EXPOSE E CMD/package.json`
```json
{
  "description": "",
  "dependencies": {
    "express": "^4.16.4"
  }
}
```

### Anotações técnicas
- **Dependência mínima**: apenas Express. Mantém a imagem mais enxuta e reduz superfície de ataque.

### Conceitos e boas práticas
- **Single responsibility**: em produção, prefira separar Node e Nginx em containers distintos.
- **Portas e contratos**: documente a porta exposta e evite hard‑coding externo; compose/ingress deve mapear.
- **Cache de layers**: separar `COPY package.json` antes de `COPY .` permite aproveitar cache, mas aqui já está no mesmo passo por simplicidade didática.
