## VPN NO MIKROTIK COM OPENVPN

Use o terminal do mikrotik para executar os comandos, ou pelo ssh.

### *1 - Certificado Servidor:*

/certificate add name=ca country=”br” state=”sao_paulo” locality=”cidade” organization=”organizacao” unit=”setor|departamento” common-name=”ca” key-size=4096 days-valid=3650 key-usage=crl-sign,key-cert-sign

/certificate sign ca ca-crl-host=127.0.0.1 name=”ca”

/certificate add name=server country=”br” state=”sao_paulo” locality=”cidade” organization=”organizacao” unit=”setor|departamento” common-name=”ip-publico-servidor” key-size=4096 days-valid=3650 key-usage=digital-signature,key-encipherment,tls-server trusted=yes

/certificate sign server ca=”ca” name=”server”

### *2 - Certificado cliente:*

/certificate add name=client country=”br” state=”sao_paulo” locality=”cidade” organization=”organizacao” unit=”setor|departamento” common-name=”client” key-size=4096 days-valid=3650 key-usage=tls-client

/certificate add name=”filial-01” copy-from=client common-name=”filial-01”

/certificate sign filial-01 ca=”ca” name=”filial-01” 

### *3 - Exportando certificados:*

/certificate export-certificate ca export-passphrase=””

/certificate export-certificate filial-01 export-passphrase=crie-senha-cliente

### *4 – Adicionar profile* 

/ppp profile add name=vpn local-address=10.10.10.1 remote-address=10.10.10.2 change-tcp-mss=yes use-compression=no use-encryption=required use-upnp=default

### *5 - Criar um novo PPP Secret*

/ppp secret add name=filial-01 service=ovpn password=#123!321# profile=vpn

name = nome do cliente  
password = senha do cliente

### *6 – Configure OVPN Server*

/interface ovpn-server server set enabled=yes port=1194 mode=ip netmask=24 max-mtu=1500 keepalive-timeout=60 default-profile=vpn certificate=server require-client-certificate=yes auth=sha1 cipher=aes256

port = porta da vpn (padrão 1194)

### *7 - Adicione rota para a rede local do mikrotik da filial*

/ip route add dst-address=172.16.2.0/24 gateway=10.10.10.1

### *8 - Adicione regras no firewall para input e nat*

/ip firewall filter add chain=input dst-port=1194 protocol=tcp

/ip firewall nat add chain=srcnat src-address=172.16.0.0/24 dst-address=172.16.2.0/24

### *9 - Faça o download dos seguintes certificados:*

cert_export_ca.crt  
cert_export_filial-01.crt  
cert_export_filial-01.key

### *10.1 - Clique em Files*
![Imagem-1](https://uploaddeimagens.com.br/images/002/827/458/full/1.png?1597336852)

### *10.2 - Clique com o direito no arquivo e em Download*
![Imagem-2](https://uploaddeimagens.com.br/images/002/830/229/original/4.png?1597430188)

### *10.3 – Escolha onde salvar*
![Imagem-3](https://uploaddeimagens.com.br/images/002/830/216/original/3.png?1597429938)

## *11 – Copie os arquivos para o mikrotik da filial*

cert_export_ca.crt  
cert_export_filial-01.crt  
cert_export_filial-01.key  

## *12 – Faça upload dos arquivos para o mikrotik da filial*

### *13 - Clique em Files*
![Imagem-4](https://uploaddeimagens.com.br/images/002/827/458/full/1.png?1597336852)

### *13.1 - Clique em Upload..*
![Imagem-5](https://uploaddeimagens.com.br/images/002/830/159/original/1.png?1597429101)

### *13.2 - Selecione os arquivos e clique em Abrir*
![Imagem-6](https://uploaddeimagens.com.br/images/002/830/197/original/2.png?1597429566)

## *14 - Importe os arquivos para o mikrotik da filial*

/certificate import file-name="cert_export_ca.crt" passphrase=""

/certificate import file-name="cert_export_filial-01.crt" passphrase=senha-cliente

/certificate import file-name="cert_export_filial-01.key" passphrase=senha-cliente

### *15 - Em certificates, veja os certificados que foram importados*
![Imagem-7](https://uploaddeimagens.com.br/images/002/830/248/original/5.png?1597430891)

### *16 - Configure OVPN Client*

/interface ovpn-client add certificate=cert_export_R2.crt_0 cipher=aes256 connect-to=ip_wan_mikrotik_server name=ovpn-filial password=senha-cliente

### *17 - Adicione rota para a rede local do mikrotik do servidor*

/ip route add dst-address=172.16.0.0/24 gateway=10.10.10.1

/ip firewall filter add chain=input dst-port=1194 protocol=1194

/ip firewall nat add chain=srcnat src-address=172.16.2.0/24 dst-address=172.16.0.0/24  