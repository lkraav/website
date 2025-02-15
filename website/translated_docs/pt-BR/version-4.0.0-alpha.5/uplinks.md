---
id: version-4.0.0-alpha.5-uplinks
title: Uplinks
original_id: uplinks
---

An *uplink* is a link with an external registry that provides acccess to external packages.

![Uplinks](https://user-images.githubusercontent.com/558752/52976233-fb0e3980-33c8-11e9-8eea-5415e6018144.png)

### Utilização

```yaml
uplinks:
  npmjs:
   url: https://registry.npmjs.org/
  server2:
    url: http://mirror.local.net/
    timeout: 100ms
  server3:
    url: http://mirror2.local.net:9000/
  baduplink:
    url: http://localhost:55666/
```

### Configuração

You can define mutiple uplinks and each of them must have an unique name (key). They can have two properties:

| Propriedade  | Tipo    | Obrigatório | Exemplo                                 | Suporte  | Descrição                                                                                                                 | Padrão     |
| ------------ | ------- | ----------- | --------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------- | ---------- |
| url          | string  | Sim         | https://registry.npmjs.org/             | completo | A url do registro                                                                                                         | npmjs      |
| ca           | string  | Não         | ~./ssl/client.crt'                      | completo | Local do certificado SSL                                                                                                  | No default |
| timeout      | string  | Não         | 100ms                                   | completo | define novo tempo limite para o pedido                                                                                    | 30s        |
| maxage       | string  | Não         | 10m                                     | completo | o limite de tempo para o cache ser válido                                                                                 | 2m         |
| fail_timeout | string  | Não         | 10m                                     | completo | define o tempo máximo quando uma solicitação se torna uma falha                                                           | 5m         |
| max_fails    | número  | Não         | 2                                       | completo | limite máximo de falhas                                                                                                   | 2          |
| cache        | boolean | Não         | [true,false]                            | >= 2.1   | armazenar em cache todos os tarballs remotos presentes no armazenamento                                                   | true       |
| auth         | lista   | Não         | [veja abaixo](uplinks.md#auth-property) | >= 2.5   | atribui o cabeçalho 'Autorização' [mais info](http://blog.npmjs.org/post/118393368555/deploying-with-npm-private-modules) | disabled   |
| headers      | lista   | Não         | authorization: "Bearer SecretJWToken==" | completo | lista de cabeçalhos customizados para o uplink                                                                            | disabled   |
| strict_ssl   | boolean | Não         | [true,false]                            | >= 3.0   | Se verdadeiro, requer certificados SSL válidos.                                                                           | true       |

#### Propriedade Auth

The `auth` property allows you to use an auth token with an uplink. Using the default environment variable:

```yaml
uplinks:
  private:
    url: https://private-registry.domain.com/registry
    auth:
      type: bearer
      token_env: true # defaults to `process.env['NPM_TOKEN']`
```

ou através de uma variável de ambiente especificada:

```yaml
uplinks:
  private:
    url: https://private-registry.domain.com/registry
    auth:
      type: bearer
      token_env: FOO_TOKEN
```

`token_env: FOO_TOKEN` internamente vai usar `process.env['FOO_TOKEN']`

ou especificando diretamente um token:

```yaml
uplinks:
  private:
    url: https://private-registry.domain.com/registry
    auth:
      type: bearer
      token: "token"
```

> Nota: `token` possui prioridade sobre `token_env`

### Deves Saber

* Os uplinks devem ser registros compatíveis com os `npm` endpoints. Por exemplo: *verdaccio*, `sinopia@1.4.0`, *npmjs registry*, *yarn registry*, *JFrog*, *Nexus* e mais.
* Configurar o `cache` para false ajudará a economizar espaço no seu disco rígido. Isto vai evitar armazenar as `tarballs` mas [irá manter o metadata em pastas](https://github.com/verdaccio/verdaccio/issues/391).
* Exceder com vários uplinks pode atrasar a busca por pacotes, pois para cada solicitação um cliente npm é encaminhado, o verdaccio, por sua vez, faz uma chamada para cada uplink.
* O formato (timeout, maxage e fail_timeout) segue as [unidades de medida NGINX](http://nginx.org/en/docs/syntax.html)