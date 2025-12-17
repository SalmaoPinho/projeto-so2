# Linux From Scratch (LFS) 12.4 - Custom USB Bootable Build

![Linux](https://img.shields.io/badge/Linux-6.18.0-blue?logo=linux)
![LFS](https://img.shields.io/badge/LFS-12.4-orange)
![GRUB](https://img.shields.io/badge/Bootloader-GRUB2-red)
![License](https://img.shields.io/badge/License-Educational-green)

##  Visão Geral

Este projeto consiste no desenvolvimento de um sistema operacional **Linux From Scratch (LFS) versão 12.4**, compilado inteiramente a partir do código-fonte. Diferente de distribuições convencionais ou versões simplificadas com BusyBox, este sistema possui um ambiente **GNU/Linux completo**, com suporte a bibliotecas dinâmicas, gerenciamento de dispositivos via Udev e um Initramfs customizado para garantir portabilidade em hardware físico via USB.

##  Status do Projeto

- [x] Compilação do Toolchain (Binutils, GCC 15.2, Glibc 2.39)
- [x] Construção do Sistema Base (Ambiente GNU completo)
- [x] Configuração de Boot (GRUB 2.12 + Kernel 6.18.0)
- [x] **Initramfs Customizado** para montagem via UUID
- [x] **Blindagem de Hardware** (Correção de Framebuffer/Vídeo e USB)
- [x] Testado e validado em QEMU e Hardware Real (Laptop/Desktop)

##  Especificações Técnicas

| Componente | Versão | Decisão de Design |
|------------|--------|-------------------|
| **Kernel** | Linux 6.18.0 | Monolítico com drivers USB/SATA embutidos e suporte DRM |
| **Userland** | GNU Toolchain | Glibc completa para suporte a software moderno |
| **Init** | SysVinit 3.14 | Sistema de iniciação clássico e estável |
| **Shell** | Bash 5.3 | Shell padrão rico em funcionalidades |
| **Bootloader** | GRUB 2.12 | Instalado com suporte a busca de partição por UUID |
| **Filesystem** | Ext4 | Partição raiz com journaling para resiliência |

##  Desafios e Engenharia de Soluções

A transição para hardware real apresentou obstáculos que exigiram depuração profunda no Kernel e no processo de iniciação:

### 1. O Problema do "Kernel Panic" em USB
**Desafio:** O Kernel inicializava mais rápido do que o barramento USB era capaz de detectar o pendrive, resultando em `Unable to mount root fs`.
**Solução:** Implementação de um **Initramfs customizado** (`initrd.img-6.18`). Este micro-sistema carrega os módulos de armazenamento antecipadamente, pausa a execução até que o hardware USB esteja pronto e utiliza o **UUID** da partição em vez de nomes estáticos (como /dev/sda1), garantindo que o sistema boote mesmo se houver outros HDs na máquina.

### 2. O Bug da Tela Preta (Graphics Stack)
**Desafio:** O sistema iniciava com sucesso, mas a tela apagava após o carregamento do Kernel devido à falta de uma ponte entre o driver gráfico e o console.
**Solução:** Recompilação do Kernel habilitando o suporte a **Framebuffer Console**.

As opções críticas ativadas foram:
- `CONFIG_DRM_FBDEV_EMULATION=y` (Emula um dispositivo de vídeo para o console)
- `CONFIG_FRAMEBUFFER_CONSOLE=y` (Permite desenhar texto na placa de vídeo)
- `CONFIG_DRM_SIMPLEDRM=y` (Garante vídeo básico caso o driver específico falhe)

##  Estrutura de Arquivos Críticos

- `mkinitramfs`: Script shell para gerar o disco RAM inicial com suporte a UUID.
- `/boot/config-6.18`: Configuração final do Kernel (drivers USB, SCSI e Vídeo embutidos).
- `grub.cfg`: Configuração do bootloader com parâmetros `rootdelay=10` e `search --fs-uuid`.
- `/etc/fstab`: Tabela de montagem corrigida para sistemas de arquivos virtuais.

##  Como Testar

### Passo 1: Download da Imagem
Baixe a imagem compactada do sistema (aprox. 16GB descompactada, mas otimizada para download):
**[Download imagem-lfs.img.gz (Google Drive)](https://drive.google.com/file/d/1coWdT8cMxbqRNr1auo0IqWwlU3_etKE8/view?usp=sharing)**

### Passo 2: Emulação (QEMU)

# Teste via comando direto (simulando hardware real)
qemu-system-x86_64 -enable-kvm -m 2G -drive file=imagem-lfs.img,format=raw

##  Conclusão

O sistema final é **funcional**, **compacto** e cumpre os requisitos de ser um boot minimalista e monolítico em arquitetura x86_64. O projeto demonstra domínio completo sobre os componentes fundamentais de um sistema operacional Linux.
