---
id: e2e
title: "Teste End-to-End"
---

Alguns projetos organizam pacotes em repositórios multi-pacotes ou [mono repositórios](https://github.com/babel/babel/blob/master/doc/design/monorepo.md). O teste E2E é um tópico que geralmente só é relevante para as Interfaces de usuário, porém do ponto de vista do Node.js, **a publicação de pacotes também requerem testes**.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Solution: a local npm registry. <a href="https://t.co/kvcyVANVSK">https://t.co/kvcyVANVSK</a></p>&mdash; Dan Abramov (@dan_abramov) <a href="https://twitter.com/dan_abramov/status/951427674844680192?ref_src=twsrc%5Etfw">11 de Janeiro de 2018</a></blockquote>

<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Tal abordagem tem sido realmente difícil de alcançar, considerando:

* Preencher pacotes canários em serviços públicos parece não ser uma boa ideia
* Alguns registros de OSS auto-hospedados são muito pesados
* Ambientes offline (redes privadas)

O **Verdaccio** é um registro leve com zero configurações que **se encaixa perfeitamente em qualquer fluxo de trabalho E2E + CI**.

## Implementação

Ainda não há uma solução única, cada implementação parece ser específica de cada projeto, você pode verificar algumas delas [clicando neste tópico](https://stackoverflow.com/a/50222427/308341).

### Exemplos usando Bash

Este é o exemplo mais simples usando o Verdaccio em um script bash (extraído do *create-react-app *).

```bash
#!/bin/sh

set -e

local_registry="http://0.0.0.0:4873"

# start local registry
tmp_registry_log=`mktemp`
sh -c "mkdir -p $HOME/.config/verdaccio"
sh -c "cp --verbose /config.yaml $HOME/.config/verdaccio/config.yaml"
sh -c "nohup verdaccio --config $HOME/.config/verdaccio/config.yaml &>$tmp_registry_log &"
# wait for `verdaccio` to boot
grep -q 'http address' <(tail -f $tmp_registry_log)
# login so we can publish packages
sh -c "npm-auth-to-token -u test -p test -e test@test.com -r $local_registry"
# Run nmp command
sh -c "npm --registry $local_registry publish"
```

## Quem está usando para E2E?

* [create-react-app](https://github.com/facebook/create-react-app/blob/master/CONTRIBUTING.md#contributing-to-e2e-end-to-end-tests) *(+64k ⭐️)*
* [Storybook](https://github.com/storybooks/storybook) *(+34k ⭐️)*
* [Gatsby](https://github.com/gatsbyjs/gatsby) *(+31k ⭐️) WIP* [#8791](https://github.com/gatsbyjs/gatsby/pull/8791) [#11525](https://github.com/gatsbyjs/gatsby/pull/11525)
* [Uppy](https://github.com/transloadit/uppy) *(+15k ⭐️)*
* [Aurelia Framework](https://github.com/aurelia) *(+10k ⭐️)*
* [bit](https://github.com/teambit/bit) *(+6k ⭐️)*
* [pnpm](https://github.com/pnpm/pnpm) *(+5k ⭐️)*
* [Mozilla Neutrino](https://github.com/neutrinojs/neutrino) *(+3k ⭐️)*
* [Hyperledger Composer](https://github.com/hyperledger/composer) *(+1.6k ⭐️)*
* [Wix Yoshi](https://github.com/wix/yoshi)

## Futuro

Babel.js pode estar interessado em integrar o verdaccio em seus fluxos de trabalho, se você quiser contribuir, verifique [este ticket ](https://github.com/babel/babel/issues/6134).

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Thinking of using verdaccio to test <a href="https://twitter.com/lernajs?ref_src=twsrc%5Etfw">@lernajs</a> v3 (+use this generally), as it&#39;s hard to know if a publish will be successful. Would like us to fix an issue where we would like to compile Babel using itself before it&#39;s published (as we self-host but from latest npm) as a smoke test</p>&mdash; Henry Zhu (@left_pad) <a href="https://twitter.com/left_pad/status/1045770889051164672?ref_src=twsrc%5Etfw">28 de Setembro de 2018</a></blockquote>

<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
