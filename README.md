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
**a) Baixar imagem do Ubuntu Server 16.04.**

Exemplo:

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

**e) Instalar o Apache e configurar o MIME**

Instalar o Apache:

```
sudo apt-get install apache2
```

Editar o arquivo mime.conf:

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

**f) Instalar o ffmpeg**

Se desejar remover pacotes existentes (instalações prévias):
  
```
rm -rf ~/ffmpeg_build ~/ffmpeg_sources ~/bin/{ffmpeg,ffprobe,ffserver,x265,nasm,ndisasm}
sed -i '/ffmpeg_build/d' ~/.manpath
hash -r
sudo apt-get autoremove autoconf automake build-essential checkinstall git libfaac-dev \
libgpac-dev libjack-jackd2-dev libopencore-amrnb-dev libopencore-amrwb-dev \
librtmp-dev libsdl1.2-dev libtheora-dev libva-dev libvdpau-dev libvorbis-dev \
libx11-dev libxfixes-dev pkg-config texi2html zlib1g-dev libass-dev cmake mercurial \
yasm libx264-dev libvpx-dev libfdk-aac-dev libmp3lame-dev libopus-dev
```

Baixar as dependencias:

```
sudo apt-get update
sudo apt-get -y install autoconf automake build-essential checkinstall git libfaac-dev \
libgpac-dev libjack-jackd2-dev libopencore-amrnb-dev libopencore-amrwb-dev \
librtmp-dev libsdl1.2-dev libtheora-dev libva-dev libvdpau-dev libvorbis-dev \
libx11-dev libxfixes-dev pkg-config texi2html zlib1g-dev libass-dev cmake mercurial
```

Criar os diretórios:

```
mkdir -p ~/ffmpeg_sources ~/bin 
```

Instalar as bibliotecas:

   + NASM (assembler): 
   
   ```
   cd ~/ffmpeg_sources && \
   wget http://www.nasm.us/pub/nasm/releasebuilds/2.13.02/nasm-2.13.02.tar.bz2 && \
   tar xjvf nasm-2.13.02.tar.bz2 && \
   cd nasm-2.13.02 && \
   PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" && \
   make && \
   make install
   ```
   
   - YASM (assembler) \- v1.3.0-2 
   - libx264 (H.264 video encoder) \- v148
   - libvpx (VP8/VP9 video encoder and decoder) \- v1.5.0-2
   - libfdk-aac (AAC audio encoder) - v0.1.3
   - libmp3lame (MP3 audio encoder) \- v3.99.5
   - libopus (Opus audio decoder and encoder) \- v1.1.2
   
   ``` 
   sudo apt-get install yasm libx264-dev libvpx-dev libfdk-aac-dev libmp3lame-dev libopus-dev
   ```
 
   - libx265 (H.265/HEVC video encoder) \- tem de ser compilada:
 
   ```
   cd ~/ffmpeg_sources && \
   if cd x265 2> /dev/null; then hg pull && hg update; else hg clone https://bitbucket.org/multicoreware/x265; fi && \
   cd x265/build/linux && \
   PATH="$HOME/bin:$PATH" cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="$HOME/ffmpeg_build" -DENABLE_SHARED:bool=off ../../source && \
   PATH="$HOME/bin:$PATH" make && \
   make install
   ```


Instalar o ffmpeg:

```
cd ~/ffmpeg_sources && \
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
```

Maiores informações sobre a instalação do ffmpeg no Ubuntu podem ser obtidas no [tutorial](http://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu) disponibilizado na página oficial.

**g) Instalar o MP4Box**

```
sudo apt install gpac
```


**h) Obter arquivo de teste**

O arquivo de teste utilizado está disponível [aqui](https://www.xiph.org/video/vid1.shtml). O arquivo é utilizado apenas para fins acadêmicos e todos os direitos são reservados aos autores do vídeo, conforme especificado na página.

```
mkdir ~/video && \
cd ~/video && \
wget http://downloads.xiph.org/video/A_Digital_Media_Primer_For_Geeks-720p.webm
```

Como o vídeo está em formato WebM, ele precisará ser convertido para MP4:

```
ffmpeg -fflags +genpts -i source.webm -r 30 source.mp4
```

Para ver informações sobre o vídeo:

```
MP4Box -info source.mp4
```
ou 
```
ffmpeg -i source.mp4
```

**i) Gerar arquivo de áudio:**

Extrair a faixa de audio e recodificá-la com dois canais (stereo) a 128, 64 e 32kps:

```
ffmpeg -i source.mp4 -c:a aac -ac 2 -b:a 128k -vn source-audio-128k.mp4 && \
ffmpeg -i source.mp4 -c:a aac -ac 2 -b:a 64k -vn source-audio-64k.mp4 && \
ffmpeg -i source.mp4 -c:a aac -ac 2 -b:a 32k -vn source-audio-32k.mp4
```

Onde 

-i   = arquivo de entrada  
-c:a = codec de áudio  
-ac  = número de canais  
-vn  = desconsidera vídeo   
-b:a = bitrate de áudio  

Para extrair a faixa de audio como é:

```
ffmpeg -i source.mp4 -c:a copy -vn source-audio.mp4
```

Maiores informações sobre o codec AAC podem sem obtidas [aqui](http://trac.ffmpeg.org/wiki/Encode/AAC).

**j) Gerar arquivos de vídeo:**


Codificação: H.264
Força a ter um quadro chave a cada 24 quadros. como o video é 24fps, tem um quadro chave por segundo.
O buffer deve ser menor que a taxa de solicitação (já que os segmentos tem duração de 1s)

ffmpeg -i source.mp4 -an -r 12 -c:v libx264 -x264opts 'keyint=12:min-keyint=12:no-scenecut' -b:v 300k -maxrate 300k -bufsize 150k -vf 'scale=256:144' source_256x144_12_300k.mp4 

ffmpeg -i source.mp4 -an -r 18 -c:v libx264 -x264opts 'keyint=18:min-keyint=18:no-scenecut' -b:v 700k -maxrate 700k -bufsize 350k -vf 'scale=320:240' source_320x240_18_700k.mp4

ffmpeg -i source.mp4 -an -r 24 -c:v libx264 -x264opts 'keyint=24:min-keyint=24:no-scenecut' -b:v 2100k -maxrate 2100k -bufsize 1050k -vf 'scale=640:480' source_640x480_24_2100k.mp4

ffmpeg -i source.mp4 -an -r 30 -c:v libx264 -x264opts 'keyint=30:min-keyint=30:no-scenecut' -b:v 3760k -maxrate 3760k -bufsize 1880k -vf 'scale=1280:720' source_1280x720_30_3760k.mp4


