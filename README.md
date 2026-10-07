# Linux Distro CI Demo

Um pequeno laboratório para demonstrar como usar **GitHub Actions** para testar um script POSIX em diferentes distribuições Linux.

Este projeto nasceu de uma pergunta simples:

> "Será que esse script realmente funciona em várias distribuições Linux?"

Em vez de discutir, decidimos colocar o script para trabalhar.

E, em homenagem ao nosso consagrado **Ryouta**, nasceu este pequeno experimento.

A ideia é simples: pegar um script POSIX extremamente pequeno e executar o mesmo teste em diferentes distribuições Linux usando **GitHub Actions + Docker**.

O resultado atual é um CI testando **13 distribuições Linux**.

## O objetivo

Este projeto não pretende ser um framework de testes nem uma suíte completa de compatibilidade.

O objetivo é demonstrar, de forma simples, como:

- criar um workflow no GitHub Actions;
- executar testes automaticamente;
- usar uma matriz (`matrix`) para repetir um teste em vários ambientes;
- utilizar containers Docker para testar diferentes distribuições;
- verificar a compatibilidade de um script POSIX;
- acompanhar os resultados diretamente pela aba **Actions** do GitHub.

Em outras palavras:

**um script pequeno, uma ideia simples e uma quantidade desnecessariamente séria de distribuições Linux.**

## O script

O teste está em:

```text
src/test.sh
```

Seu conteúdo é propositalmente simples:

```sh
#!/bin/sh

printf '%s\\n' "linux-distro-ci-demo: OK"
```

Isso é intencional.

O objetivo deste projeto não é criar um script complexo, mas ter um exemplo mínimo que permita concentrar a atenção no funcionamento do CI.

O script utiliza apenas:

- `sh`
- `printf`

Ou seja, nada específico de Bash.

## O que significa POSIX?

POSIX é uma família de padrões que define interfaces e comportamentos comuns em sistemas Unix e Unix-like.

Neste projeto, estamos interessados principalmente na utilização de:

```sh
#!/bin/sh
```

em vez de:

```sh
#!/bin/bash
```

Isso permite escrever scripts que dependem de uma base mais portátil.

Importante: **POSIX não significa automaticamente "funciona em qualquer lugar"**.

Um script ainda pode depender de comandos, opções ou comportamentos específicos de determinada implementação.

Por isso, testar é uma boa ideia.

## Distribuições testadas

Atualmente o CI testa:

| Distribuição | Container |
|---|---|
| Alpine Linux | `alpine:latest` |
| Arch Linux | `archlinux:latest` |
| Debian Stable | `debian:stable` |
| Debian Testing | `debian:testing` |
| Ubuntu 24.04 | `ubuntu:24.04` |
| Ubuntu Latest | `ubuntu:latest` |
| Fedora | `fedora:latest` |
| Rocky Linux 9 | `rockylinux:9` |
| AlmaLinux | `almalinux:latest` |
| Oracle Linux 9 | `oraclelinux:9` |
| openSUSE Tumbleweed | `opensuse/tumbleweed:latest` |
| Void Linux | `voidlinux/voidlinux:latest` |
| Gentoo | `gentoo/stage3:latest` |

A matriz pode ser expandida posteriormente para outras distribuições.

## Como funciona o GitHub Actions

O workflow está em:

```text
.github/workflows/ci.yml
```

O GitHub Actions executa o workflow quando ocorre um:

- `push`;
- `pull_request`.

O workflow cria um job para cada distribuição definida na matriz.

A ideia central é esta:

```yaml
strategy:
  matrix:
    include:
      - name: Alpine
        image: alpine:latest
      - name: Arch Linux
        image: archlinux:latest
      - name: Debian Stable
        image: debian:stable
```

O GitHub Actions pega essas entradas e executa o mesmo procedimento para cada uma delas.

Assim, em vez de escrever vários jobs manualmente, podemos definir uma única estrutura e fornecer diferentes ambientes.

## Por que usar Docker?

O GitHub Actions fornece uma máquina runner para executar o workflow.

Neste projeto, o runner executa o Docker e cada teste acontece dentro do container correspondente à distribuição.

Simplificando:

```text
GitHub Actions Runner
        │
        ├── Alpine container
        │       └── test.sh
        │
        ├── Arch container
        │       └── test.sh
        │
        ├── Debian container
        │       └── test.sh
        │
        ├── Ubuntu container
        │       └── test.sh
        │
        └── ...
```

Isso permite testar o mesmo arquivo em ambientes diferentes sem precisar criar uma máquina virtual para cada distribuição.

## Uma observação importante

O `actions/checkout` acontece no runner do GitHub.

Depois disso, o repositório é montado no container:

```yaml
docker run --rm \
  -v "$GITHUB_WORKSPACE:/workspace" \
  -w /workspace \
  "${{ matrix.image }}" \
  /bin/sh src/test.sh
```

O comando final é executado dentro da distribuição selecionada.

Isso significa que o teste efetivamente utiliza o `/bin/sh` fornecido pelo ambiente que está sendo testado.

## Criando seu próprio CI

Se você quiser reproduzir este exemplo em outro projeto, a estrutura mínima pode ser:

```text
meu-projeto/
├── .github/
│   └── workflows/
│       └── ci.yml
└── src/
    └── test.sh
```

### 1. Crie o script

Por exemplo:

```sh
#!/bin/sh

printf '%s\\n' "Teste executado com sucesso"
```

Salve como:

```text
src/test.sh
```

### 2. Crie o workflow

Crie:

```text
.github/workflows/ci.yml
```

Um exemplo mínimo:

```yaml
name: POSIX compatibility

on:
  push:
  pull_request:

jobs:
  test:
    name: ${{ matrix.name }}
    runs-on: ubuntu-latest

    strategy:
      fail-fast: false
      matrix:
        include:
          - name: Alpine
            image: alpine:latest
          - name: Debian
            image: debian:stable
          - name: Ubuntu
            image: ubuntu:latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Run POSIX script
        run: |
          docker run --rm \
            -v "$GITHUB_WORKSPACE:/workspace" \
            -w /workspace \
            "${{ matrix.image }}" \
            /bin/sh src/test.sh
```

### 3. Faça o push

Depois de enviar o projeto para o GitHub, o workflow será executado automaticamente.

No repositório, abra:

```text
Actions
```

Você verá os testes sendo executados separadamente.

## O que é uma matrix?

A `matrix` é uma das partes mais úteis deste exemplo.

Em vez de escrever:

```yaml
job-alpine:
  ...

job-debian:
  ...

job-ubuntu:
  ...
```

podemos definir:

```yaml
matrix:
  include:
    - name: Alpine
      image: alpine:latest

    - name: Debian
      image: debian:stable

    - name: Ubuntu
      image: ubuntu:latest
```

O GitHub Actions transforma essas entradas em execuções independentes.

Isso torna muito mais fácil adicionar novos ambientes.

Quer testar Fedora?

Adicione:

```yaml
- name: Fedora
  image: fedora:latest
```

Pronto.

Mais uma distribuição entrou para o laboratório.

## Por que `fail-fast: false`?

O workflow utiliza:

```yaml
fail-fast: false
```

Isso faz com que uma falha não interrompa imediatamente os outros testes da matriz.

Isso é especialmente útil quando estamos comparando ambientes.

Imagine que Alpine falhe.

Ainda queremos saber se:

- Debian passou;
- Ubuntu passou;
- Fedora passou;
- Arch passou;
- e assim por diante.

Uma falha deve nos dar **informação**, não esconder o resultado dos outros testes.

## E se uma distribuição falhar?

Primeiro, não assuma que o script está errado.

Uma falha pode acontecer por vários motivos:

- o script realmente não é compatível;
- a imagem Docker mudou;
- uma tag deixou de existir;
- a distribuição alterou algum comportamento;
- o container não conseguiu ser iniciado;
- houve um problema externo no registry.

Durante o desenvolvimento deste projeto, por exemplo, duas imagens inicialmente falharam porque as tags utilizadas não existiam como esperado.

O problema não estava no script.

Depois de ajustar as imagens para versões válidas, o CI passou.

Essa é uma das vantagens de ter o teste automatizado: ele mostra **onde** o problema realmente aconteceu.

## O que este projeto ensina?

Mesmo sendo pequeno, este projeto demonstra vários conceitos importantes:

```text
Código
  │
  ▼
Git commit
  │
  ▼
GitHub
  │
  ▼
GitHub Actions
  │
  ▼
Matrix
  │
  ├── Distro 1
  ├── Distro 2
  ├── Distro 3
  ├── ...
  └── Distro 13
  │
  ▼
Resultado
```

A partir daqui, o mesmo conceito pode ser aplicado a projetos muito maiores.

Por exemplo:

- testes automatizados;
- múltiplas versões de Python;
- múltiplas versões de Node.js;
- diferentes versões de bancos de dados;
- testes de compilação;
- lint;
- testes unitários;
- testes de integração;
- compatibilidade entre sistemas operacionais.

## Estrutura do projeto

```text
linux-distro-ci-demo/
├── .github/
│   └── workflows/
│       └── ci.yml
├── src/
│   └── test.sh
├── LICENSE
└── README.md
```

## Resultado

O objetivo final deste projeto é bastante simples:

> **Escrever uma vez, testar em vários ambientes e deixar o CI verificar o resto.**

No nosso caso, o teste é tão pequeno que provavelmente levaria menos tempo para executá-lo manualmente do que para escrever este README.

Mas esse é justamente o ponto.

Se até um script de poucas linhas pode ser usado para demonstrar CI, fica muito mais fácil entender como a mesma ideia escala para projetos reais.

E assim nasceu o:

**Ryouta Linux Compatibility Laboratory™**

Não existe laboratório.

Não existe patente.

Mas existem 13 distribuições Linux verificando um `printf`.

Isso já é suficientemente científico.

---

## Licença

Este projeto está disponível sob a licença **MIT**.

Consulte o arquivo [LICENSE](LICENSE) para os termos completos.
