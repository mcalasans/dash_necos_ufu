# dash_necos_ufu

Host: Ubuntu 16.04 e Virtualbox

I- Criar rede interna no Virtualbox
--------------------------------------------------
a) Criar a rede interna

1. File-> Host Network Manager
1. Clicar em Create
1. Marcar DHCP Server
1. Escolher a aba DHCP Server
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
