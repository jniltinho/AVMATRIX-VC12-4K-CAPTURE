# Pacotes .deb (DKMS) para Ubuntu

O driver `HwsUHDX1Capture` (AVMATRIX VC12-4K) é empacotado como **DKMS**: o `.deb` instala o código-fonte em
`/usr/src/HwsUHDX1CaptureForLinux-<versão>` e o DKMS compila o módulo na máquina, para cada kernel instalado
que tenha headers, e recompila sozinho a cada atualização de kernel.

Versões suportadas: **Ubuntu 24.04** e **Ubuntu 26.04**.

## Baixar pronto

Os pacotes ficam na página de [Releases](https://github.com/jniltinho/AVMATRIX-VC12-4K-CAPTURE/releases).
Cada push na `main` também gera os pacotes como *artifacts* no GitHub Actions (workflow `Build .deb`).

## Gerar localmente

Requisitos: `make` e Docker com BuildKit (Docker 23+).

```sh
make              # gera os pacotes para 24.04 e 26.04 em ./out
make deb-24.04    # só Ubuntu 24.04
make deb-26.04    # só Ubuntu 26.04
make clean        # apaga ./out
```

Variáveis: `UBUNTU_VERSIONS` (padrão `24.04 26.04`) e `OUT` (padrão `out`), por exemplo
`make UBUNTU_VERSIONS=26.04 OUT=/tmp/debs`.

Sem `make`, o comando equivalente é:

```sh
docker build -f packaging/Dockerfile --build-arg UBUNTU_VERSION=24.04 --output out .
```

Durante o build o módulo é compilado contra os headers do kernel *generic* da versão do Ubuntu
escolhida; se o driver não compilar, o pacote não é gerado.

## Instalar

```sh
sudo apt install linux-headers-$(uname -r)
sudo apt install ./out/hws-uhdx1-capture-dkms_*~ubuntu24.04_all.deb   # use o arquivo da sua versão
```

Conferir:

```sh
dkms status                 # HwsUHDX1CaptureForLinux/..., <kernel>, x86_64: installed
lsmod | grep HwsUHDX1Capture
```

O pacote substitui o `install.sh`/`dkms-install.sh`; não use os dois ao mesmo tempo.

## Remover

```sh
sudo apt remove hws-uhdx1-capture-dkms
```

Remove o módulo de todos os kernels e o registro no DKMS.

## Publicar uma release

Criar e enviar uma tag `v*` dispara o workflow, que gera os dois pacotes e os anexa à release da tag:

```sh
git tag v1.0.0.230324-1
git push origin v1.0.0.230324-1
```

## Arquivos

| Arquivo | Função |
|---|---|
| `packaging/Dockerfile` | Compila o módulo (teste) e monta o `.deb` para a versão em `UBUNTU_VERSION` |
| `packaging/debian/` | `control`, `postinst` (dkms add/install) e `prerm` (dkms remove) |
| `Makefile` | Atalhos `make deb`, `make deb-<versão>`, `make clean` |
| `.github/workflows/build-deb.yml` | CI: gera os pacotes e publica nas releases de tags `v*` |
