---
tags: [typescript, node]
title: Drops #03: Usando aliases para importação de módulos em TypeScript!
---

# Drops #03: Usando aliases para importação de módulos em TypeScript!

![cover](https://res.cloudinary.com/practicaldev/image/fetch/s--YvM_GVmu--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/i/j7ommp0uhs5jmmlwxjfq.png)

## Introdução

E ae dev, tudo bem com você?

Você já deve ter trabalhado com projetos onde as importações de arquivos e módulos eram (ou ficaram) cada vez mais aninhadas. Em um determinado momento você se perde a cada "ponto, ponto, barra, ponto, ponto, barra" (e ainda espera um pouquinho pra ver se o editor de texto ajuda a completar o caminho, pra onde você realmente quer ir (profundo, não?).

Seria muito mágico se houvesse uma maneira de alterar isso:

```typescript
import { MyClass } from "../../../../deep/module";
```

Pra isso:

```typescript
import { MyClass } from "@/deep/module";
```

Pois é, e TEM!

Bora pro post?

Ah! mas antes disso... Esse post faz parte de uma série de artigos "drops" que tenho aqui! Veja a lista:

- [Drops #01: Corrigindo vulnerabilidades em dependências com Yarn! (ou quase)](https://dev.to/wnqueiroz/drops-01-corrigindo-vulnerabilidades-em-dependencias-com-yarn-ou-quase-2e2p)
- [Drops #02: Como alterar o autor do commit depois do push](https://dev.to/wnqueiroz/drops-02-como-alterar-o-autor-do-commit-depois-do-push-2mgg)
- [Drops #03: Usando aliases para importação de módulos em TypeScript!](https://dev.to/wnqueiroz/drops-03-usando-aliases-para-importacao-de-modulos-em-typescript-1fe9)
- [Drops #04: Desmistificando ponteiros no Golang!](https://dev.to/wnqueiroz/drops-04-desmistificando-ponteiros-no-golang-3kj9)

---

## Iniciando o projeto

Vamos começar criando um projeto e inicializando o nosso `package.json`:

```bash
mkdir ts-module-aliases && cd ts-module-aliases && yarn init -y
```

Em seguida, vamos adicionar algumas dependências de desenvolvimento, sendo elas:

- O TypeScript (duh!);
- O [ts-node-dev](https://github.com/whitecolor/ts-node-dev) (que será responsável por rodar o nosso código em modo de desenvolvimento);
- O [Jest](https://github.com/facebook/jest) (precisaremos configurar algumas coisas no Jest para que ele interprete os caminhos absolutos que usaremos no nosso código);
- O [tsconfig-paths](https://www.npmjs.com/package/tsconfig-paths) (esse carinha será responsável por viabilizar o uso dos aliases).
- O [Babel](https://babeljs.io/) (ficará encarregado de realizar a construção do nosso projeto, interpretando os aliases e _transpilando_ o código com os caminhos respectivos).

Execute o comando:

```bash
yarn add typescript@4.0.3 ts-node-dev@1.0.0 jest@26.6.0 ts-jest@26.4.1 @types/jest@26.0.15 tsconfig-paths@3.9.0 @babel/core@7.12.3 @babel/cli@7.12.1 @babel/node@7.12.1 @babel/preset-env@7.12.1 @babel/preset-typescript@7.12.1 babel-plugin-module-resolver@4.0.0 -D
```

> Note que aqui foram fixadas as versões de cada dependência. Apenas para que a minha configuração e a sua sejam feitas com as mesmas versões dos pacotes.

Depois de instalar as dependências vamos iniciar as configurações!

## Configurando o TypeScript

Na raiz do projeto, crie um arquivo `tsconfig.json` com a seguinte configuração:

```json
{
  "compilerOptions": {
    "target": "es2017",
    "module": "commonjs",
    "lib": ["es6", "dom"],
    "allowJs": true,
    "rootDir": ".",
    "outDir": "./dist/lib",
    "declarationDir": "./dist/@types",
    "declaration": true,
    "removeComments": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*", "**/*.spec.ts"]
}
```

### Criando a base do projeto

Depois de configurar o Typescript, vamos criar alguns arquivos partindo da raiz do projeto, dentro da pasta `src`:

`src/services/BarService.ts`:

```typescript
export class BarService {
  bar() {
    console.log(`Hi! I'm bar method from BarService :)`);
  }
}
```

`src/controllers/FooController.ts`:

```typescript
import { BarService } from "../services/BarService";

export class FooController {
  private readonly barService: BarService;

  constructor() {
    this.barService = new BarService();
  }

  foo() {
    this.barService.bar();
  }
}
```

`src/index.ts`:

```typescript
import { FooController } from "./controllers/FooController";

const fooController = new FooController();

fooController.foo();
```

Para finalizar, vamos adicionar o script no `package.json` que executa o nosso código em modo de desenvolvimento:

```json
{
  "scripts": {
    "dev": "ts-node-dev --no-notify src/index.ts"
  }
}
```

Perceba que até aqui, ainda não temos um bom exemplo de arquivos SUPER aninhados. Será possível visualizar isso ao criarmos os nossos arquivos de testes!

## Configurando o Jest

Na raiz do projeto, crie um arquivo `jest.config.js` com a seguinte configuração:

```javascript
// For a detailed explanation regarding each configuration property, visit:
// https://jestjs.io/docs/en/configuration.html

module.exports = {
  clearMocks: true,
  coverageDirectory: "__tests__/coverage",
  coverageProvider: "v8",
  preset: "ts-jest",
  testEnvironment: "node",
  testMatch: ["**/__tests__/**/*.spec.ts"],
};
```

> NOTA: os testes da nossa aplicação, ficarão na raiz do projeto, dentro da pasta `__tests__`.

Vamos então criar os nossos arquivos de testes. Partindo da raiz do projeto, dentro da pasta `__tests__`:

`__tests__/unit/controllers/FooController.spec.ts`:

```typescript
import { FooController } from "../../../src/controllers/FooController";
import { BarService } from "../../../src/services/BarService";

describe("Unit: FooController", () => {
  let fooController: FooController;

  beforeEach(() => {
    fooController = new FooController();
  });

  describe("foo", () => {
    it("should call bar method from BarService", () => {
      const spy = jest.spyOn(BarService.prototype, "bar").mockImplementation();

      fooController.foo();

      expect(spy).toBeCalled();
    });
  });
});
```

`__tests__/unit/services/BarService.spec.ts`:

```typescript
import { BarService } from "../../../src/services/BarService";

describe("Unit: BarService", () => {
  let barService: BarService;

  beforeEach(() => {
    barService = new BarService();
  });

  describe("foo", () => {
    it("should call console.log", () => {
      const spy = jest.spyOn(console, "log").mockImplementation();

      barService.bar();

      expect(spy).toBeCalledWith(`Hi! I'm bar method from BarService :)`);
    });
  });
});
```

Olha ai, os benditos "ponto, ponto, barra, ponto, ponto, barra"!

## Configurando os aliases no projeto

Vamos adicionar a configuração abaixo no `tsconfig.json`:

```json
{
  "compilerOptions": {
    // (...)
    "baseUrl": "./",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

Esse mapeamento, fará com que todo `@/*` seja um alias para `./src/*` (com o `baseUrl` sendo a raiz do nosso projeto).

Vamos agora, fazer com que o `ts-node-dev` seja capaz de entender os nossos aliases. Adicione no script de dev (no `package.json`), O trecho `-r tsconfig-paths/register`:

```diff
- "dev": "ts-node-dev --no-notify src/index.ts"
+ "dev": "ts-node-dev -r tsconfig-paths/register --no-notify src/index.ts"
```

A partir daqui, podemos alterar as importações! Altere isso:

```typescript
import { FooController } from "../../../src/controllers/FooController";
import { BarService } from "../../../src/services/BarService";
```

Para isso:

```typescript
import { FooController } from "@/controllers/FooController";
import { BarService } from "@/services/BarService";
```

> NOTA: Altere todas as importações para usarmos o alias ao invés dos `../` ou `./`.

Ao executarmos o comando `yarn dev`, já podemos utilizar os aliases durante o desenvolvimento, porém, ao executarmos o `yarn test`, o Jest ainda não é capaz de entender os caminhos que estamos utilizando...

Vamos adicionar a propriedade `moduleNameMapper` no arquivo `jest.config.js` e fazer a seguinte configuração:

```javascript
const { pathsToModuleNameMapper } = require("ts-jest/utils");
const { compilerOptions } = require("./tsconfig.json");

module.exports = {
  // (...)
  moduleNameMapper: pathsToModuleNameMapper(compilerOptions.paths, {
    prefix: "<rootDir>",
  }),
};
```

Com isso, já é possível utilizarmos os aliases nas nossas importações!

## O problema

Até aqui, conseguimos configurar os aliases e utilizá-los tanto nos arquivos de testes quanto no source do projeto. Entretanto, precisamos ainda configurar o comando de construção do nosso projeto, somente assim ele estará pronto para publicarmos e utilizarmos no ambiente produtivo.

Vamos configurar o comando `yarn build` para construir o nosso projeto, e o comando `yarn start` para executar a o pacote construído.

Adicione os scripts no `package.json`.

```json
{
  "scripts": {
    // (...)
    "build": "tsc",
    "start": "node dist/lib/src/index.js"
  }
}
```

A seguir, execute o comando:

```bash
yarn build && yarn start
```

Você perceberá que o projeto não pode ser executado, por conta do seguinte erro:

```bash
❯ yarn start
yarn run v1.22.5
$ node dist/lib/src/index.js
internal/modules/cjs/loader.js:968
  throw err;
  ^

Error: Cannot find module '@/controllers/FooController'
Require stack:
```

Isso ocorre por que o `tsc` não é capaz de entender esses aliases, inclusive, para nossa versão de produção, pouco importa se estamos usando aliases ou caminhos relativos, o que importa é que funcione!

Outro problema também, é que se notarmos os arquivos que foram construídos na nossa pasta `dist` vamos encontrar todos os nossos arquivos de testes! O que não faz sentido algum ir pro ambiente de produção, não é mesmo?

Precisamos então:

- Fazer com que o comando de build gere um pacote funcional.
- Fazer com que o comando de build empacote apenas o código que irá para produção (e remover os arquivos de testes de lá).

Vamos fazer tudo isso com a substituição do `tsc` pelo `babel`!

> NOTA: O babel será responsável por alterar os nossos aliases, para os respectivos caminhos no projeto

### Configurando o Babel

Como já adicionamos as dependências do Babel no inicio do artigo, vamos começar o arquivo `babel.config.js` na raiz do projeto, com a seguinte configuração:

```javascript
module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        targets: {
          node: "current",
        },
      },
    ],
    "@babel/preset-typescript",
  ],
  plugins: [
    [
      "module-resolver",
      {
        root: ["."],
        alias: {
          "^@/(.+)": "./src/\\1",
        },
      },
    ],
  ],
  ignore: ["**/*.spec.ts"],
};
```

Com isso o babel irá alterar todo `^@/(.+)` para `./src/\\1`, e.g.: `@/services/BarService` para `../services/BarService`.

Vamos agora criar o arquivo `tsconfig.build.json` que irá herdar todas as configurações do nosso `tsconfig.json` e será usado apenas para criar os arquivos de declaração de tipos do projeto (dentro da pasta `dist/@types`). Isso é necessário, pois o Babel não fará esse trabalho. Adicione o seguinte no arquivo:

```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "rootDir": "./src"
  },
  "exclude": ["node_modules", "dist", "**/*.spec.ts"]
}
```

Em seguida, altere o script `start` (não precisaremos mais do `src` ali) e o de `build`.

Adicione também o script `postbuild` no `package.json`:

```json
{
  "scripts": {
    "start": "node dist/lib/index.js",
    "build": "babel src --extensions \".js,.ts\" --out-dir dist/lib --copy-files --no-copy-ignored",
    "postbuild": "tsc -p tsconfig.build.json --emitDeclarationOnly"
  }
}
```

Vamos apagar a pasta `dist` gerada anteriormente, construir o projeto com o Babel, e então rodar o comando de produção:

```bash
rm -rf dist && yarn build && yarn start
```

```bash
yarn run v1.22.5
$ babel src --extensions ".js,.ts" --out-dir dist/lib --copy-files --no-copy-ignored
Successfully compiled 3 files with Babel (704ms).
$ tsc -p tsconfig.build.json --emitDeclarationOnly
✨  Done in 3.74s.
yarn run v1.22.5
$ node dist/lib/index.js
Hi! I'm bar method from BarService :)
✨  Done in 0.35s.
```

## E Voilà!

Foi bastante coisa feita aqui! Espero que você tenha conseguido usar essa funcionalidade massa nos seus projetos, e entendido cada detalhe da configuração. No final desse artigo eu deixo o exemplo completo que desenvolvemos juntos!

## Finalizando…

Bem, é isso, por hoje é só!

Quero agradecer a você que chegou até aqui, e queria lhe pedir também para encaminhar-me as suas dúvidas, comentários, críticas, correções ou sugestões sobre a postagem.

Deixe seu ❤️ se gostou ou um 🦄 se esse post te ajudou de alguma maneira! Não se esqueça de ver os posts anteriores e me siga para maaaais conteúdos.

Até!

{% github wnqueiroz/typescript-tsconfig-paths no-readme %}
