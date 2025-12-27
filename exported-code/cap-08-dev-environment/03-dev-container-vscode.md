# Cap 08 - Dev Container no VSCode (exportação anotada)

## Arquivo: `cap 08 - Montando ambiente para dev/03 - Dev Container no VSCode/app-nodejs/.devcontainer/docker-compose.yml`
```yaml
version: '3'
services:
  # Update this to the name of the service you want to work with in your docker-compose.yml file
  app:
    # Uncomment if you want to override the service's Dockerfile to one in the .devcontainer 
    # folder. Note that the path of the Dockerfile and context is relative to the *primary* 
    # docker-compose.yml file (the first in the devcontainer.json "dockerComposeFile"
    # array). The sample below assumes your primary file is in the root of your project.
    #
    # build:
    #   context: .
    #   dockerfile: .devcontainer/Dockerfile

    volumes:
      # Update this to wherever you want VS Code to mount the folder of your project
      - ..:/workspaces:cached
      #- .:/home/node/app:cached

    # Uncomment the next four lines if you will use a ptrace-based debugger like C++, Go, and Rust.
    # cap_add:
    #   - SYS_PTRACE
    # security_opt:
    #   - seccomp:unconfined

    # Overrides default command so things don't shut down after the process ends.
    command: /bin/sh -c "while sleep 1000; do :; done"
```

### Anotações técnicas
- **Dev container**: define o serviço que o VS Code anexa para desenvolvimento remoto.
- **Mount da workspace**: `../:/workspaces` garante edição de arquivos no host com sincronização.
- **Comando keep-alive**: mantém o container ativo sem iniciar a aplicação.

## Arquivo: `cap 08 - Montando ambiente para dev/03 - Dev Container no VSCode/app-nodejs/docker-compose.dev.yaml`
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

## Arquivo: `cap 08 - Montando ambiente para dev/03 - Dev Container no VSCode/app-nodejs/project/Dockerfile`
```Dockerfile
FROM node:20-slim

RUN apt update && apt install -y \
        curl \
        git \
        procps \
        iputils-ping \
        fonts-powerline

USER node

WORKDIR /home/node/app

CMD [ "/home/node/app/.docker/start-dev.sh" ]
```

### Anotações técnicas
- **Ferramentas essenciais**: utilitários mínimos para debugging e networking.
- **Script de inicialização**: delega setup para o `start-dev.sh`.

## Arquivo: `cap 08 - Montando ambiente para dev/03 - Dev Container no VSCode/app-nodejs/project/.docker/start-dev.sh`
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

## Arquivo: `cap 08 - Montando ambiente para dev/03 - Dev Container no VSCode/app-nodejs/project/.eslintrc.js`
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

## Arquivo: `cap 08 - Montando ambiente para dev/03 - Dev Container no VSCode/app-nodejs/project/src/main.ts`
```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

## Arquivo: `cap 08 - Montando ambiente para dev/03 - Dev Container no VSCode/app-nodejs/project/src/app.module.ts`
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

## Arquivo: `cap 08 - Montando ambiente para dev/03 - Dev Container no VSCode/app-nodejs/project/src/app.controller.ts`
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

## Arquivo: `cap 08 - Montando ambiente para dev/03 - Dev Container no VSCode/app-nodejs/project/src/app.service.ts`
```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

## Arquivo: `cap 08 - Montando ambiente para dev/03 - Dev Container no VSCode/app-nodejs/project/src/app.controller.spec.ts`
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

## Arquivo: `cap 08 - Montando ambiente para dev/03 - Dev Container no VSCode/app-nodejs/project/test/app.e2e-spec.ts`
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

## Arquivo: `cap 08 - Montando ambiente para dev/03 - Dev Container no VSCode/app-nodejs/project/package.json`
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

### Conceitos e boas práticas
- **Dev container + Compose**: padroniza toolchain e runtime para o time.
- **Isolamento**: dependências e runtime ficam encapsulados no container.
- **Arquitetura NestJS**: módulos, controllers e services suportam escalabilidade.
