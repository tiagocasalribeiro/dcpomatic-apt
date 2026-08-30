# Repositório APT do DCP-o-matic para Debian 13

Um repositório APT **não oficial** do [DCP-o-matic](https://dcpomatic.com) para o
Debian 13 (Trixie), alojado inteiramente no GitHub e mantido de forma automática
pelo GitHub Actions.

Todos os dias, o fluxo de trabalho verifica se foi lançada uma nova versão
estável do DCP-o-matic e, em caso afirmativo, descarrega o pacote `.deb`
oficial, publica-o numa *release* do GitHub e actualiza os metadados APT
servidos pelo GitHub Pages.

---

## Índice

- [Como funciona](#como-funciona)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Requisitos](#requisitos)
- [Instalação e configuração](#instalação-e-configuração)
- [Utilização no cliente](#utilização-no-cliente)
- [Assinatura GPG (opcional)](#assinatura-gpg-opcional)
- [Resolução de problemas](#resolução-de-problemas)
- [Limitações](#limitações)
- [Manutenção do fluxo](#manutenção-do-fluxo)
- [Aviso legal e créditos](#aviso-legal-e-créditos)

---

## Como funciona

1. **Verificação diária** — uma tarefa cron (`0 0 * * *`) executa o fluxo de
   trabalho todos os dias à meia-noite (UTC). Também é possível executá-lo
   manualmente (*Run workflow*).
2. **Detecção da versão** — em vez de analisar a página
   `dcpomatic.com/download` (protegida pelo Cloudflare, que bloqueia os IPs dos
   servidores do GitHub), o fluxo consulta a API oficial do projecto
   [`cth103/dcpomatic`](https://github.com/cth103/dcpomatic) e selecciona a
   etiqueta (*tag*) estável mais recente. O DCP-o-matic segue um ciclo de
   versões em que o número menor **par** é estável (p. ex. `2.18.x`) e o
   **ímpar** é de desenvolvimento (p. ex. `2.19.x`).
3. **Descarregamento do pacote** — o `.deb` para Debian 13 (amd64) é
   descarregado de `dcpomatic.com` com a biblioteca Python `curl_cffi`, que
   imita a assinatura TLS do Chrome e atravessa o Cloudflare. O fluxo reproduz
   os três passos da página de doação:
   1. pedido inicial com `paid=0` (obtém o cookie de sessão);
   2. segundo pedido com o cookie (página «Your download should start soon»);
   3. pedido final **sem** o parâmetro `paid`, que devolve o ficheiro binário.
4. **Publicação do pacote** — como o `.deb` (~200 MB) excede o limite de
   100 MB por ficheiro imposto pelo Git, é anexado a uma *release* do GitHub
   (limite de 2 GB por asset), com a etiqueta `v<VERSÃO>`.
5. **Metadados APT** — os ficheiros `Packages`, `Packages.gz` e `Release` são
   gerados com `dpkg-scanpackages` e `apt-ftparchive`. O campo `Filename:` do
   `Packages` contém o URL **absoluto** do asset na release, pelo que o `apt`
   descarrega o pacote directamente daí. Os metadados são publicados no ramo
   `gh-pages` (GitHub Pages).
6. **Limpeza** — são mantidas apenas as três *releases* mais recentes, para
   não consumir o espaço de armazenamento da conta.

Se não houver nova versão, o fluxo termina sem fazer qualquer alteração.

## Estrutura do repositório

Ramo principal (`main`):

```
dcpomatic-apt/
├── .github/
│   └── workflows/
│       └── apt-repo.yml        # fluxo de actualização automática
└── README.md
```

Ramo `gh-pages` (criado e mantido pelo fluxo):

```
gh-pages/
├── Packages        # índice de pacotes (Filename: com URL absoluto)
├── Packages.gz     # índice comprimido
├── Release         # metadados e somas de verificação
└── version.txt     # última versão publicada
```

## Instalação no cliente

Cria o ficheiro `/etc/apt/sources.list.d/dcpomatic.list`:

```bash
echo "deb [trusted=yes] https://tiagocasalribeiro.github.io/dcpomatic-apt/ ./" \
  | sudo tee /etc/apt/sources.list.d/dcpomatic.list
```

> **Nota:** o `./` no fim é obrigatório — trata-se de um repositório *flat*
> (sem a estrutura `dists/`). O `[trusted=yes]` é necessário porque o
> repositório não está assinado com GPG por omissão (ver
> [Assinatura GPG](#assinatura-gpg-opcional)).

### Formato deb822 (Debian 13)

Em alternativa, cria `/etc/apt/sources.list.d/dcpomatic.sources`:

```
Types: deb
URIs: https://tiagocasalribeiro.github.io/dcpomatic-apt/
Suites: ./
Trusted: yes
```

### Instalar e actualizar

```bash
sudo apt update
sudo apt install dcpomatic
```

Para verificar a origem do pacote:

```bash
apt-cache policy dcpomatic
```

As dependências do DCP-o-matic são resolvidas automaticamente a partir dos
repositórios oficiais do Debian 13. As actualizações chegam com o habitual:

```bash
sudo apt update && sudo apt upgrade
```

## Assinatura GPG (opcional)

Para assinar o repositório e remover o `[trusted=yes]`:

1. Gera uma chave e exporta-a:

   ```bash
   gpg --full-gen-key
   gpg --export-secret-keys --armor <ID_DA_CHAVE> > dcpomatic-apt.asc
   ```

2. Guarda o conteúdo de `dcpomatic-apt.asc` num segredo do repositório
   (**Settings → Secrets and variables → Actions → New repository secret**),
   p. ex. `GPG_SECRET`.

3. Acrescenta ao fluxo, antes do *commit*:

   ```yaml
   - name: Assinar o repositório
     working-directory: ./apt-repo
     env:
       GPG_SECRET: ${{ secrets.GPG_SECRET }}
     run: |
       echo "$GPG_SECRET" | gpg --batch --import
       gpg --batch --yes --default-key <ID_DA_CHAVE> --clearsign -o InRelease Release
       gpg --batch --yes --default-key <ID_DA_CHAVE> --detach-sign --armor -o Release.gpg Release
       git add -f InRelease Release.gpg
   ```

4. Publica a chave pública e ajusta a linha do cliente para
   `deb [signed-by=/usr/share/keyrings/dcpomatic.gpg] ...`.

## Resolução de problemas

| Sintoma | Causa provável | Solução |
| --- | --- | --- |
| `apt update` devolve `404 Not Found` | Pages inactivo ou `gh-pages` ainda sem ficheiros | Verificar **Settings → Pages** e o conteúdo do ramo `gh-pages` |
| `NO_PUBKEY` / assinatura não verificável | Repositório não assinado | Usar `[trusted=yes]` ou configurar a assinatura GPG |
| Fluxo falha no descarregamento | O Cloudflare alterou o desafio ou a página de doação mudou | Consultar os registos da execução; poderá ser necessário ajustar o `impersonate` ou os passos do pedido |
| `This exceeds GitHub's file size limit of 100 MB` | O `.deb` foi cometido para o Git | O `.deb` deve ir **apenas** para a release, nunca para o `gh-pages` |
| A versão não muda apesar de novo lançamento | Atraso do cron ou etiqueta ainda não criada no GitHub | Executar o fluxo manualmente |
| *Release* já existe ao reexecutar | Reexecução manual | O fluxo apaga e recria a release automaticamente |

## Limitações

- Apenas a arquitectura **amd64** (x86) do Debian 13 é publicada por omissão.
  Para ARM ou a variante CLI, ver [Manutenção do fluxo](#manutenção-do-fluxo).
- O repositório **não é oficial** e não está associado ao autor do
  DCP-o-matic; os pacotes são redistribuídos tal como publicados em
  `dcpomatic.com`.
- O descarregamento depende da estrutura actual da página de doação e do
  comportamento do Cloudflare; alterações a qualquer um dos dois podem obrigar
  a ajustar o fluxo.
- Por omissão, são conservadas apenas as três *releases* mais recentes.

## Manutenção do fluxo

- **Outras arquitecturas/variantes** — altera o identificador no URL de
  descarregamento (`id=debian-13-x86`) para `debian-13-arm` (ARM) ou
  `debian-13-x86-cli` (apenas linha de comandos), e adapta o nome do ficheiro
  e o passo de geração de metadados.
- **Número de releases conservadas** — ajusta o índice `releases[3:]` no passo
  de limpeza.
- **Frequência da verificação** — altera a expressão cron no gatilho
  `schedule`.

## Aviso legal e créditos

- **DCP-o-matic** © Carl Hetherington ([cth103](https://github.com/cth103)),
  disponibilizado sob a GNU General Public License.
  <https://dcpomatic.com>
- Este repositório é um esforço independente de empacotamento, criado para
  facilitar a instalação e a actualização do DCP-o-matic em Debian 13.
  Todos os nomes de produtos e marcas pertencem aos respectivos proprietários.
- Os scripts e o fluxo de trabalho deste repositório são disponibilizados tal
  como estão, sem qualquer garantia.
