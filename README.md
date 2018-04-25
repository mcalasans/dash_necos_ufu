# dash_necos_ufu

Host: Ubuntu 16.04 e Virtualbox 5.2.10

I- Criar rede interna no Virtualbox
--------------------------------------------------
**a) Criar a rede interna**

1. File-> Host Network Manager
2. Clicar em Create
3. Marcar DHCP Server
4. Escolher a aba DHCP Server
   - Server Address: 192.168.56.100
   - Server Mask: 255.255.255.0
   - Lower Address Bound: 192.168.56.101
   - Upper Address Bound: 192.168.56.254
 

II - Criar VM do servidor DASH
--------------------------------------------------
**a) Baixar imagem do Ubuntu Server 16.04. Exemplo:**

```
wget http://ubuntu.c3sl.ufpr.br/releases/16.04.4/ubuntu-16.04.4-server-amd64.iso
```

**b) Criar a máquina no Virtualbox**

1. Clicar em New
2. Clicar em Expert Mode
3. Configurar
   - Name: DASHServer
   - Type: Linux
   - Version: Ubuntu 64 bits
   - Memory: 512 MB
   - Hard disk: Create a virtual hard disk now
4. Clicar em Create
5. Configurar:
   - File location: DASHServer
   - File size: 30 GB
   - File type: VDI
   - Storage on physical HD: Dynamically allocated
6. Clicar em Create 
7. Clicar em Settings (com a VM selecionada)
8. Clicar em Network -> Adapter 2
9. Configurar:
   - Enable Network Adapter (marcar)
   - Attached to: Internal Network
10. Clicar em Network -> Adapter 3
11. Configurar:
    - Enable Network Adapter (marcar)
    - Attached to: Host-only Adapter
    - Name: vboxnet0 (rede criada em I)
12. Clicar em OK


**c) Configurar a ISO de instalação do Ubuntu**

1. Clicar em Settings (com a VM selecionada)
2. Clicar em Storage
3. Selecionar o ícone do CD (empty) em Storage Devices
4. Clicar no ícone do CD em Optical Drive
5. Clicar em "Choose Virtual..."
6. Selecionar a ISO e clicar em Open
7. Marcar Live CD/DVD
8. Clicar em OK

**d) Instalar o Ubuntu**

1.  Selecionar a VM DASHServer e clicar em Start
2.  Selecionar English
3.  Selecionar Install Ubuntu Server
4.  Language: English - English
5.  Location: Other -> South America -> Brazil
6.  Locales: United States - en_US.UTF-8
7.  Autodetect Keyboard: No
8.  Keyboard origin: Portuguese (Brazil)
9.  Keyboard layout: Portuguese (Brazil)
10. Primary network: enp0s3 <adaptador1>
11. Hostname: dashserver
12. User: serveruser
13. Username: serveruser
14. Password: 
15. Homedir Encryption: No
16. Clock: America/Sao_Paulo (Yes)
17. Partition Method: Manual
18. Partition Disks: (sda)
    - Create new partition table: Yes
    - Selecionar o espaço livre
    - Create a new partition: "2GB" "primary" "beginning" "use as:swap" Done
    - Selecionar o espaço livre
    - Create a new partition: "30.2GB" "primary" "use as:Ext4..." "mount point:\" Done
    - Finish parttioning(...)
    - Verificar e clicar em Yes
19. Proxy: <nao configurar>
20. Tasksel: No automatic updates
21. Software: standard system utilities; OpenSSH server; 
22. GRUB Install: Yes
23. Finalizar a instalação clicando em Continue
		
Remover a ISO após instalação.

III - Configurar o servidor DASH
-------------------------------------------------
Obs.: Recomendável salvar uma cópia de segurança antes, clonando a máquina (desligada).

**a) Atualizar os pacotes e instalar o unzip**

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
sudo apt-get install unzip
```

**b) Alterar o /etc/sudoers para não solicitar senha**

```
sudo visudo
```

Incluir a seguinte linha no final do arquivo:

```
serveruser ALL=(ALL:ALL) NOPASSWD: ALL
```

Salvar como /etc/sudoers.

**c) Alterar o /etc/inputrc para pesquisar o histórico com PgUp e PgDn**

```
sudo vim /etc/inputrc
```

Descomentar as linhas (41 e 42):

```
"\e[5~": history-search-backward
"\e[6~": history-search-forward
```

Salvar o arquivo (:x) e logar novamente.

**d) Configurar as interfaces no /etc/network/interfaces.** 

Obs.: Para saber o nome das interfaces use antes o comando ```ip link show```.

```
sudo vim /etc/network/interfaces
```

O arquivo deve incluir a configuração das três interfaces:

```
# NAT
# The primary network interface
auto enp0s3
iface enp0s3 inet dhcp

# Internal Network
auto enp0s8
iface enp0s8 inet static
address 172.16.0.10
netmask 255.255.255.0
network 172.16.0.0
broadcast 172.16.0.255

# Host-only
auto enp0s9
iface enp0s9 inet static
address 192.168.56.10
netmask 255.255.255.0
network 192.168.56.0
broadcast 192.168.56.255
```

Reiniciar o servidor:

```
sudo shutdown -r now
```

O restante do documento pode ser executado via ssh da máquina hospederia (192.168.56.1) para o servidor:

```
ssh serveruser@192.168.56.10
```

**e) Instalar o apache e configurar o MIME**

```
sudo apt-get install apache2
```

Editar o arquivo mime.conf

```
sudo vim /etc/apache2/mods-available/mime.conf
```

Incluir as linhas no local apropriado:

```
        AddType application/dash+xml .mpd
        AddType application/x-mpegURL .m3u8
        
        AddType audio/aac .aac
        AddType audio/mp4 .mp4 .m4a
        AddType audio/mpeg .mp1 .mp2 .mp3 .mpg .mpeg
        AddType audio/ogg .oga .ogg
        AddType audio/wav .wav
        AddType audio/webm .webm

        AddType video/mp4 .mp4 .m4v
        AddType video/ogg .ogv
        AddType video/webm .webm
        AddType video/MP2T .ts
```

Reiniciar configuração do apache:

```
sudo systemctl reload apache2
```

f) Instalar o ffmpeg

  Se desejar remover pacotes existentes (instalações prévias):
  
  ```
  rm -rf ~/ffmpeg_build ~/ffmpeg_sources ~/bin/{ffmpeg,ffprobe,ffserver,vsyasm,x264,yasm,ytasm}
  sed -i '/ffmpeg_build/d' ~/.manpath
  hash -r
  sudo apt-get autoremove autoconf automake build-essential git libass-dev libgpac-dev \
  libmp3lame-dev libopus-dev libsdl1.2-dev libtheora-dev libtool libva-dev libvdpau-dev \
  libvorbis-dev libvpx-dev libx11-dev libxext-dev libxfixes-dev texi2html zlib1g-dev yasm mercurial
  ```

1. Baixar as dependencias:

  ```
  udo apt-get update
  sudo apt-get -y install autoconf automake build-essential checkinstall git libfaac-dev \
  libgpac-dev libjack-jackd2-dev libopencore-amrnb-dev libopencore-amrwb-dev \
  librtmp-dev libsdl1.2-dev libtheora-dev libva-dev libvdpau-dev libvorbis-dev \
  libx11-dev libxfixes-dev pkg-config texi2html zlib1g-dev libass-dev cmake mercurial
  ```

2. Criar os diretórios:

  ```
  mkdir -p ~/ffmpeg_sources ~/bin 
  ```

3. Instalar as bibliotecas:

   - NASM (assembler): 
   
   ```
   cd ~/ffmpeg_sources && \
   wget http://www.nasm.us/pub/nasm/releasebuilds/2.13.02/nasm-2.13.02.tar.bz2 && \
   tar xjvf nasm-2.13.02.tar.bz2 && \
   cd nasm-2.13.02 && \
   PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" && \
   make && \
   make install
   ```
   
. YASM (assembler) - v1.3.0-2 
.libx264 (H.264 video encoder) - v148
.libvpx (VP8/VP9 video encoder and decoder) - v1.5.0-2
.libfdk-aac (AAC audio encoder) - v0.1.3
.libmp3lame (MP3 audio encoder) - v3.99.5
.libopus (Opus audio decoder and encoder) - v1.1.2

$ sudo apt-get install yasm libx264-dev libvpx-dev libfdk-aac-dev libmp3lame-dev libopus-dev

.libx265 (H.265/HEVC video encoder) - tem de ser compilada

$ cd ~/ffmpeg_sources && \
if cd x265 2> /dev/null; then hg pull && hg update; else hg clone https://bitbucket.org/multicoreware/x265; fi && \
cd x265/build/linux && \
PATH="$HOME/bin:$PATH" cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="$HOME/ffmpeg_build" -DENABLE_SHARED:bool=off ../../source && \
PATH="$HOME/bin:$PATH" make && \
make install

5. Instalar ffmpeg

$ cd ~/ffmpeg_sources && \
wget -O ffmpeg-snapshot.tar.bz2 http://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2 && \
tar xjvf ffmpeg-snapshot.tar.bz2 && \
cd ffmpeg && \
PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure \
  --prefix="$HOME/ffmpeg_build" \
  --pkg-config-flags="--static" \
  --extra-cflags="-I$HOME/ffmpeg_build/include" \
  --extra-ldflags="-L$HOME/ffmpeg_build/lib" \
  --extra-libs="-lpthread -lm" \
  --bindir="$HOME/bin" \
  --enable-gpl \
  --enable-libass \
  --enable-libfdk-aac \
  --enable-libfreetype \
  --enable-libmp3lame \
  --enable-libopus \
  --enable-libtheora \
  --enable-libvorbis \
  --enable-libvpx \
  --enable-libx264 \
  --enable-libx265 \
  --enable-nonfree && \
PATH="$HOME/bin:$PATH" make && \
make install && \
hash -r

