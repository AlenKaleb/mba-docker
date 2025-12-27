# Cap 06 - Volumes (exportação anotada)

## Arquivo: `cap 06 - Volumes/02 - Criando volumes pelo volume create/docker-compose.yaml`
```yaml
version: '3'

services:
  db:
    image: mysql:8.0.30-debian
    environment:
      - MYSQL_ROOT_PASSWORD=root
    volumes:
      - my-volume:/var/lib/mysql
  
  nginx:
    image: nginx:1.19.10-alpine
    volumes:
      - my-volume-nginx:/usr/share/nginx/html

volumes:
  my-volume:
    external: true
  my-volume-nginx:
    external: true
```

### Anotações técnicas
- **Volumes externos**: `external: true` indica volumes previamente criados (`docker volume create`).
- **Persistência de dados**: `my-volume` preserva o banco MySQL, mesmo se o container for recriado.
- **Conteúdo estático**: `my-volume-nginx` persiste arquivos servidos pelo Nginx.

## Arquivo: `cap 06 - Volumes/03 - Criando volumes automaticamente/docker-compose.yaml`
```yaml
version: '3'

services:
  db:
    image: mysql:8.0.30-debian
    environment:
      - MYSQL_ROOT_PASSWORD=root
    volumes:
      - ./dbdata:/var/lib/mysql
  
  nginx:
    image: nginx:1.19.10-alpine
    ports:
      - 8080:80
    volumes:
      - ./my-site:/usr/share/nginx/html
```

### Anotações técnicas
- **Bind mounts locais**: diretórios do host (`./dbdata`, `./my-site`) são montados no container.
- **Ideal para dev**: facilita edição do conteúdo sem rebuild.

## Arquivo: `cap 06 - Volumes/03 - Criando volumes automaticamente/docker-compose-volumes-named.yaml`
```yaml
version: '3'

services:
  db:
    image: mysql:8.0.30-debian
    environment:
      - MYSQL_ROOT_PASSWORD=root
    volumes:
      - volume-dbdata:/var/lib/mysql
  
  nginx:
    image: nginx:1.19.10-alpine
    ports:
      - 8080:80
    volumes:
      - volume-nginx:/usr/share/nginx/html

volumes:
  volume-dbdata:
  volume-nginx:
```

### Anotações técnicas
- **Volumes nomeados gerenciados**: o Docker cria volumes automaticamente se não existirem.
- **Boas práticas**: volumes nomeados são mais portáveis que bind mounts em produção.

### Conceitos e boas práticas
- **Separação de dados e runtime**: persistência fora do ciclo de vida do container.
- **Escolha entre bind e named**: bind para dev, named para produção e portabilidade.
