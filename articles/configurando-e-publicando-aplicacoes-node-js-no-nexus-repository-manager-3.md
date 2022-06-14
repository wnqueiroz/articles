---
tags: [node, npm]
title: Configurando e publicando aplicações NodeJS no Nexus Repository Manager 3
---

# Configurando e publicando aplicações NodeJS no Nexus Repository Manager 3

## Introdução

E ae dev, tudo bem com você?

Fazia um tempo que não publicava nada por aqui, eis então que surgiu uma oportunidade de falar sobre o [Nexus Repository Manager 3](https://help.sonatype.com/repomanager3).

O Nexus é um gerenciador de repositórios e artefatos! Ele possibilita termos nosso próprio [Docker Hub](https://hub.docker.com/) e [NPM](https://www.npmjs.com/) privados!

Hoje vou mostrar como configurá-lo para publicar suas aplicações NodeJS, e compartilhá-las entre projetos.

Ao final deste artigo, você irá:

- Configurar um contêiner Nexus para publicação de projetos NodeJS.
- Configurar aplicações para publicação no Nexus.

Bora pro post?

## Sumário

- [Criando um contêiner Docker com o Nexus](#criando-um-contêiner-docker-com-o-nexus)
  - [Obtendo a senha do usuário padrão](#obtendo-a-senha-do-usuário-padrão)
- [Configurando o Nexus](#configurando-o-nexus)
  - [Criando um usuário para publicação de pacotes](#criando-um-usuário-para-publicação-de-pacotes)
  - [Configurando um armazenamento Blob](#configurando-um-armazenamento-blob)
  - [Criando um repositório privado (hosted)](#criando-um-repositório-privado-hosted)
  - [Criando proxy para NPM público e agrupamento de repositórios](#criando-proxy-para-npm-público-e-agrupamento-de-repositórios)
- [Configurando aplicação para publicação no Nexus](#configurando-aplicação-para-publicação-no-nexus)
  - [Habilitando Realm NPM de autenticação no Nexus](#habilitando-realm-npm-de-autenticação-no-nexus)
  - [Realizando a publicação da aplicação no Nexus](#realizando-a-publicação-da-aplicação-no-nexus)
  - [Salvando configurações de login no projeto (basic)](#salvando-configurações-de-login-no-projeto-basic)
- [Instalando dependências a partir do Nexus](#instalando-dependências-a-partir-do-nexus)
- [Finalizando...](#finalizando)
- [FAQ](#faq)
  - [Devo versionar o arquivo .npmrc contendo as credenciais do Nexus no projeto?](#devo-versionar-o-arquivo-npmrc-contendo-as-credenciais-do-nexus-no-projeto)

## Criando um contêiner Docker com o Nexus

Bom, inicialmente, precisaremos configurar o Nexus localmente para realizarmos as configurações para habilitar o nosso "NPM privado". Felizmente, a [Sonatype](https://www.sonatype.com/company/) disponibiliza uma imagem Docker pronta para executarmos!

Vamos começar subindo um contêiner docker com o seguinte comando:

```bash
$ docker run -d -p 8081:8081 --name nexus sonatype/nexus3:3.29.2
```

Em seguida vamos visualizar os logs desse contêiner:

```bash
$ docker logs nexus -f
```

Como o processo de inicialização desse contêiner é demorado, aguarde até visualizar a seguinte saída nos logs:

```bash
-------------------------------------------------

Started Sonatype Nexus OSS 3.29.2-02

-------------------------------------------------
```

> Veja mais em: https://help.sonatype.com/repomanager3/installation/installation-methods#InstallationMethods-InstallingwithDocker

Após isso, acesse a URL http://localhost:8081/, você visualizará a tela inicial do Nexus:

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/abup4104f579yu9757hx.png)

### Obtendo a senha do usuário padrão

No canto inferior direito, clique em **Sign in**. Você visualizará essa tela:

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/pi3v87ceq9mkfjw6afci.png)

Perceba que aqui, temos que obter a senha do usuário padrão (admin), em `/nexus-data/admin.password`.

Vamos acessar o contêiner e obter a senha com os comandos:

```bash
$ docker exec -it nexus /bin/bash

$ echo "$(cat /nexus-data/admin.password)" # d9f3e86b-1a5d-45f8-a851-afcba3d05fdb
```

Copie a saída do segundo comando e realize o login.

Avance na janela de configuração exibida, e defina uma nova senha. Em seguida, marque a opção _**Disable anonymous access**_.

Isso fará com que apenas usuários logados possam navegar no nosso Nexus.

Saia do contêiner digitando `exit` no terminal.

## Configurando o Nexus

Antes de sermos capazes de armazenar nossos projetos no Nexus, é necessário realizar algumas configurações.

### Criando um usuário para publicação de pacotes

Vamos no painel, na barra superior e clique no símbolo de engrenagem e navegue em `Security / Users`. Clique em **Create local user** e crie o usuário com as seguintes informações:

|    **Campo**     |      **Valor**      |
| :--------------: | :-----------------: |
|        ID        |       npmuser       |
|    First name    |         NPM         |
|    Last name     |        User         |
|      Email       | npmuser@company.com |
|     Password     |       npmuser       |
| Confirm Password |       npmuser       |
|      Status      |       Active        |
|     Granted      |      nx-admin       |

### Configurando um armazenamento Blob

Aqui criaremos uma partição lógica que queremos impor para nossos diferentes tipos de projeto, ou seja, queremos segregar nossos binários NPM e binários Maven, por exemplo, para evitar quaisquer conflitos (de nome ou outros).
Internamente, o Nexus apenas cria pastas diferentes para cada armazenamento de _Blob_ que criamos.

> Veja mais em: https://help.sonatype.com/repomanager3/high-availability/configuring-blob-stores#ConfiguringBlobStores-WhatisaBlobStore?

Navegue em `Repository / Blob Stores`, clique em **Create blob store** e crie o _blob store_ com as seguintes as informações:

| **Campo** |       **Valor**       |
| :-------: | :-------------------: |
|   Type    |         File          |
|   Name    |          NPM          |
|   Path    | /nexus-data/blobs/NPM |

Qualquer pacote que enviarmos agora (para esse Blob) será persistido no volume sob a pasta associada à ele.

### Criando um repositório privado (hosted)

Esse repositório será responsável por manter todos os nossos projetos privados que enviamos ao Nexus.

Navegue em `Repository / Repositories`, clique em **Create repository**. O Nexus trará uma lista de _recipes_ disponíveis para configuração. Selecione **npm (hosted)** e crie o repositório com as seguintes informações:

|     **Campo**     |   **Valor**    |
| :---------------: | :------------: |
|       Name        |  npm-private   |
|    Blob store     |      NPM       |
| Deployment Policy | Allow redeploy |

Aqui, em _Blob store_, selecionamos aquele que criamos anteriormente. É interessante falar que, em _Deployment Policy_, estamos habilitando o "redeploy" para ser possível sobrescrever a mesma versão do pacote que é enviado.

Note que aqui isso é configurando apenas pro nosso exemplo. Essa estratégia deve ser analisada juntamente com o time responsável que realizará a manutenção dos projetos mantidos no Nexus.

### Criando proxy para NPM público e agrupamento de repositórios

O repositório `proxy` realiza o proxy (duh!) de todas as nossas solicitações de leitura para o registro público do NPM (https://registry.npmjs.org).

Já o repositório `group` combina o repositório `proxy` e o `hosted` para possibilitar a instalação de bibliotecas de terceiros (NPM) quanto as privadas (npm-private).

Ainda em `Repository / Repositories`, clique em **Create repository**. Selecione **npm (proxy)** e crie o repositório com as seguintes as informações:

|   **Campo**    |         **Valor**          |
| :------------: | :------------------------: |
|      Name      |         npm-proxy          |
| Remote Storage | https://registry.npmjs.org |
|   Blob store   |            NPM             |

Voltando para `Repository / Repositories`, clique em **Create repository**. Selecione **npm (group)** e crie o repositório com as seguintes as informações:

| **Campo**  |       **Valor**        |
| :--------: | :--------------------: |
|    Name    |       npm-group        |
| Blob store |          NPM           |
|  Members   | npm-proxy, npm-private |

## Configurando aplicação para publicação no Nexus

Afim de agilizar o processo de configuração, eu já deixei um projeto pré-configurado para brincarmos aqui. Baixe o projeto com o comando abaixo:

```bash
$ git clone https://github.com/wnqueiroz/nodejs-nexus-repository-setup.git
```

O nosso objetivo aqui será publicar a `application-a` no Nexus, e realizar a instalação dela na `application-b`.

Acesse o projeto `application-a`, e abra o `package.json`.

Note que lá temos uma configuração de publicação:

```jsonc
{
  // ...
  "publishConfig": {
    "registry": "http://localhost:8081/repository/npm-private/"
  }
}
```

Ou seja, toda vez que executarmos o comando `npm publish`, o NPM irá utilizar o repositório privado (que criamos lá atrás, `npm-private`) como registry de publicação.

A mesma configuração está disponível na `application-b`.

Em um terminal, na `application-a`, vamos tentar realizar a publicação do projeto. Execute o comando:

```bash
$ npm publish
```

Perceba que não foi possível realizar a publicação, pois ainda não estamos logados na CLI do NPM:

```bash
npm ERR! code E401
npm ERR! Unable to authenticate, need: BASIC realm="Sonatype Nexus Repository Manager"
```

Vamos realizar o login no repositório privado, usando o usuário de publicação que criamos em _[Criando um usuário para publicação de pacotes](#criando-um-usuário-para-publicação-de-pacotes)_.

Execute o comando abaixo informando as crendenciais de acesso:

```bash
$ npm login --registry=http://localhost:8081/repository/npm-private/
```

Em seguida, tente realizar a publicação novamente:

```bash
$ npm publish
```

Veja que, mesmo logados, ainda temos o mesmo problema:

```bash
npm ERR! code E401
npm ERR! Unable to authenticate, need: BASIC realm="Sonatype Nexus Repository Manager"
```

Isso acontece por que precisamos dizer ao Nexus, que ele aceite esse tipo de publicação (logados localmente através da CLI). Para fazer isso, vamos habilitar um dos _Realms_ do Nexus, precisamente o `npm Bearer Token Realm`.

### Habilitando Realm NPM de autenticação no Nexus

Os _Realms_ no Nexus, nada mais são de que um mecanismo de segurança para segregar os tipos de autenticação permitidas.

> Veja mais em: https://help.sonatype.com/repomanager3/system-configuration/access-control/realms

Voltando à interface do Nexus, na seção de configuração, acesse `Security / Realms`. Selecione `npm Bearer Token Realm` em **Available**, e adicione na guia **Active**:

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/78kw574odghhqinal8up.png)

Clique em **Save** para finalizar a configuração.

### Realizando a publicação da aplicação no Nexus

Voltando ao terminal na `application-a`, vamos tentar realizar novamente a publicação do projeto. Execute o comando:

```bash
$ npm publish
```

Veja que agora conseguimos realizar a publicação! 🚀🎉

```bash
npm notice
npm notice 📦  application-a@1.0.0
npm notice === Tarball Contents ===
npm notice 39B  src/index.js
npm notice 321B package.json
npm notice === Tarball Details ===
npm notice name:          application-a
npm notice version:       1.0.0
npm notice package size:  368 B
npm notice unpacked size: 360 B
npm notice shasum:        f40f2d6547110507a8d72481be0614eab3e9b659
npm notice integrity:     sha512-Aw1e784PXCFUT[...]BQKZZEnlJ61Yg==
npm notice total files:   2
npm notice
+ application-a@1.0.0
```

### Salvando configurações de login no projeto (basic)

Em alguns casos, não podemos utilizar a CLI do NPM para realizar o login (usando STDIN e STDOUT), como por exemplo, em um ambiente de CI. Podemos configurar o nosso projeto para utilizar uma autenticação básica (um pouco diferente da autenticação através do Realm e Login).

Na raiz do projeto, crie um arquivo `.npmrc` com o seguinte conteúdo:

```bash
email=npmuser@company.com
always-auth=true
_auth=<BASE_64_TOKEN>
```

Em um terminal, gere o Base64 do usuário e senha que criamos em _[Criando um usuário para publicação de pacotes](#criando-um-usuário-para-publicação-de-pacotes)_:

```bash
$ echo -n 'npmuser:npmuser' | openssl base64 # bnBtdXNlcjpucG11c2Vy
```

Substitua `<BASE_64_TOKEN>` no `.npmrc` com o Base64 gerado. Seu arquivo deve estar com o seguinte conteúdo:

```bash
email=npmuser@company.com
always-auth=true
_auth=bnBtdXNlcjpucG11c2Vy
```

Agora vamos validar a configuração realizando o logout da CLI do NPM e publicando o pacote novamente. Em um terminal, na `application-a`, execute os comandos:

```bash
$ npm logout --registry=http://localhost:8081/repository/npm-private/
$ npm publish
```

> Aqui, como habilitamos o "redeploy" em _[Criando um repositório privado (hosted)](#criando-um-repositório-privado-hosted)_, é possível sobrescrevermos a versão 1.0.0, publicada anteriormente.

Veja que agora conseguimos realizar a publicação sem realizarmos o login na CLI do NPM! 🚀🎉

## Instalando dependências a partir do Nexus

Agora vamos configurar a `application-b` para instalarmos a `application-a` no projeto.

Vamos copiar o conteúdo do `.npmrc` criado na `application-a` e vamos criar um `.npmrc` na `application-b`:

```bash
email=npmuser@company.com
always-auth=true
_auth=bnBtdXNlcjpucG11c2Vy
```

Aqui também vamos adicionar o `registry` com o `npm-group`:

```bash
registry=http://localhost:8081/repository/npm-group/
```

Usamos o repositório `npm-group` para conseguirmos obter tanto as aplicações públicas (direto do NPM registry, através do repositório `npm-proxy`) tanto quanto as aplicações privadas (através do repositório `npm-private`). E como dito anteriormente, isso só é possível pois agrupamos esses repositórios no `npm-group`.

Ao final, seu arquivo deverá conter o seguinte:

```bash
email=npmuser@company.com
always-auth=true
_auth=bnBtdXNlcjpucG11c2Vy
registry=http://localhost:8081/repository/npm-group/
```

Vamos testar a configuração realizando a instalação da `application-a` na `application-b`. Em um terminal, na `application-b`, execute o comando:

```bash
$ npm install application-a
```

Aqui já conseguimos instalar aplicações em outros projetos a partir do Nexus.

Agora, vamos instalar o `axios` para testar se conseguimos instalar aplicações públicas, direto do registry do NPM:

```bash
$ npm install axios
```

Finalizamos todas as nossa configurações! 🚀🎉

Caso tenha alguma dúvida, ou tenha deixado passar alguma configuração, no projeto base existe uma _branch_ com toda a configuração realizada até aqui:

https://github.com/wnqueiroz/nodejs-nexus-repository-setup/tree/final-setup

## Finalizando...

Bem, é isso, por hoje, é só!

Quero te agradecer por chegar até aqui, e queria lhe pedir também para me encaminhar as suas dúvidas, comentários, críticas, correções ou sugestões sobre a publicação.

Deixe seu ❤️ se gostou ou um 🦄 se esse post te ajudou de alguma maneira! Não se esqueça de ver os posts anteriores e me siga para maaaais conteúdos.

Até!

## FAQ

### Devo versionar o arquivo .npmrc contendo as credenciais do Nexus no projeto?

Bem, idealmente, nenhum arquivo contendo informações sensíveis devem ser "versionados" no SCM. Toda e qualquer informação inerente à ambiente, devem estar disponíveis através de variáveis de ambiente. Entretanto, a CLI do NPM não nos permite realizar o login sem ser através de STDIN e STDOUT (de maneira não interativa). O CircleCI por exemplo, sugere para inserirmos o token `_auth` no arquivo `.npmrc` em tempo de execução na própria pipeline (obtendo o token através de uma variável de ambiente), algo como:

```bash
$ echo -n "_auth=$NPM_TOKEN" >> .npmrc
```

A variável `$NPM_TOKEN` deve conter o token de autenticação.

Alternativamente, existem algumas soluções como a https://www.npmjs.com/package/npm-cli-login, que possibilita realizar o login no NPM através de variáveis de ambiente.

A decisão é dada através de um alinhamento com todo o time para que não só a esteira de CI funcione, mas também a possibilidade do desenvolvedor ter mais flexibilidade quando for atuar nos projetos privados.

Pessoalmente, eu crio um usuário "devops" com específicas permissões e mantenho o arquivo `.npmrc` com o `_auth`. O combinado não sai caro! 😏
