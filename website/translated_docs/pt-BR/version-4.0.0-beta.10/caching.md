---
id: version-4.0.0-beta.10-caching
title: Estratégias de Cashing
original_id: caching
---

Verdaccio armazena todos os pacotes por padrão na pasta `/storage`. Mas você pode decidir se quer seguir uma estratégia diferente. Usando plugins você pode usar a nuvem ou qualquer tipo de banco de dados.

## Possibilidades de Cache

* A criação de um projeto Node.js em servidores de **Integração Contínua** (Bamboo, GitLab, Jenkins, etc.) é uma atividade que pode ser executada várias vezes ao dia, portanto, o servidor fará o download do registro de um grande número de tarballs sempre que isso ocorrer. Como de costume, as ferramentas de IC limpam o cache após cada compilação e o processo é iniciado repetidamente. Isto é um desperdício de bandwidth e reduz o tráfego externo. **Você pode usar o Verdaccio para fazer cache de tarballs e metadados em nossa rede interna e dar um impulso em seu tempo de compilação.**
* **Latência e Conectividade**, nem todos os países desfrutam de uma conexão de alta velocidade. Neste caso, armazenar pacotes localmente em sua rede é muito útil. Mesmo se você estiver viajando, ou tiver uma conexão fraca, estiver em roaming ou enfrentando países com firewalls fortes que possam afetar a experiência do usuário (por exemplo: corrompendo tarballs).
* **Modo Offline**, todos os Node Package Managers hoje em dia usam seu próprio cache interno, mas é comum que projetos diferentes usem ferramentas diferentes, o que implica arquivos de bloqueio e assim por diante. Essas ferramentas não podem compartilhar o cache, a única solução é centralizada e depende de um registro de proxy. Verdaccio armazena em cache todos os metadados e tarballs são baixadas por demanda, sendo capaz de compartilhá-los em todo o seu projeto.
* Evita que qualquer registro remoto retorne repentinamente erros de *HTTP 404* para tarballs que estavam previamente disponíveis, também conhecido como ([left-pad issue](https://www.theregister.co.uk/2016/03/23/npm_left_pad_chaos/)).

# Estratégias para construções mais rápidas

> Estamos à procura de mais estratégias, sinta-se à vontade para compartilhar sua experiência neste campo

## Evite armazenar tarballs

Se você tiver um espaço de armazenamento limitado, pode ser necessário evitar tarballs de cache, ativar o `cache` false em cada uplink armazenará apenas os arquivos de metadados.

    uplinks:
      npmjs:
        url: https://registry.npmjs.org/
        cache: false
    

## Estendendo o Tempo de Expiração do Cache

O Verdaccio, por padrão, aguarda 2 minutos para invalidar os metadados do cache antes de buscar novas informações do registro remoto.

```yaml
uplinks:
  npmjs:
    url: https://registry.npmjs.org/
    maxage: 30m
```

Ao aumentar o valor de `maxage` em cada `uplink`, o controle remoto será consultado com menos frequência. This might be a valid stragegy if you don't update dependencies so often.

## Usando a memória em vez do disco rígido

Às vezes, o armazenamento em cache de pacotes não é uma etapa crítica, em vez de rotear pacotes de diferentes registros e alcançar tempos de construção mais rápidos. There are two plugins that avoid write in a phisical hardrive at all using the memory.

```bash
  npm install -g verdaccio-auth-memory
  npm install -g verdaccio-memory
```

A configuração aparece como a seguir

```yaml
auth:
  auth-memory:
    users:
      foo:
        name: test
        password: test
store:
  memory:
    limit: 1000
```

Remember, once the server is restarted the data is being lost, we recomend this setup in cases where you do not need to persist at all.