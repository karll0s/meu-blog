---
title: "🐧 Guia Completo de Instalação do Arch Linux"
date: 2025-12-13
draft: false
tags: ["archlinux", "linux", "instalação", "guia"]
categories: ["Linux"]
author: "Karllos"
showToc: true
TocOpen: true
summary: "Guia detalhado de instalação do Arch Linux do zero, explicando cada etapa com calma e profundidade."
---

# 🐧 Guia Completo de Instalação do Arch Linux

> **Atenção:** Este é um guia detalhado para instalação do Arch Linux do zero. Recomendado para usuários intermediários a avançados que desejam entender cada etapa do processo.

## 📡 Configuração Inicial

Quando você inicializa o Arch Linux pela primeira vez através da ISO, você é apresentado a um terminal. Aqui começamos a configurar o ambiente básico para a instalação.

### Teclado e Conectividade

O primeiro passo é configurar o layout do teclado. Por padrão, o sistema vem com layout americano, então precisamos mudá-lo para o brasileiro.

```bash
# Configurar layout do teclado brasileiro (ABNT2)
# Isso garante que os caracteres especiais funcionem corretamente
loadkeys br-abnt2

# Testar a conexão com a internet (essencial para continuar)
# Use Ctrl+C para parar o ping após confirmar que funciona
ping google.com

# Desbloquear o WiFi caso esteja bloqueado por software
# Alguns laptops vêm com o WiFi bloqueado por padrão
rfkill unblock wifi
rfkill unblock all
```

**Por que isso é importante?** Sem internet, não conseguimos baixar os pacotes necessários para a instalação. O `rfkill` remove bloqueios de software que podem impedir o funcionamento do WiFi.

### Conectar ao WiFi

O Arch usa o `iwctl` (iwd control) para gerenciar conexões WiFi durante a instalação. É uma ferramenta interativa e bem simples de usar.

```bash
# Entrar no utilitário de WiFi
iwctl

# Listar dispositivos de rede disponíveis
# Geralmente você verá algo como "wlan0"
device list

# Escanear redes WiFi disponíveis
# Substitua "wlan0" pelo nome do seu dispositivo, se diferente
station wlan0 scan

# Listar as redes encontradas
station wlan0 get-networks

# Conectar a uma rede específica
# Substitua "nome-da-rede" pelo SSID da sua rede
# Você será solicitado a digitar a senha
station wlan0 connect "nome-da-rede"

# Sair do iwctl
exit
```

**Dica:** Se você estiver usando cabo ethernet, pode pular esta etapa. O Arch geralmente detecta e conecta automaticamente via cabo.

**Troubleshooting:** Se o WiFi não funcionar, verifique se a antena wireless está habilitada no BIOS/UEFI do seu computador.

---

## 💾 Particionamento do Disco

O particionamento é uma das etapas mais importantes e delicadas da instalação. Aqui definimos como o disco será organizado.

### Verificar e Particionar

Primeiro, precisamos identificar qual disco vamos usar e criar as partições necessárias.

```bash
# Listar todos os discos e partições disponíveis
# Isso ajuda a identificar qual disco será usado
lsblk

# Abrir o utilitário de particionamento para o disco principal
# /dev/vda em máquinas virtuais, /dev/sda ou /dev/nvme0n1 em físicas
cfdisk /dev/vda
```

**Interface do cfdisk:**

- Use as setas para navegar
- Selecione "gpt" como tipo de tabela de partição (para sistemas UEFI)
- Crie cada partição conforme a tabela abaixo

**Esquema de partições recomendado:**

| Partição | Tamanho | Tipo | Uso | Explicação |
| --- | --- | --- | --- | --- |
| `/dev/vda1` | 256 MB | EFI System | Bootloader (GRUB) | Armazena os arquivos de inicialização do sistema |
| `/dev/vda2` | 20 GB | Linux filesystem | Sistema (/) | Sistema operacional e programas instalados |
| `/dev/vda3` | Restante | Linux filesystem | Home (/home) | Seus arquivos pessoais, documentos, configurações |
| `/dev/vda4` | 8 GB | Linux swap | Swap | Memória virtual, importante para hibernação |

**Explicando cada partição:**

1. **EFI System (256 MB):** Necessária para sistemas UEFI modernos. É aqui que o GRUB (bootloader) será instalado. Sem ela, o sistema não inicializa.
  
2. **Sistema / (20 GB):** O "C:" do Linux. Aqui ficam o kernel, bibliotecas do sistema, e programas. 20 GB é suficiente para a maioria dos casos, mas se você planeja instalar muitos programas, considere 40-50 GB.
  
3. **Home (restante do disco):** Sua pasta pessoal. Separar o /home facilita reinstalações futuras - você pode reinstalar o sistema mantendo seus arquivos intactos.
  
4. **Swap (8 GB):** Funciona como memória RAM extra quando a física enche. O tamanho ideal é igual à sua RAM se você quiser usar hibernação, caso contrário, 2-4 GB são suficientes.
  

**Após criar todas as partições:**

- Selecione "Write" para salvar as alterações
- Digite "yes" para confirmar
- Selecione "Quit" para sair

**⚠️ AVISO:** O comando "Write" é DESTRUTIVO! Ele apaga todos os dados do disco. Certifique-se de estar no disco correto!

### Formatar Partições

Criar partições é apenas o primeiro passo. Agora precisamos formatá-las com sistemas de arquivos que o Linux entende.

```bash
# Verificar se as partições foram criadas corretamente
# Você deve ver vda1, vda2, vda3 e vda4
lsblk

# Formatar a partição EFI com FAT32
# O UEFI só reconhece FAT32, por isso usamos este formato
mkfs.fat -F32 /dev/vda1

# Formatar a partição do sistema com ext4
# ext4 é o sistema de arquivos padrão e mais estável do Linux
mkfs.ext4 /dev/vda2

# Formatar a partição home com ext4
mkfs.ext4 /dev/vda3

# Preparar a partição de swap
# Swap tem formato próprio, diferente dos sistemas de arquivos normais
mkswap /dev/vda4
```

**Entendendo os sistemas de arquivos:**

- **FAT32:** Sistema simples e universal, necessário para a partição EFI. Todo sistema UEFI sabe ler FAT32.
  
- **ext4 (Fourth Extended Filesystem):** O sistema de arquivos mais usado no Linux. Oferece excelente performance, confiabilidade e suporte a journaling (recuperação de falhas).
  
- **Swap:** Não é um "sistema de arquivos" tradicional, mas um espaço especial para memória virtual.
  

**Por que formatar?** Criar uma partição é como definir os limites de um terreno. Formatá-la é como construir as fundações de uma casa - você define como os dados serão organizados e armazenados.

---

## 🔧 Montar Partições

Agora que as partições estão formatadas, precisamos "montá-las" - ou seja, conectá-las a pontos específicos do sistema de arquivos para que possamos acessá-las e instalar o sistema nelas.

```bash
# Montar a partição raiz (/) em /mnt
# /mnt será o ponto de entrada do nosso novo sistema
mount /dev/vda2 /mnt

# Verificar se a montagem funcionou
ls /mnt

# Criar o diretório para a pasta home
# Precisamos criar manualmente pois o /mnt está vazio
mkdir /mnt/home

# Criar o diretório para a partição EFI
# -p cria todos os diretórios intermediários necessários
mkdir -p /mnt/boot/efi

# Verificar a estrutura criada
ls /mnt

# Montar a partição home
# Agora /mnt/home aponta para nossa partição separada
mount /dev/vda3 /mnt/home

# Montar a partição EFI
# O bootloader será instalado aqui
mount /dev/vda1 /mnt/boot/efi

# Ativar a partição swap
# Diferente das outras, swap não é "montada", mas "ativada"
swapon /dev/vda4

# Verificar tudo novamente
# Agora você deve ver [SWAP] ao lado da partição vda4
lsblk
```

**Entendendo a montagem:**

No Linux, tudo é um arquivo. Quando você "monta" uma partição, está dizendo: "esta partição física será acessível através deste diretório".

**Estrutura final:**

- `/mnt` → Partição do sistema (vda2)
- `/mnt/home` → Partição home (vda3)
- `/mnt/boot/efi` → Partição EFI (vda1)
- Swap (vda4) → Ativada em memória

**Por que essa ordem importa?** Você precisa montar a raiz primeiro, depois criar os subdiretórios, e só então montar as outras partições dentro deles. É como construir uma casa: primeiro a base, depois as divisões internas.

**Verificação importante:** O comando `lsblk` deve mostrar todas as partições montadas nos locais corretos. Se algo estiver errado, desmonte com `umount` e monte novamente.

---

## 🌐 Otimizar Mirrors

Os mirrors são servidores ao redor do mundo que hospedam os pacotes do Arch Linux. Escolher os mirrors mais rápidos acelera significativamente o download e atualização de pacotes.

```bash
# Sincronizar a lista de pacotes
# Isso atualiza o índice de pacotes disponíveis
pacman -Sy

# Instalar o reflector
# Ferramenta que testa e classifica mirrors automaticamente
pacman -S reflector

# Executar o reflector
# --latest 10: Testa os 10 mirrors atualizados mais recentemente
# --sort rate: Ordena por velocidade de download
# --verbose: Mostra o progresso detalhado
# --save: Salva o resultado no arquivo de configuração
reflector --latest 10 --sort rate --verbose --save /etc/pacman.d/mirrorlist

# Visualizar os mirrors selecionados (opcional)
# Você pode editar manualmente se quiser
nano /etc/pacman.d/mirrorlist

# Atualizar novamente com os novos mirrors
pacman -Sy
```

**Por que isso é importante?**

Imagine que você precise baixar 2 GB de pacotes. Com um mirror lento (500 KB/s), levaria mais de 1 hora. Com um mirror rápido (10 MB/s), apenas 3 minutos!

**Como funciona o reflector:**

1. Baixa a lista de todos os mirrors oficiais
2. Testa a velocidade de cada um deles
3. Remove os desatualizados ou offline
4. Organiza do mais rápido para o mais lento
5. Salva os melhores no arquivo de configuração

**Dica:** Se você quiser escolher mirrors de países específicos, use:

```bash
reflector --country Brazil,US --latest 10 --sort rate --save /etc/pacman.d/mirrorlist
```

**Entendendo o pacman:**

- `-S`: Sincronizar/instalar pacotes
- `-y`: Atualizar lista de pacotes
- `-u`: Atualizar todos os pacotes instalados
- `-Syu`: Comando completo para atualizar todo o sistema

---

## 📦 Instalação Base do Sistema

Aqui vamos instalar os pacotes fundamentais do Arch Linux. Este é o coração do sistema operacional.

```bash
# Instalar os pacotes base do sistema
# Este comando pode levar alguns minutos dependendo da sua conexão
pacstrap /mnt base base-devel linux linux-lts linux-headers linux-lts-headers linux-firmware nano vim
```

**Entendendo cada pacote:**

- **base:** Meta-pacote com os componentes essenciais do sistema (shell, utilitários básicos do GNU, etc.)
  
- **base-devel:** Ferramentas de desenvolvimento (compiladores, make, etc.). Essencial para compilar pacotes do AUR posteriormente.
  
- **linux:** O kernel Linux mais recente e estável. É o "cérebro" do sistema operacional.
  
- **linux-lts:** Kernel LTS (Long Term Support). Versão mais antiga, mas extremamente estável. Útil como fallback se o kernel normal apresentar problemas.
  
- **linux-headers:** Arquivos necessários para compilar drivers e módulos para o kernel principal.
  
- **linux-lts-headers:** Headers para o kernel LTS.
  
- **linux-firmware:** Firmware proprietário para diversos hardwares (WiFi, Bluetooth, placas de vídeo, etc.). Sem ele, muito hardware não funciona.
  
- **nano:** Editor de texto simples e amigável para iniciantes.
  
- **vim:** Editor de texto poderoso para usuários avançados.
  

**Por que instalar dois kernels?** Se uma atualização do kernel principal causar problemas (driver incompatível, bug, etc.), você pode inicializar pelo kernel LTS no GRUB e ter um sistema funcional enquanto resolve o problema.

**Tempo de instalação:** Dependendo da sua conexão, esta etapa pode levar de 5 a 20 minutos. Os pacotes totalizam aproximadamente 500-800 MB.

### Gerar fstab

O fstab (File System Table) é um arquivo de configuração crucial que diz ao sistema quais partições montar automaticamente na inicialização.

```bash
# Visualizar como ficaria o fstab
# Apenas para verificação, não salva nada
genfstab /mnt

# Gerar e salvar o fstab no novo sistema
# -U usa UUIDs em vez de nomes de dispositivos (mais confiável)
# >> adiciona ao arquivo sem sobrescrevê-lo
genfstab -U /mnt >> /mnt/etc/fstab

# Verificar o conteúdo do fstab gerado
# IMPORTANTE: confira se todas as partições estão listadas
cat /mnt/etc/fstab

# Verificar a estrutura do /mnt
ls /mnt
```

**Por que o fstab é importante?**

Sem o fstab, o sistema não saberia onde montar o /home, o /boot/efi ou ativar a swap. Cada vez que você reiniciasse, teria que montar tudo manualmente!

**O que são UUIDs?**

UUID (Universally Unique Identifier) é um identificador único para cada partição. Por exemplo:

- `/dev/sda1` pode mudar se você conectar outro disco
- `UUID=1234-5678-90AB` nunca muda, sempre identifica a mesma partição

**Exemplo de entrada no fstab:**

```
UUID=xxxx-xxxx-xxxx  /boot/efi  vfat  defaults  0  2
UUID=yyyy-yyyy-yyyy  /          ext4  defaults  0  1
UUID=zzzz-zzzz-zzzz  /home      ext4  defaults  0  2
UUID=wwww-wwww-wwww  none       swap  defaults  0  0
```

**Os números no final significam:**

- Primeiro número (0-2): Se fazer backup com dump
- Segundo número (0-2): Ordem de verificação no boot (0 = não verificar)

**⚠️ Verificação obrigatória:** Sempre confira o fstab com `cat`. Se estiver errado, o sistema não iniciará corretamente!

---

## ⚙️ Configuração do Sistema

Agora entraremos no sistema recém-instalado para configurá-lo. É como entrar pela primeira vez na casa que você acabou de construir para decorá-la e torná-la funcional.

### Entrar no chroot

```bash
# "Change root" - muda a raiz do sistema para /mnt
# Agora você está "dentro" do novo sistema, não mais no instalador
arch-chroot /mnt
```

**O que é chroot?**

Chroot é como criar uma "Matrix" dentro da Matrix. Você está fisicamente no ambiente de instalação, mas todos os comandos agora afetam o novo sistema em /mnt. É essencial para configurar o sistema antes de reiniciar.

### Configurar Pacman

O Pacman é o gerenciador de pacotes do Arch. Vamos deixá-lo mais bonito e funcional!

```bash
# Abrir o arquivo de configuração do pacman
nano /etc/pacman.conf
```

**Dentro do nano, descomentar (remover o #) ou adicionar:**

```ini
# Cores nos outputs (mais legível)
Color

# Downloads paralelos (mais rápido)
ParallelDownloads = 5

# Easter egg: animação do Pac-Man ao instalar
ILoveCandy

# Habilitar repositório multilib (programas 32-bit)
[multilib]
Include = /etc/pacman.d/mirrorlist
```

**Detalhando cada opção:**

- **Color:** Adiciona cores aos outputs do pacman, facilitando identificar erros, avisos e sucessos.
  
- **ParallelDownloads:** Por padrão, o pacman baixa um pacote por vez. Com esta opção, ele baixa até 5 simultaneamente, acelerando muito atualizações grandes.
  
- **ILoveCandy:** Substitui a barra de progresso chata por um Pac-Man comendo pontinhos. Porque não? 😄
  
- **[multilib]:** Repositório com versões 32-bit de bibliotecas. Necessário para jogos, Steam, Wine, e alguns programas que ainda dependem de código 32-bit.
  

**Como editar no nano:**

- Use as setas para navegar
- Delete o `#` na frente das linhas
- `Ctrl + O` para salvar
- `Enter` para confirmar
- `Ctrl + X` para sair

**Dica:** Se você errar algo, não se preocupe! Você pode sempre voltar e editar depois com `nano /etc/pacman.conf`.

### Fuso Horário

Configurar o fuso horário corretamente é essencial para que logs, timestamps de arquivos e aplicativos funcionem corretamente.

```bash
# Criar link simbólico do fuso horário
# Isso define São Paulo como nosso fuso horário
ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime

# Sincronizar o relógio do hardware com o do sistema
# Garante que o relógio da BIOS esteja correto
hwclock --systohc

# Habilitar sincronização automática via NTP
# NTP (Network Time Protocol) mantém o relógio sempre preciso via internet
timedatectl set-ntp true

# Verificar a data e hora atuais
date
```

**Entendendo cada comando:**

- **ln -sf:** Cria um link simbólico (atalho). O arquivo `/etc/localtime` apontará para as configurações de São Paulo.
  
- **hwclock --systohc:** Seu computador tem dois relógios:
  
  - **Relógio do sistema:** Usado pelo Linux enquanto ligado
  - **Relógio do hardware (RTC):** Na BIOS, funciona mesmo com PC desligado
  
  Este comando sincroniza o hardware com o sistema.
  
- **timedatectl set-ntp true:** Habilita a sincronização automática com servidores de tempo na internet. Assim seu relógio sempre estará correto, mesmo depois de meses desligado.
  

**Por que isso importa?**

Imagine compilar um programa e o sistema achar que estamos em 2020. Os timestamps ficariam errados e causar problemas em logs, backups, certificados SSL, e até em sistemas de arquivos.

**Outros fusos horários comuns no Brasil:**

- America/Manaus (Amazonas)
- America/Fortaleza (Ceará)
- America/Recife (Pernambuco)
- America/Bahia (Bahia)

Para ver todos disponíveis: `ls /usr/share/zoneinfo/America/`

### Localização

```bash
nano /etc/locale.gen
```

**Descomentar:** `pt_BR.UTF-8 UTF-8`

```bash
locale-gen
nano /etc/locale.conf
```

**Adicionar:** `LANG=pt_BR.UTF-8`

```bash
nano /etc/vconsole.conf
```

**Adicionar:** `KEYMAP=br-abnt2`

### Hostname

O hostname é o "nome" do seu computador na rede. É como ele será identificado em redes locais e em alguns comandos do sistema.

```bash
# Definir o hostname do sistema
# Aqui estamos usando "arch", mas você pode escolher qualquer nome
echo "arch" >> /etc/hostname
```

**Boas práticas para hostname:**

- Use apenas letras minúsculas, números e hífens
- Evite espaços e caracteres especiais
- Seja descritivo: "laptop-trabalho", "desktop-casa", "servidor-media"

Agora precisamos configurar o arquivo `hosts`, que mapeia nomes para endereços IP:

```bash
# Editar o arquivo hosts
nano /etc/hosts
```

**Adicione estas linhas:**

```
127.0.0.1     localhost
::1           localhost
127.0.1.1     arch.localdomain     arch
```

**Entendendo cada linha:**

1. **127.0.0.1 localhost:** O endereço de loopback IPv4. Sempre aponta para o próprio computador. Usado por programas que precisam se comunicar localmente.
  
2. **::1 localhost:** Mesma coisa, mas para IPv6 (próxima geração do protocolo de internet).
  
3. **127.0.1.1 arch.localdomain arch:** Mapeia seu hostname para um IP local. Isso permite que programas encontrem seu computador pelo nome, não apenas por IP.
  

**Por que 127.0.1.1 e não 127.0.0.1?**

Por convenção, 127.0.0.1 é reservado para "localhost". Usar 127.0.1.1 para o hostname evita conflitos e é a prática recomendada em sistemas desktop.

**O que é .localdomain?**

É um domínio falso usado para resolver o nome completo do host (FQDN - Fully Qualified Domain Name). Útil em ambientes corporativos, mas em desktops domésticos é apenas uma formalidade.

**Exemplo prático:** Se seu hostname é "arch", você pode fazer `ping arch` e funcionará, porque o sistema vai consultar o arquivo /etc/hosts e encontrar o IP 127.0.1.1.

**Importante:** Se você mudou o hostname de "arch" para outro nome, ajuste também no arquivo /etc/hosts!

---

## 👤 Usuários e Senhas

Configurar usuários e permissões é fundamental para a segurança do sistema. Nunca use o root para tarefas cotidianas!

### Configurar Senha do Root

O root é o super-usuário, com poder absoluto sobre o sistema. Use-o apenas para manutenção e configurações críticas.

```bash
# Definir senha para o usuário root
# Escolha uma senha FORTE e guarde em local seguro
passwd
```

**Por que o root precisa de senha?**

Mesmo que você planeje usar apenas seu usuário normal, situações de emergência (sistema não inicia, ambiente gráfico quebrado) exigem login como root. Sem senha, você fica travado!

**Dicas para senha forte:**

- Mínimo 12 caracteres
- Misture maiúsculas, minúsculas, números e símbolos
- Não use palavras do dicionário
- Exemplo: `$enh4Sup3rS3gur4!2025`

### Configurar Sudo

O `sudo` permite que usuários comuns executem comandos específicos com privilégios de root, sem precisar da senha do root.

```bash
# Editar o arquivo de configuração do sudo
nano /etc/sudoers
```

**⚠️ IMPORTANTE:** Sempre use `visudo` ou tenha muito cuidado ao editar este arquivo. Um erro pode travar o sistema!

**Procure e descomente a linha:**

```bash
%wheel ALL=(ALL:ALL) ALL
```

**Decifrando a sintaxe:**

- `%wheel`: Membros do grupo "wheel" (convenção Unix para administradores)
- `ALL`: Em todos os hosts (irrelevante em desktop, mas útil em servidores)
- `(ALL:ALL)`: Pode executar como qualquer usuário e grupo
- `ALL`: Pode executar qualquer comando

**Em português:** "Qualquer pessoa no grupo wheel pode executar qualquer comando como root usando sudo"

### Criar Seu Usuário

Agora vamos criar seu usuário pessoal, que você usará no dia a dia.

```bash
# Criar usuário e adicionar aos grupos necessários
# -m: Cria a pasta home automaticamente
# -G wheel: Adiciona ao grupo wheel (permite usar sudo)
useradd -mG wheel karllos

# Definir senha para o usuário
passwd karllos
```

**Substituindo "karllos":** Use seu nome preferido! Convenções:

- Tudo em minúsculas
- Sem espaços (use hífen ou underscore se necessário)
- Evite acentos

**Por que grupo wheel?**

O grupo wheel é uma convenção BSD que foi adotada pelo Linux. Historicamente, apenas membros deste grupo podiam usar o `su` (switch user). No Arch, membros do wheel podem usar sudo.

**Estrutura final de segurança:**

```
root (senha forte, uso emergencial)
  └─ seu-usuario (senha normal, uso diário)
      └─ sudo (poder temporário de root)
```

**Teste após instalação:** Tente `sudo pacman -Syu`. Se pedir senha do seu usuário e funcionar, o sudo está configurado corretamente!

---

## 🚀 Bootloader e Drivers

O bootloader é o primeiro programa que executa quando você liga o computador. Ele carrega o sistema operacional. Sem ele, o sistema não inicia!

### Pacotes Essenciais

Vamos instalar os componentes necessários para inicialização e conectividade básica.

```bash
pacman -S dosfstools mtools os-prober networkmanager iwd grub efibootmgr amd-ucode
```

**Detalhando cada pacote:**

- **dosfstools:** Utilitários para trabalhar com sistemas de arquivos FAT (necessário para a partição EFI).
  
- **mtools:** Ferramentas para manipular arquivos FAT sem montá-los (útil para diagnóstico).
  
- **os-prober:** Detecta automaticamente outros sistemas operacionais instalados (Windows, outras distros) e os adiciona ao menu do GRUB. Essencial para dual-boot!
  
- **networkmanager:** Gerenciador de rede moderno e user-friendly. Funciona tanto em terminal quanto com interfaces gráficas.
  
- **iwd:** (iNet Wireless Daemon) Alternativa moderna ao wpa_supplicant. Mais rápido, mais estável, melhor gerenciamento de WiFi.
  
- **grub:** (GRand Unified Bootloader) O bootloader mais popular do Linux. Fornece um menu bonito para escolher qual sistema iniciar.
  
- **efibootmgr:** Ferramenta para gerenciar entradas de boot UEFI. Permite adicionar/remover/reordenar opções de boot.
  
- **amd-ucode:** Microcódigo para processadores AMD. Atualiza o firmware do processador para corrigir bugs e melhorar estabilidade.
  
  - **Se você tem Intel:** Substitua por `intel-ucode`
  - **Como saber qual tenho?** Execute: `lscpu | grep Vendor`

**Por que microcode importa?**

Processadores têm bugs. Fabricantes lançam correções via microcode. Sem ele, você pode ter:

- Instabilidade do sistema
- Vulnerabilidades de segurança (Spectre, Meltdown)
- Performance reduzida

### Instalar GRUB

Agora vamos instalar o GRUB na partição EFI para que o computador possa inicializar o Arch Linux.

```bash
# Instalar o GRUB na partição EFI
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB --recheck
```

**Decifrando os parâmetros:**

- **--target=x86_64-efi:** Indica que estamos instalando para sistemas UEFI 64-bit (padrão em computadores modernos).
  
- **--efi-directory=/boot/efi:** Localização da partição EFI que montamos anteriormente.
  
- **--bootloader-id=GRUB:** Nome que aparecerá no menu de boot do UEFI. Você pode mudá-lo para "ArchLinux" ou qualquer outro nome.
  
- **--recheck:** Força o GRUB a verificar novamente os dispositivos. Útil se você já tentou instalar antes.
  

**O que esse comando faz?**

1. Cria os arquivos do GRUB em `/boot/efi/EFI/GRUB/`
2. Registra o GRUB na NVRAM do UEFI
3. Define prioridade de boot no firmware

**Mensagens de sucesso esperadas:**

```
Installing for x86_64-efi platform.
Installation finished. No error reported.
```

Agora vamos configurar o GRUB para detectar todos os sistemas operacionais:

```bash
# Editar configurações do GRUB
nano /etc/default/grub
```

**Procure e descomente (remova o #):**

```bash
GRUB_DISABLE_OS_PROBER=false
```

**Por que isso é necessário?**

Por padrão (por razões de segurança), o GRUB não busca automaticamente outros sistemas. Desabilitar essa proteção permite que o `os-prober` encontre Windows, Ubuntu, etc., e os adicione ao menu de boot.

**Se você NÃO tem dual-boot:** Pode deixar comentado. Menos código executando = boot mais rápido.

Agora geramos o arquivo de configuração final:

```bash
# Gerar o arquivo de configuração do GRUB
# Este comando lê /etc/default/grub e cria /boot/grub/grub.cfg
grub-mkconfig -o /boot/grub/grub.cfg
```

**O que acontece aqui:**

1. O GRUB procura kernels disponíveis (linux, linux-lts)
2. Lê o microcode (amd-ucode ou intel-ucode)
3. Executa os-prober para buscar outros sistemas
4. Gera um arquivo de configuração complexo mas otimizado
5. Salva tudo em `/boot/grub/grub.cfg`

**Verificações:**

- Deve aparecer "Found linux image: /boot/vmlinuz-linux"
- Se tiver Windows, deve aparecer "Found Windows Boot Manager"

**⚠️ Erro comum:** Se disser "grub-probe: error", verifique se `/boot/efi` está montado corretamente!

### Habilitar Serviços de Rede

Serviços que precisam iniciar automaticamente devem ser "habilitados" usando systemd. Vamos configurar a rede para funcionar no próximo boot.

```bash
# Habilitar NetworkManager e iwd para iniciarem automaticamente
systemctl enable NetworkManager iwd
```

**O que é systemd?**

Systemd é o gerenciador de sistema e serviços do Linux moderno. Ele:

- Inicia serviços em paralelo (boot mais rápido)
- Gerencia dependências entre serviços
- Monitora e reinicia serviços que falharem
- Controla logs do sistema

**Entendendo enable vs start:**

- `systemctl enable`: Configura para iniciar no boot (mas NÃO inicia agora)
- `systemctl start`: Inicia agora (mas NÃO salva para próximo boot)
- `systemctl enable --now`: Faz ambos simultaneamente

**Por que não usamos --now aqui?**

Estamos em chroot! O sistema não está realmente "rodando", então não faz sentido iniciar serviços agora. Eles iniciarão no primeiro boot real.

Agora vamos configurar o NetworkManager para usar iwd em vez de wpa_supplicant:

```bash
# Criar/editar configuração do NetworkManager
nano /etc/NetworkManager/NetworkManager.conf
```

**Adicione esta seção:**

```ini
[device]
wifi.backend=iwd
```

**Por que iwd em vez de wpa_supplicant?**

| Característica | wpa_supplicant | iwd |
| --- | --- | --- |
| Idade | Antigo (~2003) | Moderno (~2015) |
| Código | C complexo | C moderno, limpo |
| Conexão | 5-10 segundos | 1-3 segundos |
| CPU | Mais uso | Mais eficiente |
| Estabilidade | Muito estável | Muito estável |

**Configuração explicada:**

- `[device]`: Seção de configuração de dispositivos
- `wifi.backend=iwd`: Define iwd como backend de WiFi

**Alternativa:** Se você tiver problemas com iwd (raro), pode:

1. Remover esta configuração
2. Desabilitar iwd: `systemctl disable iwd`
3. NetworkManager voltará a usar wpa_supplicant automaticamente

### Finalizar Instalação Base

Parabéns! A instalação base está completa. Hora de sair do chroot e reiniciar no novo sistema.

```bash
# Sair do ambiente chroot
# Isso nos leva de volta ao instalador
exit

# Reiniciar o computador
# Remova a mídia de instalação (pendrive/DVD) quando solicitado
reboot
```

**O que acontece no reboot:**

1. O computador desliga
2. UEFI lê a NVRAM e encontra a entrada do GRUB
3. GRUB é carregado e mostra o menu
4. Você escolhe "Arch Linux" (ou espera 5 segundos para boot automático)
5. GRUB carrega o kernel Linux e initramfs
6. Kernel inicializa hardware e monta partições (usando fstab)
7. Systemd inicia serviços habilitados (NetworkManager, iwd, etc.)
8. Você vê a tela de login!

**Se algo der errado:**

- **Tela preta após GRUB:** Problema com drivers de vídeo. Adicione `nomodeset` nos parâmetros do kernel no menu do GRUB (pressione 'e' para editar).
  
- **GRUB não aparece:** Problema na instalação do bootloader. Reinicie pela ISO, monte as partições novamente, entre no chroot e reinstale o GRUB.
  
- **Kernel panic:** fstab incorreto ou partição raiz não encontrada. Verifique UUIDs no fstab.
  
- **Sem rede:** NetworkManager não habilitado. Entre como root e execute `systemctl enable --now NetworkManager`.
  

**Checklist pré-reboot:**

- ✅ GRUB instalado e configurado
- ✅ fstab gerado corretamente
- ✅ Senha do root definida
- ✅ Usuário criado e no grupo wheel
- ✅ NetworkManager habilitado
- ✅ Timezone, locale e hostname configurados

**Próximos passos após o reboot:**

1. Login com seu usuário
2. Conectar ao WiFi
3. Atualizar o sistema
4. Instalar drivers gráficos
5. Instalar ambiente desktop (GNOME, KDE, Hyprland, etc.)

---

## 🌟 Pós-Instalação

Bem-vindo ao seu novo sistema Arch Linux! Agora vamos configurar o ambiente para uso diário.

### Primeiro Boot

Você deve ver a tela de login. Digite seu nome de usuário e senha.

```bash
# Configurar sincronização automática de horário
# Garante que o relógio esteja sempre correto
timedatectl set-ntp true
```

**Por que fazer isso de novo?**

No chroot, configuramos o timezone. Mas o NTP (sincronização via internet) precisa ser ativado no sistema real, não no chroot.

### Conectar ao WiFi via Terminal

```bash
# Listar redes WiFi disponíveis
nmcli device wifi list

# Conectar a uma rede
# --ask solicita a senha de forma segura
nmcli device wifi connect "nome-da-rede" --ask
```

**NetworkManager CLI (nmcli):**

O `nmcli` é a interface de linha de comando do NetworkManager. Muito mais simples que iwctl!

**Comandos úteis do nmcli:**

```bash
# Ver status da conexão
nmcli general status

# Ver conexões salvas
nmcli connection show

# Desconectar
nmcli device disconnect wlan0

# Reconectar a uma rede salva
nmcli connection up "nome-da-rede"

# Esquecer uma rede salva
nmcli connection delete "nome-da-rede"
```

**Dica:** O NetworkManager salva senhas automaticamente. Na próxima vez, conectará automaticamente!

### Instalar Fastfetch

Vamos instalar uma ferramenta para exibir informações do sistema de forma bonita.

```bash
# Instalar fastfetch (versão moderna e rápida do neofetch)
sudo pacman -S fastfetch

# Executar para ver informações do sistema
fastfetch
```

**O que o fastfetch mostra:**

- Distribuição e versão do kernel
- Uptime (tempo ligado)
- Packages instalados
- Shell em uso
- Ambiente desktop
- CPU, GPU, RAM
- Cores do terminal
- ASCII art do logo do Arch

**Neofetch vs Fastfetch:**

Neofetch está abandonado (último update em 2021). Fastfetch é o sucessor:

- 10x mais rápido
- Escrito em C (neofetch era bash)
- Mais opções de customização
- Mantido ativamente

### Drivers AMD

Se você tem placa de vídeo AMD (integrada ou dedicada), vamos instalar os drivers necessários para aceleração gráfica completa.

```bash
sudo pacman -S xf86-video-amdgpu mesa lib32-mesa vulkan-radeon lib32-vulkan-radeon
```

**Detalhando cada pacote:**

- **xf86-video-amdgpu:** Driver DDX (Display Device X) para GPUs AMD modernas (GCN 3.0+). Fornece aceleração 2D para o X11.
  
- **mesa:** Implementação open-source do OpenGL, OpenGL ES, Vulkan, e outros APIs gráficos. É o "coração" dos gráficos Linux.
  
- **lib32-mesa:** Versão 32-bit da mesa. Necessária para rodar jogos e aplicativos antigos que ainda são 32-bit (Steam, Wine, muitos jogos).
  
- **vulkan-radeon:** Driver Vulkan para AMD (RADV). Vulkan é uma API gráfica moderna e de alta performance, usada por jogos modernos.
  
- **lib32-vulkan-radeon:** Versão 32-bit do driver Vulkan. Novamente, para compatibilidade com jogos antigos.
  

**Para usuários Intel:**

```bash
sudo pacman -S xf86-video-intel mesa lib32-mesa vulkan-intel lib32-vulkan-intel
```

**Para usuários NVIDIA:**

```bash
# Drivers proprietários (recomendado)
sudo pacman -S nvidia nvidia-utils lib32-nvidia-utils

# OU drivers open-source (experimental, menos performance)
sudo pacman -S xf86-video-nouveau mesa lib32-mesa
```

**Por que lib32 é importante?**

Muitos jogos, especialmente no Steam, ainda rodam em código 32-bit. Sem as bibliotecas lib32, você verá erros como "missing shared libraries" ao tentar jogar.

**Testando os drivers:**

Após instalar o ambiente gráfico, você pode testar com:

```bash
# Ver qual driver está sendo usado
glxinfo | grep "OpenGL renderer"

# Ver informações do Vulkan
vulkaninfo | grep "deviceName"

# Benchmark simples
glxgears
```

**Nota importante:** Estes são drivers open-source. Para placas AMD, a performance é geralmente excelente (às vezes melhor que no Windows!). Para NVIDIA, os drivers proprietários são recomendados para melhor performance.

### Utilitários Essenciais

Vamos instalar ferramentas fundamentais para gerenciar o sistema e facilitar o uso diário.

```bash
sudo pacman -S git btop htop wget unzip zip bash-completion openssh reflector
```

**Explicando cada ferramenta:**

- **git:** Sistema de controle de versão. Essencial para baixar código do GitHub, clonar repositórios AUR, e desenvolvimento.
  
- **btop:** Monitor de sistema moderno e bonito. Mostra CPU, RAM, disco, rede, e processos em tempo real com gráficos coloridos. Substituiu o antigo htop para muitos usuários.
  
- **htop:** Monitor de processos interativo. Mais user-friendly que o `top` padrão. Use para matar processos travados, ver consumo de recursos, etc.
  
- **wget:** Baixa arquivos da internet via linha de comando. Suporta resumir downloads interrompidos.
  
- **unzip:** Extrai arquivos .zip. Surpreendentemente, não vem por padrão!
  
- **zip:** Cria arquivos .zip. Útil para comprimir arquivos para compartilhar.
  
- **bash-completion:** Autocomplete inteligente no terminal. Pressione TAB duas vezes para ver sugestões de comandos, arquivos, opções, etc. MUITO útil!
  
- **openssh:** Cliente e servidor SSH. Permite conectar remotamente a outros computadores de forma segura, transferir arquivos com scp/sftp, usar túneis, etc.
  
- **reflector:** Já usamos na instalação! Útil ter instalado para atualizar os mirrors periodicamente.
  

**Como usar cada ferramenta:**

```bash
# btop - Monitor bonito (pressione q para sair)
btop

# htop - Monitor de processos (F9 para matar processo, F10 para sair)
htop

# wget - Baixar arquivo
wget https://exemplo.com/arquivo.zip

# Git - Clonar repositório
git clone https://github.com/usuario/projeto.git

# Bash completion - Testar
sudo pac[TAB][TAB]  # Mostra: pacman, pacman-key, etc.

# SSH - Conectar a servidor remoto
ssh usuario@ip-do-servidor

# Reflector - Atualizar mirrors
sudo reflector --latest 10 --sort rate --save /etc/pacman.d/mirrorlist
```

**Dica de produtividade:** Configure aliases no `~/.bashrc` para comandos comuns:

```bash
alias update='sudo pacman -Syu'
alias install='sudo pacman -S'
alias remove='sudo pacman -Rns'
alias search='pacman -Ss'
```

### Instalar YAY (AUR Helper)

```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd ..
rm -rf yay
```

### Bluetooth

Se seu computador tem Bluetooth, vamos instalar e configurar o suporte.

```bash
# Instalar stack Bluetooth
sudo pacman -S bluez bluez-utils

# Habilitar o serviço Bluetooth para iniciar automaticamente
sudo systemctl enable bluetooth
```

**Entendendo os pacotes:**

- **bluez:** Stack Bluetooth oficial do Linux. Implementa protocolos Bluetooth, drivers, e gerenciamento de dispositivos.
  
- **bluez-utils:** Utilitários de linha de comando para gerenciar Bluetooth (`bluetoothctl`, `hciconfig`, `hcitool`, etc.).
  

**Usando Bluetooth no terminal:**

```bash
# Iniciar a ferramenta interativa
bluetoothctl

# Dentro do bluetoothctl:
power on              # Ligar Bluetooth
agent on              # Ativar agente de pareamento
default-agent         # Definir como agente padrão
scan on               # Escanear dispositivos
pair MAC:DO:DEVICE    # Parear com dispositivo
connect MAC:DO:DEVICE # Conectar ao dispositivo
trust MAC:DO:DEVICE   # Confiar no dispositivo (reconectar automaticamente)
exit                  # Sair
```

**No ambiente gráfico:**

GNOME, KDE, e outros ambientes desktop têm gerenciadores gráficos de Bluetooth integrados. Você verá um ícone na barra de sistema para gerenciar dispositivos facilmente.

**Troubleshooting comum:**

**Problema:** Bluetooth não liga

```bash
# Verificar se está bloqueado
rfkill list

# Desbloquear se necessário
sudo rfkill unblock bluetooth

# Verificar status do serviço
systemctl status bluetooth

# Iniciar manualmente se não estiver rodando
sudo systemctl start bluetooth
```

**Problema:** Dispositivo não pareia

```bash
# Remover dispositivo e tentar novamente
bluetoothctl
remove MAC:DO:DEVICE
scan on
pair MAC:DO:DEVICE
```

**Problema:** Áudio Bluetooth com qualidade ruim

Instale codecs de áudio adicionais:

```bash
sudo pacman -S pipewire-pulse libldac
```

**Dispositivos comuns:**

- Fones de ouvido
- Mouses e teclados
- Controles de videogame
- Smartphones (transferência de arquivos)
- Caixas de som

**Dica:** Se você não usa Bluetooth, pode desabilitar completamente para economizar bateria:

```bash
sudo systemctl disable bluetooth
sudo systemctl stop bluetooth
```

---

## 🖥️ Ambiente Gráfico - GNOME

Até agora trabalhamos apenas no terminal. Hora de instalar uma interface gráfica completa! GNOME é moderno, bonito, e user-friendly.

### Atualizar Sistema

Sempre atualize antes de instalar grandes conjuntos de pacotes.

```bash
# Atualizar todos os pacotes instalados
sudo pacman -Syu
```

**Por que atualizar primeiro?**

Garante que:

- Você tem as versões mais recentes
- Evita conflitos de dependências
- Corrige bugs conhecidos
- Aplica patches de segurança

**Frequência recomendada:** Atualize semanalmente. Arch é rolling release - atualizações constantes são normais e esperadas.

### Instalar Xorg

Xorg (ou X11) é o sistema de janelas que permite interfaces gráficas funcionarem no Linux.

```bash
# Instalar servidor X e utilitários
sudo pacman -S xorg xorg-xinit
```

**O que é Xorg?**

Xorg é o servidor de display que:

- Gerencia sua tela, teclado e mouse
- Renderiza janelas e gráficos
- Coordena entre hardware e aplicações gráficas

**xorg-xinit:** Contém `startx`, comando para iniciar interface gráfica manualmente (útil para debugging).

**Alternativa moderna:** Wayland

Wayland é o sucessor do Xorg, mais moderno e seguro. GNOME já suporta bem Wayland e usará por padrão. Mas instalamos Xorg como fallback para compatibilidade com programas antigos.

### Instalar GNOME

GNOME é um ambiente desktop completo, elegante e fácil de usar.

```bash
# Instalar componentes principais do GNOME
sudo pacman -S gdm gnome-shell gnome-desktop gnome-backgrounds gnome-tweaks \
               gnome-session gnome-keyring gnome-control-center \
               gnome-settings-daemon gnome-console gnome-terminal \
               xdg-user-dirs-gtk adwaita-icon-theme nautilus loupe
```

**Decifrando cada componente:**

- **gdm (GNOME Display Manager):** Tela de login gráfica. Mostra os usuários e permite login com senha.
  
- **gnome-shell:** O "shell" do GNOME. Interface principal, barra superior, overview (apertar tecla Super), etc.
  
- **gnome-desktop:** Bibliotecas base do GNOME, usadas por todos os componentes.
  
- **gnome-backgrounds:** Papéis de parede padrão do GNOME. Bonitos e em alta resolução!
  
- **gnome-tweaks:** Ferramenta para customização avançada. Mude temas, fontes, comportamentos, extensões, etc.
  
- **gnome-session:** Gerencia a sessão do usuário (login, logout, lock screen, etc.).
  
- **gnome-keyring:** Armazena senhas e chaves de forma segura e criptografada. Usado por navegadores, WiFi, SSH, etc.
  
- **gnome-control-center:** Painel de configurações do sistema (Settings). Configure tudo aqui!
  
- **gnome-settings-daemon:** Roda em background gerenciando configurações (teclado, mouse, temas, etc.).
  
- **gnome-console:** Terminal moderno e bonito do GNOME (substituiu o gnome-terminal como padrão).
  
- **gnome-terminal:** Terminal clássico do GNOME. Mais recursos que o console, preferido por power users.
  
- **xdg-user-dirs-gtk:** Cria pastas padrão (Documentos, Downloads, Imagens, etc.) no primeiro login.
  
- **adwaita-icon-theme:** Ícones padrão do GNOME. Sem isso, muitos ícones não aparecem!
  
- **nautilus:** Gerenciador de arquivos (equivalente ao Explorer do Windows). Essencial!
  
- **loupe:** Visualizador de imagens moderno e rápido do GNOME.
  

**Instalação mínima vs completa:**

Esta é uma instalação minimalista! O meta-pacote `gnome` instala 100+ aplicações (Calculadora, Calendário, Weather, etc.). Se preferir tudo:

```bash
sudo pacman -S gnome gnome-extra
```

**Vantagens da instalação mínima:**

- Sistema mais leve
- Menos bloatware
- Você escolhe cada aplicação que quer
- Instale depois conforme necessidade

### Codecs Multimídia

```bash
sudo pacman -S gstreamer gst-libav gst-plugins-base gst-plugins-good \
               gst-plugins-bad gst-plugins-ugly ffmpeg
```

### Fontes

```bash
sudo pacman -S ttf-roboto ttf-droid ttf-opensans \
               ttf-jetbrains-mono ttf-jetbrains-mono-nerd
```

### Habilitar GDM e Audio

```bash
sudo systemctl enable gdm
systemctl --user enable pipewire pipewire-pulse
reboot
```

---

## 🎨 Ambiente Gráfico - Hyprland

*Seção em desenvolvimento...*

---

## 📝 Notas Importantes

- **Backup:** Sempre faça backup dos seus dados antes de instalar
- **Partições:** Ajuste os tamanhos conforme suas necessidades
- **Drivers:** Substitua `amd-ucode` por `intel-ucode` se usar Intel
- **Desktop:** Escolha entre GNOME, Hyprland ou outro ambiente de sua preferência

## 🔗 Recursos Úteis

- [Arch Wiki](https://wiki.archlinux.org/)
- [Arch Linux Brasil](https://archlinux-br.org/)
- [r/archlinux](https://www.reddit.com/r/archlinux/)

---

**Instalação concluída! Aproveite seu Arch Linux! 🎉**
