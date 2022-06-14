---
tags: [javascript, node, git]
title: Criando e automatizando o versionamento semântico de projetos NodeJS
---

# Criando e automatizando o versionamento semântico de projetos NodeJS

![cover](https://res.cloudinary.com/practicaldev/image/fetch/s--SoVh2s7u--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/i/pxkcelqkp2luw2svox49.jpg)

<i><span>Photo by <a href="https://unsplash.com/@jdubs?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Johnson Wang</a> on <a href="https://unsplash.com/s/photos/stack?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span></i>

## Introdução

E ae dev, tudo bem com você?

Desde que eu comecei a trabalhar com JavaScript do lado do servidor, e a utilizar as mais diversas bibliotecas lá do NPM, sempre me questionei como elas eram mantidas... Desde as suas versões publicadas, padrões dos projetos que podem ser seguidos por um ou mais desenvolvedores, ou até mesmo por uma equipe dedicada apenas nisso.

A questão que mais me intrigava era: como saber quais versões DEVEM ser publicadas, quando uma alteração é feita?

Bom, com base nisso, nos últimos dias dediquei os estudos a explorar os mais variados repositórios no GitHub, e as bibliotecas mais populares no momento no NPM. Identifiquei alguns padrões que podem facilitar a manutenção dos projetos com automatização de processos!

Ao final deste artigo, você irá:

- Entender a importância de padronizar um projeto, antes de desenvolver.
- Entender como funciona um versionamento semântico.
- Entender o que são commits semânticos.
- Aprender a automatizar a publicação/distribuição do seu projeto com base no versionamento.

Bora pro post?

## Sumário

- [O problema](#o-problema)
- [Entendendo o versionamento semântico](#entendendo-o-versionamento-semântico)
- [Entendendo o que são commits semânticos](#entendendo-o-que-são-commits-semânticos)
  - [Especificação Conventional Commits](#especificação-conventional-commits)
  - [Por que usar?](#por-que-usar)
  - [Como isso se relaciona com o SemVer?](#como-isso-se-relaciona-com-o-semver)
- [Hands-On](#hands-on)
  - [Iniciando o projeto](#iniciando-o-projeto)
  - [Habilitando a padronização de commits semânticos](#habilitando-a-padronização-de-commits-semânticos)
  - [Instalando o husky e integrando-o ao commitlint](#instalando-o-husky-e-integrando-o-ao-commitlint)
  - [Facilitando a criação de commits padronizados](#facilitando-a-criação-de-commits-padronizados)
  - [Gerando versões semânticas e CHANGELOG](#gerando-versões-semânticas-e-changelog)
- [Workflow de desenvolvimento](#workflow-de-desenvolvimento)

## O problema

Imagine que você esteja trabalhando com o cenário, onde as versões do seu projeto precisam ser coerentes com cada ajuste que você precise fazer, isto é, as versões devem indicar aquilo que foi feito. Seja uma implementação de uma nova funcionalidade, uma correção de um bug, ou até mesmo um _breaking change_ por remover uma _feature_ ou mudar completamente a integração do seu projeto, com os demais projetos que o utilizam.

> A coerência com as versões do seu projeto irá auxiliar na manutenção do projeto como um todo!

O **SemVer** está aqui para nos ajudar!

## Entendendo o versionamento semântico

Vamos entender melhor como funciona a especificação SemVer!

Ela aborda um conjunto simples de regras e requisitos que determinam como os números da versão são atribuídos e por sua vez, incrementados.

> "Sob esse esquema, os números da versão e a maneira como eles mudam transmitem **significado** sobre o código subjacente e o que foi modificado de uma versão para a seguinte".

Em resumo, dado o número de versão `MAJOR`.`MINOR`.`PATCH`, deve-se incrementá-las seguindo as seguintes regras:

1. **MAJOR**: quando você realizar alterações incompatíveis da API;

2. **MINOR**: quando você adicionar funcionalidades compatíveis com versões anteriores;

3. **PATCH**: quando você corrigir erros compatíveis com versões anteriores.

Para a nossa configuração, o essencial está nesse resumo. Você pode ler mais sobre a especificação em: https://semver.org/

Recomendo também a leitura da seção _FAQ_ disponível no site, lá você encontrará respostas para dúvidas do tipo: _"Como sei quando lançar a 1.0.0?"_.

> A especificação SemVer foi originalmente criada por [Tom Preston-Werner](https://en.wikipedia.org/wiki/Tom_Preston-Werner), inventor do [Gravatar](https://en.wikipedia.org/wiki/Gravatar) e co-fundador do [GitHub](https://en.wikipedia.org/wiki/GitHub).

## Entendendo o que são commits semânticos

Você em algum momento já se questionou como deveria escrever uma mensagem de commit (se deveria colocar muitos detalhes, descrever melhor o que fez no corpo do commit, usar algum prefixo, e etc.).

Seria mágico termos algum padrão para usarmos no nosso projeto, que siga uma maneira consistente e coesa de escrever os commits, e que informe exatamente o que foi feito ali, não é mesmo?

Pois é, e TEM!

### Especificação Conventional Commits

A especificação _Conventional Commits_ é inspirada e baseada fortemente no [guideline de commits do Angular](https://github.com/angular/angular/blob/22b96b9/CONTRIBUTING.md#-commit-message-guidelines). É uma convenção bem simples para seguir ao escrever commits e que oferece um conjunto fácil de regras para manter um histórico de commits mais explícito e facilmente compreendido.

Em resumo, para seguir a especificação, um commit deve ser estruturado da seguinte maneira:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

O commit pode conter alguns elementos estruturais, que comunicam a intenção aos "consumidores" do seu projeto:

1. **fix**: um commit "do tipo" _fix_ indica que aquela alteração corrige algum bug no projeto (isso se correlaciona ao `PATCH` do SemVer);

2. **feat**: um commit "do tipo" _feat_ indica que aquela alteração adiciona alguma nova funcionalidade ao projeto (isso se correlaciona ao `MINOR` do SemVer);

3. **BREAKING CHANGE**: um commit que possui um rodapé com _BREAKING CHANGE_ ou está diretamente na mensagem com `!` depois do _type_ ou _scope_, indica que aquela alteração, muda a compatibilidade da sua API com os "consumidores" (isso se correlaciona com o `MAJOR` do SemVer). Um _BREAKING CHANGE_ pode fazer parte de commits de qualquer _type_;

4. Outros tipos além de `feat` e `fix` também são permitidos.

Um _scope_ pode ser fornecido ao _type_ do commit, para fornecer informações contextuais adicionais e pode ser encontrado entre parênteses na mensagem, e.g.:

```
feat(parser): add ability to parse arrays.
```

> Você pode ler a especificação completa em: https://www.conventionalcommits.org/en/v1.0.0/#specification

### Por que usar?

Ao adotar os padrões no seu projeto, você poderá:

- Determinar automaticamente o _bump_ das versões (de maneira semântica, com base nos tipos dos commits criados);
- Comunicar com clareza a natureza das alterações (seja aos colegas de equipe ou ao público);
- Automatizar o processo de _build_ e publicação/distribuição do projeto.
- **Gerar CHANGELOGs automaticamente.**

> Outro ganho é que os desenvolvedores serão encorajados a **granularizar** os commits, uma vez que os padrões adotados visam contextualizar melhor a alteração feita (com tipos, escopos, e etc.). Isso é muito bom ao trabalhar com _Code Review_ e _Pull Requests_.

### Como isso se relaciona com o SemVer?

Como vimos, os tipos dos commits se relacionam com cada "acrônimo" da especificação SemVer:

- _fix:_ deve ser traduzido em liberações de _PATCH_;
- _feat:_ deve ser traduzido em liberações _MINOR_;
- _BREAKING CHANGE:_ deve ser traduzido, independentemente do tipo, em liberações _MAJOR_;

## Hands-On

Bom, agora que já entendemos como o versionamento e os commits semânticos funcionam, vamos criar um projeto com as configurações ideais para:

- Realizar o incremento automático de versões (coesas, através da análise dos commits);
- Realizar a geração automática do arquivo `CHANGELOG.md`.
- Realizar a distribuição/publicação da versão gerada (com a ajuda de CI/CD).

> TL;DR: O arquivo `CHANGELOG.md`, na raiz do projeto, contém o descritivo de todas as alterações (pertintentes) feitas naquele específico projeto. A razão para criar e manter um _changelog_ é simples: quando um colaborador ou usuário final deseja verificar se alguma alteração foi feita em um software, ele pode fazer isso com facilidade e precisão lendo o registro de alterações. Tudo o que eles precisam fazer é ir para o _changelog_ e ele mostrará o que e quando as alterações foram feitas entre as diferentes versões ou lançamentos daquele software.

### Iniciando o projeto

1. Vamos criar um novo projeto NodeJS e criar o `package.json`, com o seguinte comando:

```bash
$ mkdir my-project && cd my-project && yarn init -y
```

2. Posteriormente, usaremos um hook do Git para que a cada vez que realizarmos um commit, seja feita uma análise do commit em questão para identificar se ele está ou não no padrão especificado pelo _Conventional Commits_. Então, vamos inicialiazar o git no projeto:

```bash
$ git init
```

### Habilitando a padronização de commits semânticos

Para realizar a análise dos commits criados, precisamos de uma ferramenta que fará esse trabalho e nos auxiliará a adotar os padrões que vimos anteriormente. Vamos então instalar e configurar o [commitlint](https://commitlint.js.org/#/).

1. Comece instalando os pacotes `cli` e `config-conventional` do commitlint nas dependências de desenvolvimento:

```bash
$ yarn add -D @commitlint/{config-conventional,cli}
```

2. Vamos criar o arquivo de configuração `commitlint.config.js` na raiz do projeto com o trecho abaixo:

```javascript
module.exports = {
  extends: ["@commitlint/config-conventional"],
};
```

3. Valide as configurações com o comando:

```bash
echo 'foo: bar' | yarn commitlint
```

Você deverá visualizar no terminal algo como:

```bash
⧗   input: foo: bar
✖   Please add rules to your `commitlint.config.js`
    - Getting started guide: https://git.io/fhHij
    - Example config: https://git.io/fhHip [empty-rules]

✖   found 1 problems, 0 warnings
ⓘ   Get help: https://github.com/conventional-changelog/commitlint/#what-is-commitlint
```

### Instalando o husky e integrando-o ao commitlint

Bom, até aqui, apenas configuramos a ferramenta que realiza análise dos nossos commits. Para que ela seja usada, a cada novo commit, precisaremos instalar o [husky](https://www.npmjs.com/package/husky) e configurá-lo com o `commitlint`.

1. Comece instalando o `husky` como dependência de desenvolvimento:

```bash
$ yarn add -D husky
```

2. Vamos agora habilitar o hook `commit-msg` criando um arquivo `.huskyrc` (na raiz do projeto) com o trecho abaixo:

```json
{
  "hooks": {
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
  }
}
```

3. Valide a configuração criando um commit, no seguinte formato:

```bash
$ git commit -m "foo: bar" --allow-empty
```

Você deverá visualizar no terminal algo como:

```bash
husky > commit-msg (node v12.16.1)
⧗   input: foo: bar
✖   Please add rules to your `commitlint.config.js`
    - Getting started guide: https://git.io/fhHij
    - Example config: https://git.io/fhHip [empty-rules]

✖   found 1 problems, 0 warnings
ⓘ   Get help: https://github.com/conventional-changelog/commitlint/#what-is-commitlint

husky > commit-msg hook failed (add --no-verify to bypass)
```

Perceba que o `husky` habilitou o _hook_ **commit-msg**, o `commitlint`, por sua vez, foi executado e fez a análise do que escrevemos. Com isso os nossos commits serão analisados antes de serem criados!

Para que haja sucesso na criação do commit, ele deve estar padronizado seguindo a especificação.

### Facilitando a criação de commits padronizados

Imagine que você esteja realizando um commit, e por ventura não se lembre de algum tipo que esteja na especificação, ou até mesmo não lembre do formato específico que comunique um _breaking change_, por exemplo. O [Commitizen](#https://github.com/commitizen/cz-cli) disponibiliza uma CLI que nos auxilia na criação de commits padronizados.

> Essa configuração, é opcional, mas é bem interessante tê-la aqui para os futuros colaboradores terem mais facilidade e entendimento quanto aos padrões do projeto.

1. Comece configurando a CLI no repositório com o comando:

```bash
$ npx commitizen init cz-conventional-changelog --yarn --dev
```

Será adicionado ao `package.json`:

```json
{
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  }
}
```

2. A seguir, vamos adicionar um script ao `package.json` para iniciar a CLI:

```json
{
  "scripts": {
    "commit": "git-cz"
  }
}
```

Execute o comando `yarn commit --allow-empty`. Você verá a ferramenta entrar em ação!

> A flag `--allow-empty`, apesar de intuitiva vale a pena informar que ela possibilita a criação de commits sem ser necessário adicionar uma modificação em _staged_.

  <div align="center">
    <img width="720" src="https://dev-to-uploads.s3.amazonaws.com/i/hbujbz2nii62b9yxxqt3.png" alt="commitzen-git-cz.png"/>
  </div>

> É possível definir o commitizen como padrão ao executar `git commit` no terminal, veja mais em: https://github.com/commitizen/cz-cli#optional-running-commitizen-on-git-commit

Extra: se o seu projeto for **open-source**, com essa configuração, é possível adicionar a _badge_ "commitizen friendly" no `README.md` do repositório:

  <div align="center">

[![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg)](http://commitizen.github.io/cz-cli/)

  </div>

```markdown
[![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg)](http://commitizen.github.io/cz-cli/)
```

### Gerando versões semânticas e CHANGELOG

Até aqui, já podemos gerar os commits semânticos. Através deles, utilizaremos uma ferramenta que realiza a análise dos commits novos (adicionados desde a última versão do projeto) e determina qual será essa nova versão para a distribuição. Por padrão, ela também criará o CHANGELOG.md automaticamente de acordo com a análise feita.

Vamos configurar o projeto com o [standard-version](https://github.com/conventional-changelog/standard-version).

1. Comece instale o `standard-version` como dependência de desenvolvimento:

```bash
$ yarn add -D standard-version
```

2. Em seguida, adicione o script abaixo no `package.json`:

```json
{
  "scripts": {
    "release": "standard-version"
  }
}
```

Ao executar o comando `yarn release` (ou `npm rum release`):

<div align="center">
  <img width="720" src="https://dev-to-uploads.s3.amazonaws.com/i/aev8aczwuiw7i5xxq5cw.gif" alt="cmd-yarn-release.gif" />
</div>

- Será realizada a análise dos commits feitos após o último _release_.
- Será incrementada a versão do projeto no `package.json`, com base na análise dos commits.
- Será gerado o `CHANGELOG.md`, incluindo os detalhes da nova versão.
- Será criada uma [tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging) com base na versão do `package.json`.

  > Para remover o prefixo "v" da tag, basta adicionar a flag `-t` sem nenhum valor no script criado, e.g.: `standard-version -t \"\"`.

Depois de executar o comando, você pode publicar o projeto com o `npm publish` e enviar a _tag_ gerada para o repositório remoto com `git push --follow-tags origin master`.

## Workflow de desenvolvimento

Com ajuda de uma esteira de _CI/CD_, é possível automatizar o processo de publicação/distribuição de novas versões, para que a cada nova modificação na branch `master`, sejam executados os comandos:

1. Gerando uma nova versão: `yarn release` (ou nom `run release`);

2. Publicando a nova versão: `npm publish`

3. Enviando a tag gerada para o repositório: `git push --follow-tags origin master`

Mas para que isso seja possível, deve-se seguir o seguinte fluxo de desenvolvimento:

<div align="center">
  <img width="720" src="https://dev-to-uploads.s3.amazonaws.com/i/fk46e2v3d6t2ted2xj69.png" alt="development-workflow.png" />
</div>

1. Criar uma nova _[feature branch](#https://martinfowler.com/bliki/FeatureBranch.html)_ a partir da branch principal (master);

2. Realizar as alterações e "commitá-las" nos padrões estabelecidos pelas especificações;

3. Mesclar as alterações na branch principal via _Pull Request_;

4. A esteira de CI/CD deve ser acionada assim que houver uma nova alteração na branch principal, e (além de executar outros passos durante o processo, como testes, coleta de cobertura, lint, e etc.) incluir os comandos mencionados anteriormente.

## Finalizando...

Bem, é isso, por hoje, é só!

Quero te agradecer por chegar até aqui, e queria lhe pedir também para me encaminhar as suas dúvidas, comentários, críticas, correções ou sugestões sobre a publicação.

Deixe seu ❤️ se gostou ou um 🦄 se esse post te ajudou de alguma maneira! Não se esqueça de ver os posts anteriores e me siga para maaaais conteúdos.

Até!
