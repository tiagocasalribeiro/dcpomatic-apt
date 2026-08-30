# Repositório APT do DCP-o-matic para Debian 13

Repositório APT não oficial do [DCP-o-matic](https://dcpomatic.com) para o Debian 13 (Trixie), mantido de forma automática pelo GitHub Actions.

O fluxo verifica diariamente as páginas de downloads do DCP-o-matic e publica automaticamente as novas versões. Estão disponíveis dois canais:

| Canal | Origem | Descrição |
| --- | --- | --- |
| `stable` | `Stable release:` em [dcpomatic.com/download](https://dcpomatic.com/download) | Versão estável recomendada |
| `testing` | `Test release:` em [dcpomatic.com/test-download](https://dcpomatic.com/test-download) | Versão de desenvolvimento |

> **Nota:** Este projecto não está associado ao autor do DCP-o-matic.

## Instalação

### 1. Instalar a chave pública

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://tiagocasalribeiro.github.io/dcpomatic-apt/dcpomatic-apt.asc \
  | sudo gpg --dearmor -o /etc/apt/keyrings/dcpomatic-apt.gpg
```

### 2. Adicionar o canal pretendido

**Stable** (recomendado):

```bash
sudo tee /etc/apt/sources.list.d/dcpomatic-stable.sources << 'EOF'
Types: deb
URIs: https://tiagocasalribeiro.github.io/dcpomatic-apt/
Suites: stable
Components: main
Signed-By: /etc/apt/keyrings/dcpomatic-apt.gpg
EOF
```

**Testing** (desenvolvimento):

```bash
sudo tee /etc/apt/sources.list.d/dcpomatic-testing.sources << 'EOF'
Types: deb
URIs: https://tiagocasalribeiro.github.io/dcpomatic-apt/
Suites: testing
Components: main
Signed-By: /etc/apt/keyrings/dcpomatic-apt.gpg
EOF
```

### 3. Actualizar e instalar

```bash
sudo apt update
sudo apt install dcpomatic
```

## Usar os dois canais em simultâneo

Como ambos os canais contêm um pacote com o mesmo nome (`dcpomatic`), se activares os dois, o `apt` escolherá por omissão a versão mais recente (testing).

Para dar prioridade a um canal, cria um ficheiro de pinning:

```bash
sudo tee /etc/apt/preferences.d/dcpomatic << 'EOF'
Package: dcpomatic
Pin: release a=stable
Pin-Priority: 900

Package: dcpomatic
Pin: release a=testing
Pin-Priority: 500
EOF
```

## Actualização

```bash
sudo apt update && sudo apt upgrade
```

Para instalar uma versão específica:

```bash
sudo apt install dcpomatic/stable     # versão estável
sudo apt install dcpomatic/testing    # versão de teste
```

## Resolução de problemas

| Sintoma | Solução |
| --- | --- |
| `404 Not Found` no `apt update` | O GitHub Pages pode ainda não estar pronto; aguarda alguns minutos |
| `NO_PUBKEY` | Volta a executar o passo 1 da instalação |
| O pacote não aparece | Executa `sudo apt update` primeiro |
| `404` ao instalar | Limpa a cache: `sudo rm -rf /var/lib/apt/lists/* && sudo apt update` |

## Estrutura do repositório

```
dcpomatic-apt/
├── dcpomatic-apt.asc
├── stable-version.txt
├── testing-version.txt
├── pool/
│   ├── stable/main/d/dcpomatic/
│   │   └── dcpomatic_<versão>_amd64.deb
│   └── testing/main/d/dcpomatic/
│       └── dcpomatic_<versão>_amd64.deb
└── dists/
    ├── stable/
    │   ├── InRelease, Release, Release.gpg
    │   └── main/binary-amd64/Packages(.gz)
    └── testing/
        ├── InRelease, Release, Release.gpg
        └── main/binary-amd64/Packages(.gz)
```

## Créditos

DCP-o-matic © Carl Hetherington, sob licença GNU GPL.
<https://dcpomatic.com>
