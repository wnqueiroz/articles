---
tags: [git, github]
title: Drops #02: Como alterar o autor do commit depois do push
---

# Drops #02: Como alterar o autor do commit depois do push

![cover](https://res.cloudinary.com/practicaldev/image/fetch/s--WU5GicXu--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/i/plfz332gbsx4h7xvledr.jpg)

<i><span>Photo by <a href="https://unsplash.com/@yancymin?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Yancy Min</a> on <a href="https://unsplash.com/@yancymin?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span></i>

E ae dev, tudo bem com você?

Semana passada eu finalmente comecei a usar o dev.to, inaugurando-o com um post que eu havia escrito originalmente lá no Medium, se você ainda não leu, já salva pra ler mais tarde: [Drops #01: Corrigindo vulnerabilidades em dependências com Yarn! (ou quase)
](https://dev.to/wnqueiroz/drops-01-corrigindo-vulnerabilidades-em-dependencias-com-yarn-ou-quase-2e2p)

Bom, continuando essa série _"drops"_ com dicas e truques que eu uso no dia-a-dia...

Quem nunca subiu um código para o repositório depois de ter "comitado" com o usuário que você usa na empresa, não é mesmo? HAHAHA

Calma, isso é mais comum do que você imagina. Ainda mais se costuma utilizar uma única máquina tanto para o seu trabalho, quanto para aqueles projetos pessoais que você costuma publicar no Github.

A dica de hoje é de como alterar o autor do commit depois de já tê-lo enviado ao seu repositório remoto.

Bora pro post?

Ah! mas antes disso... Esse post faz parte de uma série de artigos "drops" que tenho aqui! Veja a lista:

- [Drops #01: Corrigindo vulnerabilidades em dependências com Yarn! (ou quase)](https://dev.to/wnqueiroz/drops-01-corrigindo-vulnerabilidades-em-dependencias-com-yarn-ou-quase-2e2p)
- [Drops #02: Como alterar o autor do commit depois do push](https://dev.to/wnqueiroz/drops-02-como-alterar-o-autor-do-commit-depois-do-push-2mgg)
- [Drops #03: Usando aliases para importação de módulos em TypeScript!](https://dev.to/wnqueiroz/drops-03-usando-aliases-para-importacao-de-modulos-em-typescript-1fe9)
- [Drops #04: Desmistificando ponteiros no Golang!](https://dev.to/wnqueiroz/drops-04-desmistificando-ponteiros-no-golang-3kj9)

---

## Alterando o usuário do repositório

Antes de realizarmos as alterações nos commits, vamos definir o usuário que, **daqui pra frente**, será o autor dos commits futuros.

Para isso, abra um terminal na pasta do seu **repositório**, e execute os comandos abaixo:

```powershell
git config --local user.email "<GIT_EMAIL>" --replace-all
git config --local user.name "<GIT_USER>" --replace-all
```

> NOTA: a flag `--replace-all` irá substituir todas as configurações anteriores.

Altere `<GIT_EMAIL>` e `<GIT_USER>` para os seus respectivos valores.

Opcional:

Eu costumo armazenar as credenciais usando o helper **store**, essa configuração evita que uma janela de credenciais seja aberta a cada interação com o repositório remoto.

Caso você queira usar a mesma configuração, execute o comando abaixo:

```powershell
git config --local credential.helper store
```

> Veja mais em: https://git-scm.com/docs/git-credential-store

Note que todos os comandos, servirão apenas para o repositório atual, caso queira essa configuração em outros repositórios, basta repetir o processo nos repositórios desejados.

## Selecionando os commits para alteração

O [git-rebase](https://git-scm.com/docs/git-rebase) é uma ferramenta poderosíssima, que nos permite alterar a linha do tempo sem ser necessário gerar novas ramificações. Com isso é possível juntar commits, reordenar, removê-los, e um bocado de outras coisas.

Vamos utilizá-lo para selecionar os commits!

Se você quer alterar todos os commits, desde o primeiro, basta executar o seguinte comando:

```powershell
git rebase -i --root
```

> Caso queira alterar apenas os 3 ou N últimos commits, basta utilizar o comando `git rebase -i HEAD~3` ou `git rebase -i HEAD~n`, sendo `n` o número de commits que deseja alterar.

Isso abrirá um arquivo chamado `git-rebase-todo`, que é exatamente ali onde informaremos o que queremos fazer com os commits, e.g.:

```powershell
pick 687f049 foo: bar
pick 25a798a fooz: barz

# Rebase f157872..25a798a onto f157872 (2 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

No nosso caso, iremos substituir `pick` por `edit`:

```diff
- pick 687f049 foo: bar
+ edit 687f049 foo: bar
- pick 25a798a fooz: barz
+ edit 25a798a fooz: barz
```

Podemos então, salvar a edição do arquivo, e fechá-lo.

## Alterando o autor

No terminal, veremos a seguinte saída após fechar o arquivo:

```sh
You can amend the commit now, with

  git commit --amend

Once you are satisfied with your changes, run

  git rebase --continue
```

Beleza! Estamos prontos.

Para cada um dos commits, iremos alterar o autor, com o seguinte comando:

```powershell
git commit --amend --author="<GIT_USER> <<GIT_EMAIL>>" --no-edit
# e.g.: git commit --amend --author="Lorem Ipsum <lorem@ipsum.com>" --no-edit
```

Altere `<GIT_EMAIL>` e `<GIT_USER>` para os seus respectivos valores.

Após isso, você deve continuar para o próximo commit, que selecionou na edição do `git-rebase-todo`:

```powershell
git rebase --continue
```

Esse processo ocorre para cada commit. Quando finalizado, o _rebasing_ é concluído.

Aqui é importante observar que, você não conseguirá enviar de volta os commits modificados com apenas o comando `git push`. Isso porque estamos reescrevendo a nossa linha do tempo, isso significa que devemos substituir a branch atual que existe lá no remoto.

Para isso, devemos usar a flag `--force`:

```powershell
git push --force
```

## E Voilà!

Alteramos o autor dos commits lá do repositório e ainda garantimos que os próximos commits sejam criados com o usuário correto!

## Finalizando…

Bem, é isso, por hoje, é só!

Quero agradecer a você que chegou até aqui, e queria lhe pedir também para encaminhar-me as suas dúvidas, comentários, críticas, correções ou sugestões sobre a postagem.

Deixe seu ❤️ se gostou ou um 🦄 se esse post te ajudou de alguma maneira! Não se esqueça de ver os posts anteriores e me siga para maaaais conteúdos.

Até!
