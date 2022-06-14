---
tags: [node, javascript, security]
title: Drops #01: Corrigindo vulnerabilidades em dependências com Yarn! (ou quase)
---

# Drops #01: Corrigindo vulnerabilidades em dependências com Yarn! (ou quase)

![cover](https://res.cloudinary.com/practicaldev/image/fetch/s--KcaxraR7--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/i/1240w44hb6z3g1gnpazf.jpeg)

_Photo by Alexander Sinn on [Unsplash](https://unsplash.com/photos/KgLtFCgfC28)_

## Disclaimer

E ae dev, tudo bem com você?

Esse post foi originado lá no meu [Medium](https://medium.com/@wnqueiroz), estou migrando-o para cá, pois adotarei o [dev.to](https://dev.to/) por diversas vantagens em relação ao Medium (o suporte ao Markdown me venceu heauehau).

Bora pro post?

Ah! mas antes disso... Esse post faz parte de uma série de artigos "drops" que tenho aqui! Veja a lista:

- [Drops #01: Corrigindo vulnerabilidades em dependências com Yarn! (ou quase)](https://dev.to/wnqueiroz/drops-01-corrigindo-vulnerabilidades-em-dependencias-com-yarn-ou-quase-2e2p)
- [Drops #02: Como alterar o autor do commit depois do push](https://dev.to/wnqueiroz/drops-02-como-alterar-o-autor-do-commit-depois-do-push-2mgg)
- [Drops #03: Usando aliases para importação de módulos em TypeScript!](https://dev.to/wnqueiroz/drops-03-usando-aliases-para-importacao-de-modulos-em-typescript-1fe9)
- [Drops #04: Desmistificando ponteiros no Golang!](https://dev.to/wnqueiroz/drops-04-desmistificando-ponteiros-no-golang-3kj9)

---

Fala pessoal!! Quanto tempo!

Alguns dias atrás, acessei o repositório de um exemplo que usei no post: [Entendendo a Context API do React: criando um componente de loading](https://medium.com/reactbrasil/entendendo-a-context-api-do-react-criando-um-componente-de-loading-a84f84007dc7) e me deparei com isso:

<br/>

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/zii4aerv2j5a3ljaz853.png)

Eu precisava então atualizar as dependências desse projeto lá no meu [Github](https://github.com/wnqueiroz). E como tenho utilizado o [Yarn](https://classic.yarnpkg.com/en/) como gerenciador de pacotes principal, quis fazer o processo de correção com ele.

Até aí, BELEZA.

<div align="center">
  <img width="480" src="https://dev-to-uploads.s3.amazonaws.com/i/uxw30r4x1poig3rmzerm.gif" alt="PLOT TWIST!">
</div>

Entretanto, notei que o Yarn até possui um script para executar a auditoria das dependências do projeto, porém, não inclui a atualização **automática** e **transparente** delas (assim como o NPM faz com o `npm audit fix`).

Se você executar no seu terminal o script `yarn audit --help`, verá que de fato não existe um script que corrija automaticamente as dependências com vulnerabilidades…

> Você pode ler mais sobre o comando audit do Yarn em: https://classic.yarnpkg.com/en/docs/cli/audit

Existem algumas _issues_ no repositório do Yarn, solicitando o recurso e etc. Não vou entrar tanto em detalhes, mas você pode dar uma olhada começando por aqui: https://github.com/yarnpkg/yarn/issues/5808

---

## Troubleshoot

A ideia aqui é aproveitar o script do NPM, e ainda assim continuar utilizando o Yarn como o gerenciador principal dos seus pacotes.

Inicialmente, vamos obter apenas `package-lock.json` que o NPM gera ao instalar as dependências (mais à frente explicarei o porque disso):

```powershell
npm i --package-lock-only
```

Então utilizaremos o script `npm audit fix`. Ele utilizará o `package-lock.json` gerado:

```powershell
npm audit fix
```

O comando atualizará a dependências **passíveis de atualização**.

E o que eu quero dizer com isso? O comando é capaz de identificar possíveis _breaking changes_ que impactam diretamente o seu projeto. Você talvez veja na saída do terminal algo como:

_"x" vulnerabilities required manual review and could not be updated_

> Você pode usar a flag `-- force`, porém, o NPM irá avisá-lo: **_I sure hope you know what you are doing_** HAHA <br/><br/> Veja mais sobre em: https://docs.npmjs.com/cli/audit

Ainda não terminamos! Até então, temos o arquivo `package-lock.json` criado e, possivelmente, o `package.json` modificado no projeto. O _lock_ de dependências de um projeto que utiliza o Yarn como gerenciador, é o arquivo `yarn.lock`.

O que faremos aqui, é gerar esse arquivo a partir do nosso `package-lock.json`.

<div align="center">
  <img width="480" src="https://dev-to-uploads.s3.amazonaws.com/i/313f22g1gc11tpp5r29i.gif" alt="PLOT TWIST!">
</div>

Antes de executar o comando a seguir, remova o arquivo `yarn.lock` para não termos algum problema na criação do novo arquivo com o `yarn import`:

```powershell
rm yarn.lock && yarn import
```

> Veja mais sobre em: https://classic.yarnpkg.com/en/docs/cli/import/

## E Voilà!

Temos o `yarn.lock` fresquinho, com as correções feitas pelo NPM, e prontinho para ser versionado!

Ah! Não esqueça de remover o `package-lock.json` gerado lá no início.

# Finalizando…

Bem, é isso, por hoje, é só!

Quero agradecer a você que chegou até aqui, e queria lhe pedir também para encaminhar-me as suas dúvidas, comentários, críticas, correções ou sugestões sobre a postagem.

Deixe seu :heart: se gostou e me siga para mais conteúdos.

Até!
