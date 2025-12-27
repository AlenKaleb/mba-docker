# Cap 04 - Build de uma nova imagem (exportação anotada)

## Arquivo: `cap 04 - Dockerfile/01-Build de uma nova imagem/Dockerfile`
```Dockerfile
FROM nginx:1.19.10-alpine

COPY index.html /usr/share/nginx/html
```

### Anotações técnicas
- **Base image enxuta**: `nginx:1.19.10-alpine` usa Alpine Linux para reduzir tamanho e superfície de ataque, comum em imagens de produção.
- **COPY direto do conteúdo estático**: copia `index.html` para o diretório padrão do Nginx (`/usr/share/nginx/html`). Isso caracteriza uma arquitetura de **static site container** (aplicação só de arquivos estáticos).

### Conceitos e boas práticas
- **Imutabilidade**: o conteúdo estático fica embutido na imagem, promovendo releases imutáveis e reprodutíveis.
- **Simplicidade na arquitetura**: um único processo (Nginx) serve o conteúdo; não há runtime adicional.
- **Versão fixa**: fixar a tag da imagem evita mudanças inesperadas (melhor para previsibilidade de build).
