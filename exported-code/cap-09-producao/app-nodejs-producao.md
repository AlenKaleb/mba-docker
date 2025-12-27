# Cap 09 - App Node.js para produção (exportação anotada)

## Arquivo: `cap 09 - Montagem de imagens para produção/01 - Montagem de imagens para produção/app-nodejs/Dockerfile.prod`
```Dockerfile
FROM node:20-slim

LABEL maintainer="Luiz Carlos"
LABEL email="argentinaluiz@gmail.com"
LABEL version="1.0.0"

RUN apt update -y && \
    apt install --no-install-recommends -y \
    openssl && \
    rm -rf /var/lib/apt/lists/*

USER node

RUN mkdir -p /home/node/app

WORKDIR /home/node/app

COPY --chown=node:node package*.json ./

RUN npm install && npm cache clean --force

COPY --chown=node:node . .

RUN npm run build

EXPOSE 3000

CMD [ "npm", "run", "start:prod" ]
```

### Anotações técnicas
- **Labels**: metadados para rastreabilidade (maintainer, versão).
- **`--no-install-recommends`**: reduz dependências e tamanho final.
- **Usuário não-root**: boa prática de segurança.
- **Build do NestJS**: `npm run build` gera `dist/` para execução em produção.

## Arquivo: `cap 09 - Montagem de imagens para produção/01 - Montagem de imagens para produção/app-nodejs/Dockerfile.multi-stage.prod`
```Dockerfile
FROM node:20-slim as builder

RUN apt update -y && \
    apt install --no-install-recommends -y \
    openssl && \
    rm -rf /var/lib/apt/lists/*

USER node

RUN mkdir -p /home/node/app

WORKDIR /home/node/app

COPY --chown=node:node package*.json ./

RUN npm ci

COPY --chown=node:node . .

RUN npm run build

FROM node:20-slim as production

LABEL maintainer="Luiz Carlos"
LABEL email="argentinaluiz@gmail.com"
LABEL version="1.0.0"

ENV NODE_ENV production

RUN apt update -y && \
    apt install --no-install-recommends -y \
    openssl && \
    rm -rf /var/lib/apt/lists/*

USER node

RUN mkdir -p /home/node/app

WORKDIR /home/node/app

COPY --chown=node:node package*.json ./

RUN npm ci --only=production && npm cache clean --force

COPY --from=builder --chown=node:node /home/node/app/dist ./dist

EXPOSE 3000

CMD [ "npm", "run", "start:prod" ]

FROM gcr.io/distroless/nodejs20-debian12:nonroot

COPY --from=production --chown=nonroot:nonroot /home/node/app/dist /app/dist
COPY --from=production --chown=nonroot:nonroot /home/node/app/node_modules /app/node_modules

WORKDIR /app

EXPOSE 3000

CMD ["dist/main.js" ]
```

### Anotações técnicas
- **Multi-stage build**: separa build, runtime e imagem final (distroless).
- **`npm ci`**: builds reprodutíveis com `package-lock`.
- **Distroless**: remove shell e utilitários, reduzindo superfície de ataque.
- **`NODE_ENV=production`**: ajusta comportamento e dependências.

## Arquivo: `cap 09 - Montagem de imagens para produção/01 - Montagem de imagens para produção/app-nodejs/src/main.ts`
```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

## Arquivo: `cap 09 - Montagem de imagens para produção/01 - Montagem de imagens para produção/app-nodejs/src/app.module.ts`
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

## Arquivo: `cap 09 - Montagem de imagens para produção/01 - Montagem de imagens para produção/app-nodejs/src/app.controller.ts`
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

## Arquivo: `cap 09 - Montagem de imagens para produção/01 - Montagem de imagens para produção/app-nodejs/src/app.service.ts`
```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

## Arquivo: `cap 09 - Montagem de imagens para produção/01 - Montagem de imagens para produção/app-nodejs/src/app.controller.spec.ts`
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

## Arquivo: `cap 09 - Montagem de imagens para produção/01 - Montagem de imagens para produção/app-nodejs/test/app.e2e-spec.ts`
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

## Arquivo: `cap 09 - Montagem de imagens para produção/01 - Montagem de imagens para produção/app-nodejs/.eslintrc.js`
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

## Arquivo: `cap 09 - Montagem de imagens para produção/01 - Montagem de imagens para produção/app-nodejs/package.json`
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
    "axios": "^1.6.0",
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
- **Multi-stage e distroless**: melhoram segurança e tamanho da imagem final.
- **Separação build/runtime**: `builder` produz artefatos, `production` executa apenas o necessário.
- **User non-root**: recomendado para produção.
