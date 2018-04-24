# dash_necos_ufu

Host: Ubuntu 16.04 e Virtualbox 5.2.10

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


d) Instalar o Ubuntu

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
14. Password: u@Dash$1
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
		

III - Configurar o servidor DASH
-------------------------------------------------
Observação: Recomendável salvar uma cópia de segurança antes, clonando a máquina.
