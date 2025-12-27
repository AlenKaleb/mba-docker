# Cap 09 - Go para produção (exportação anotada)

## Arquivo: `cap 09 - Montagem de imagens para produção/01 - Montagem de imagens para produção/golang/Dockerfile`
```Dockerfile
FROM golang:1.21 as builder
WORKDIR /src
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

FROM scratch
COPY --from=builder /bin/hello /bin/hello
CMD ["/bin/hello"]
```

### Anotações técnicas
- **Multi-stage**: compila o binário no estágio `builder` e copia para uma imagem `scratch` minimalista.
- **`scratch`**: imagem vazia, ideal para binários estáticos (Go) e com superfície de ataque mínima.
- **`COPY <<EOF`**: here-doc embute código Go diretamente no build, simplificando o exemplo.

### Conceitos e boas práticas
- **Imagens mínimas**: reduzem tempo de deploy e riscos de segurança.
- **Build determinístico**: a imagem final contém apenas o binário necessário.
