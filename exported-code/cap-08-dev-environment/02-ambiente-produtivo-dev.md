# Cap 08 - Montando ambiente produtivo para dev (exportação anotada)

## Arquivo: `cap 08 - Montando ambiente para dev/02 - Montando ambiente produtivo para dev/app-nodejs/docker-compose.dev.yaml`
```yaml
version: '3'

services:
  app:
    build: 
      context: ./project
    ports:
      - 3000:3000
    volumes:
      - ./project:/home/node/app:cached
  
  db:
    image: mysql:8.0.30-debian
    environment:
      - MYSQL_DATABASE=nest
      - MYSQL_ROOT_PASSWORD=root
    volumes:
      - ./.docker/dbdata:/var/lib/mysql:delegated
    security_opt:
      - seccomp:unconfined
```

### Anotações técnicas
- **Bind mount com cache**: `:cached` otimiza performance de filesystem no macOS/Windows.
- **Persistência do banco**: volume local em `./.docker/dbdata` para dados do MySQL.
- **`seccomp:unconfined`**: relaxa políticas para evitar bloqueios em ambientes de dev.

## Arquivo: `cap 08 - Montando ambiente para dev/02 - Montando ambiente produtivo para dev/app-nodejs/project/Dockerfile`
```Dockerfile
FROM node:20-slim

RUN apt update && apt install -y \
        curl \
        git \
        wget \
        zsh \
        procps \
        iputils-ping \
        fonts-powerline

USER node

WORKDIR /home/node/app

RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.1.5/zsh-in-docker.sh)" -- \
    -p https://github.com/zdharma-continuum/fast-syntax-highlighting \
    -p https://github.com/zsh-users/zsh-autosuggestions \
    -p https://github.com/zsh-users/zsh-completions \
    -a 'export TERM=xterm-256color'

CMD [ "/home/node/app/.docker/start-dev.sh" ]
```

### Anotações técnicas
- **Ferramentas de dev**: instala utilitários e zsh para experiência interativa dentro do container.
- **Usuário não-root**: `USER node` reforça segurança mesmo em dev.
- **Script de bootstrap**: `start-dev.sh` faz setup local e mantém o container vivo.

## Arquivo: `cap 08 - Montando ambiente para dev/02 - Montando ambiente produtivo para dev/app-nodejs/project/.docker/start-dev.sh`
```bash
#!/bin/bash

#preparar algum env
if [ ! -f "./.env" ]; then
    cp ./.env.example ./.env
fi

#instalar dependencias
npm install

#executar outros comandos
#    - migração
#    - seed

tail -f /dev/null
```

### Anotações técnicas
- **Bootstrap de ambiente**: garante `.env` e instala dependências.
- **Container persistente**: `tail -f /dev/null` mantém o container ativo para uso interativo.

## Arquivo: `cap 08 - Montando ambiente para dev/02 - Montando ambiente produtivo para dev/app-nodejs/project/.eslintrc.js`
```js
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: 'tsconfig.json',
    tsconfigRootDir: __dirname,
    sourceType: 'module',
  },
  plugins: ['@typescript-eslint/eslint-plugin'],
  extends: [
    'plugin:@typescript-eslint/recommended',
    'plugin:prettier/recommended',
  ],
  root: true,
  env: {
    node: true,
    jest: true,
  },
  ignorePatterns: ['.eslintrc.js'],
  rules: {
    '@typescript-eslint/interface-name-prefix': 'off',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-explicit-any': 'off',
  },
};
```

### Anotações técnicas
- **Lint + Prettier**: padroniza estilo de código TypeScript.
- **Regras relaxadas**: facilita prototipagem e exemplos didáticos.

## Arquivo: `cap 08 - Montando ambiente para dev/02 - Montando ambiente produtivo para dev/app-nodejs/project/src/main.ts`
```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

### Anotações técnicas
- **Bootstrap do NestJS**: cria a aplicação e inicializa o servidor HTTP.
- **Porta configurável**: padrão 3000 para dev.

## Arquivo: `cap 08 - Montando ambiente para dev/02 - Montando ambiente produtivo para dev/app-nodejs/project/src/app.module.ts`
```ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

### Anotações técnicas
- **Módulo raiz**: registra controller e service; base do padrão **Dependency Injection** do NestJS.

## Arquivo: `cap 08 - Montando ambiente para dev/02 - Montando ambiente produtivo para dev/app-nodejs/project/src/app.controller.ts`
```ts
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

### Anotações técnicas
- **Camada de apresentação**: controller lida com HTTP e delega ao serviço.
- **Injeção de dependências**: `AppService` é injetado via construtor.

## Arquivo: `cap 08 - Montando ambiente para dev/02 - Montando ambiente produtivo para dev/app-nodejs/project/src/app.service.ts`
```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

### Anotações técnicas
- **Camada de domínio**: serviço concentra lógica de negócio (aqui, resposta simples).

## Arquivo: `cap 08 - Montando ambiente para dev/02 - Montando ambiente produtivo para dev/app-nodejs/project/src/app.controller.spec.ts`
```ts
import { Test, TestingModule } from '@nestjs/testing';
import { AppController } from './app.controller';
import { AppService } from './app.service';

describe('AppController', () => {
  let appController: AppController;

  beforeEach(async () => {
    const app: TestingModule = await Test.createTestingModule({
      controllers: [AppController],
      providers: [AppService],
    }).compile();

    appController = app.get<AppController>(AppController);
  });

  describe('root', () => {
    it('should return "Hello World!"', () => {
      expect(appController.getHello()).toBe('Hello World!');
    });
  });
});
```

### Anotações técnicas
- **Teste unitário**: usa o `TestingModule` do Nest para isolar controller e service.

## Arquivo: `cap 08 - Montando ambiente para dev/02 - Montando ambiente produtivo para dev/app-nodejs/project/test/app.e2e-spec.ts`
```ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from './../src/app.module';

describe('AppController (e2e)', () => {
  let app: INestApplication;

  beforeEach(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  it('/ (GET)', () => {
    return request(app.getHttpServer())
      .get('/')
      .expect(200)
      .expect('Hello World!');
  });
});
```

### Anotações técnicas
- **Teste end‑to‑end**: sobe a aplicação completa e valida o endpoint real.

## Arquivo: `cap 08 - Montando ambiente para dev/02 - Montando ambiente produtivo para dev/app-nodejs/project/package.json`
```json
{
  "name": "app-nodejs",
  "version": "0.0.1",
  "description": "",
  "author": "",
  "private": true,
  "license": "UNLICENSED",
  "scripts": {
    "build": "nest build",
    "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "node dist/main",
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
    "test:e2e": "jest --config ./test/jest-e2e.json"
  },
  "dependencies": {
    "@nestjs/common": "^10.0.0",
    "@nestjs/core": "^10.0.0",
    "@nestjs/platform-express": "^10.0.0",
    "reflect-metadata": "^0.1.13",
    "rxjs": "^7.8.1"
  },
  "devDependencies": {
    "@nestjs/cli": "^10.0.0",
    "@nestjs/schematics": "^10.0.0",
    "@nestjs/testing": "^10.0.0",
    "@types/express": "^4.17.17",
    "@types/jest": "^29.5.2",
    "@types/node": "^20.3.1",
    "@types/supertest": "^2.0.12",
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "eslint": "^8.42.0",
    "eslint-config-prettier": "^9.0.0",
    "eslint-plugin-prettier": "^5.0.0",
    "jest": "^29.5.0",
    "prettier": "^3.0.0",
    "source-map-support": "^0.5.21",
    "supertest": "^6.3.3",
    "ts-jest": "^29.1.0",
    "ts-loader": "^9.4.3",
    "ts-node": "^10.9.1",
    "tsconfig-paths": "^4.2.0",
    "typescript": "^5.1.3"
  },
  "jest": {
    "moduleFileExtensions": [
      "js",
      "json",
      "ts"
    ],
    "rootDir": "src",
    "testRegex": ".*\\.spec\\.ts$",
    "transform": {
      "^.+\\.(t|j)s$": "ts-jest"
    },
    "collectCoverageFrom": [
      "**/*.(t|j)s"
    ],
    "coverageDirectory": "../coverage",
    "testEnvironment": "node"
  }
}
```

### Anotações técnicas
- **Scripts padronizados**: `start:dev`, `test:e2e`, `lint` suportam ciclo de desenvolvimento.
- **Stack NestJS**: dependências indicam arquitetura modular com DI.

### Conceitos e boas práticas
- **Dev container**: ambiente replicável para desenvolvimento com volume bindado.
- **Arquitetura em camadas**: controller → service → domínio.
- **Automação**: scripts de build, lint e testes para qualidade contínua.
