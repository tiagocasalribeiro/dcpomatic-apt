# Repositório APT do DCP-o-matic para Debian 13

Repositório APT não oficial do [DCP-o-matic](https://dcpomatic.com) para o
Debian 13 (Trixie), mantido automaticamente pelo GitHub Actions. Todos os
dias é verificada a existência de uma nova versão estável e, se houver, o
pacote é publicado aqui.

> **Nota:** Este repositório não está associado ao autor do DCP-o-matic.

## Instalação

1. Descarrega e instala a chave pública:

   ```bash
   sudo mkdir -p /etc/apt/keyrings
   curl -fsSL https://tiagocasalribeiro.github.io/dcpomatic-apt/dcpomatic-apt.asc \
     | sudo gpg --dearmor -o /etc/apt/keyrings/dcpomatic-apt.gpg
   ```

2. Adiciona o repositório:

   ```bash
   echo "deb [signed-by=/etc/apt/keyrings/dcpomatic-apt.gpg] https://tiagocasalribeiro.github.io/dcpomatic-apt/ ./" \
     | sudo tee /etc/apt/sources.list.d/dcpomatic.list
   ```

3. Actualiza e instala:

   ```bash
   sudo apt update
   sudo apt install dcpomatic
   ```

## Actualização

```bash
sudo apt update && sudo apt upgrade
```

## Resolução de problemas

| Sintoma | Solução |
| --- | --- |
| `404 Not Found` no `apt update` | O GitHub Pages pode ainda não estar pronto; aguarda alguns minutos |
| `NO_PUBKEY` | Volta a executar o passo 1 da instalação |
| O pacote não aparece | Executa `sudo apt update` primeiro |

## Créditos

DCP-o-matic © Carl Hetherington, sob licença GNU GPL.
<https://dcpomatic.com>
