# Sistema Operacional Linux Minimalista Embarcado

![Linux](https://img.shields.io/badge/Linux-6.6.30-blue?logo=linux)
![BusyBox](https://img.shields.io/badge/BusyBox-1.36.1-orange)
![GRUB](https://img.shields.io/badge/Bootloader-GRUB2-red)
![License](https://img.shields.io/badge/License-Educational-green)

## 📋 Visão Geral

Sistema Operacional Linux minimalista e monolítico desenvolvido do zero, com foco em **gerenciamento de arquivos mono-usuário**. O sistema foi otimizado para inicializar via **BIOS Legacy (MBR)** a partir de um pendrive, garantindo máxima portabilidade e desempenho mínimo.

Este projeto demonstra a capacidade de construir um sistema operacional funcional aplicando princípios rigorosos de engenharia de sistemas embarcados, contornando falhas críticas de hardware e firmware.

## 🎯 Objetivo do Projeto

Desenvolver um SO Linux minimalista e monolítico capaz de:
- ✅ Inicializar a partir de dispositivos USB (pendrive)
- ✅ Operar com recursos mínimos de hardware
- ✅ Fornecer gerenciamento básico de arquivos
- ✅ Executar em modo mono-usuário sem dependências externas

## 🏗️ Arquitetura e Componentes

| Componente | Versão | Decisão de Design |
|------------|--------|-------------------|
| **Kernel** | Linux 6.6.30 (LTS) | Compilado estaticamente com drivers essenciais embutidos (Built-in) |
| **Userland** | BusyBox 1.36.1 | Linkagem Estática (`CONFIG_STATIC=y`) para eliminar dependências de bibliotecas (`glibc`) |
| **Bootloader** | GRUB2 | Instalado na MBR, com configuração de compatibilidade blindada |
| **Filesystem** | Ext4 | Padrão robusto para a partição raiz (`/dev/sdb1`) |
| **Ambiente Host** | Ubuntu | Utilizado para cross-compilation e sandbox via QEMU/Loopback |

## ⚙️ Otimizações Implementadas

### BusyBox - Máximo Minimalismo

A userland foi submetida a um regime de **máximo minimalismo** para reduzir o binário final e a pegada de memória:

#### 🔹 Linkagem Estática
- Compilado com `CONFIG_STATIC=y`
- Binário compacto de **~1.6 MB**
- Não requer bibliotecas externas em `/lib`

#### 🔹 Limpeza Agressiva
- ❌ Desabilitação completa de subsistemas de rede
- ❌ Remoção de autenticação (`login`, `passwd`)
- ❌ Exclusão de editores complexos (`vi`, `awk`)
- ✅ Mantidos apenas comandos básicos de I/O (`ls`, `cp`, `mount`)

#### 🔹 Inicialização
- Configurado com `/etc/init.d/rcS`
- Montagem automática de `/proc` e `/sys`
- Início do shell interativo

## 🛡️ Kernel: Blindagem de Hardware e Portabilidade

A estabilidade no hardware físico foi atingida após correção dos drivers essenciais e parâmetros de boot, resolvendo falhas que ocorriam no hardware físico (apesar do sucesso no QEMU).

### Drivers Críticos (Built-in `[*]`)

Para evitar falhas como **"Kernel Panic: VFS: Unable to mount root fs"**, os seguintes drivers foram compilados de forma **estática/embutida**:

#### 💾 Disco USB/SATA
- `USB Mass Storage support`
- `AHCI SATA support`
- `Generic ATA/PATA support`

#### 🔌 Barramento USB
- `xHCI HCD` (USB 3.0)
- `EHCI HCD` (USB 2.0)

#### 🖥️ Vídeo (Anti-Tela Preta)
- `VESA VGA graphics support`
- `Framebuffer support`

### Correção de Boot (GRUB) - Solução Definitiva

A solução para a **tela preta** e a **falha de montagem** foi injetada diretamente na linha de comando do Kernel via GRUB:

| Parâmetro | Finalidade |
|-----------|------------|
| `root=/dev/sdb1` | **Endereço Definitivo**: Força o Kernel a montar a partição raiz no segundo disco (`/dev/sdb`), a posição provável do pendrive em um PC com HD interno (`/dev/sda`) |
| `rootwait` | **Latência USB**: Instrução para o Kernel aguardar a inicialização do barramento USB antes de tentar montar a partição raiz |
| `nomodeset` | **Estabilidade de Vídeo**: Impede que o Kernel carregue drivers gráficos complexos, forçando o uso do modo de vídeo básico VESA |
| `vga=normal` | **Visualização**: Força o modo de texto clássico |
| `console=tty0` | **Output**: Envia o output do console para o monitor físico |

### Configuração Final de Boot

Arquivo `grub.cfg` responsável pelo boot bem-sucedido:

```grub
menuentry "LFS" {
    linux /boot/vmlinuz-lfs-minimal root=/dev/sdb1 rw rootwait nomodeset vga=normal console=tty0
}
```

## 🚀 Como Usar

### Pré-requisitos

- Pendrive (mínimo 512 MB)
- Computador com BIOS Legacy (MBR)
- Arquivo `lfs-boot.img` disponível neste repositório

### Instalação

1. **Grave a imagem no pendrive**:
   ```bash
   sudo dd if=lfs-boot.img of=/dev/sdX bs=4M status=progress
   sync
   ```
   > ⚠️ **Atenção**: Substitua `/dev/sdX` pelo dispositivo correto do seu pendrive

2. **Configure a BIOS**:
   - Acesse a BIOS do computador
   - Configure o boot para **Legacy/MBR mode**
   - Defina o pendrive como primeiro dispositivo de boot

3. **Inicialize o sistema**:
   - Reinicie o computador
   - O sistema deve carregar automaticamente via GRUB

## 📁 Estrutura do Projeto

```
projeto-so2/
├── README.md              # Este arquivo
├── Documentacao.pdf       # Documentação técnica completa
└── lfs-boot.img          # Imagem bootável do sistema
```

## 🔧 Desenvolvimento

### Ambiente de Compilação

O sistema foi desenvolvido utilizando:
- **Host OS**: Ubuntu
- **Cross-compilation**: GCC toolchain
- **Testing**: QEMU para testes em sandbox
- **Loopback devices**: Para montagem e modificação da imagem

### Processo de Build

O sistema foi construído seguindo a metodologia **Linux From Scratch (LFS)**, incluindo:

1. Compilação do Kernel Linux 6.6.30
2. Build do BusyBox com linkagem estática
3. Configuração do sistema de arquivos Ext4
4. Instalação e configuração do GRUB2
5. Otimização e testes de compatibilidade

## 🐛 Troubleshooting

### Problema: Tela Preta no Boot
**Solução**: Adicione `nomodeset vga=normal` aos parâmetros do kernel no GRUB

### Problema: Kernel Panic - Unable to mount root fs
**Solução**: Verifique se os drivers USB estão compilados como built-in e adicione `rootwait` aos parâmetros de boot

### Problema: Sistema não encontra o pendrive
**Solução**: Ajuste o parâmetro `root=/dev/sdb1` para o dispositivo correto (pode ser `/dev/sdc1` dependendo da configuração)

## 📊 Especificações Técnicas

- **Arquitetura**: x86_64
- **Tamanho do Binário BusyBox**: ~1.6 MB
- **Modo de Boot**: BIOS Legacy (MBR)
- **Tipo de Sistema**: Monolítico
- **Usuários**: Mono-usuário
- **Requisitos Mínimos**:
  - CPU: x86_64
  - RAM: 64 MB (mínimo)
  - Storage: 512 MB

## 🎓 Aprendizados

Este projeto demonstrou:
- ✅ Construção de um sistema operacional do zero
- ✅ Aplicação de princípios de engenharia de sistemas embarcados
- ✅ Resolução de falhas críticas de hardware (driver de disco/vídeo e latência USB)
- ✅ Resolução de conflitos de firmware (nomenclatura de disco)
- ✅ Otimização extrema de recursos

## 📝 Conclusão

O sistema final é **funcional**, **compacto** e cumpre os requisitos de ser um boot minimalista e monolítico em arquitetura x86_64. O projeto demonstra domínio completo sobre os componentes fundamentais de um sistema operacional Linux.

## 📄 Licença

Este projeto foi desenvolvido para fins educacionais como parte do curso de Sistemas Operacionais 2.

---

**Desenvolvido com** ❤️ **para aprendizado de Sistemas Operacionais**
