# dash_necos_ufu

Host: Ubuntu 16.04 e Virtualbox

I- Criar rede interna no Virtualbox
--------------------------------------------------
a) Criar a rede interna

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
a) Baixar imagem do Ubuntu Server 16.04. Exemplo:

```
$ wget http://ubuntu.c3sl.ufpr.br/releases/16.04.4/ubuntu-16.04.4-server-amd64.iso
```

b) Criar a máquina no Virtualbox

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

c) Configurar a ISO de instalação do Ubuntu

1. Clicar em Settings (com a VM selecionada)
2. Clicar em Storage
3. Selecionar o ícone do CD (empty) em Storage Devices
4. Clicar no ícone do CD em Optical Drive
5. Clicar em "Choose Virtual..."
6. Selecionar a ISO e clicar em Open
7. Clicar em OK


