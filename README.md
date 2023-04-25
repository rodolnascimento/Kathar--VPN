Universidade Federal do Maranhão

**Segurança  em Redes**

# Laboratório - *VPN-Virtual Private Network*

## Objetivo

O objetivo deste experimento consiste em aprender como funciona e como pode ser implementada uma VPN usando o software OpenVPN.

![Topologia de Rede][1]


## Laboratório  -- Criando uma VPN

Uma *Virtual Private Network* (VPN) permite o acesso a partir do exterior a uma rede interna de uma empresa ou organização.
Adicionalmente permite proteger criptograficamente o tráfego que circula nas redes públicas.

nesse experimento vamos configurar o PC1 para se conectar com a rede interna, para ter certeza do sucesso da configuração é possível configurar regras de firewall para impedir acesso da rede externa.

O servidor VPN vai fornecer o serviço OpenVPN, ou seja, os computadores que queriam usar a vpn terão que se conectar a ele. O PC1 vai utilizar o serviço, ou seja, vai ser o cliente OpenVPN.

1. O OpenVPN utiliza chaves assimétricas e certificados para autenticsar os utilizadores.
utilizaremos scripts do OpenVPN para gerar uma CA e certificados.
No servidor VPN mude para o diretório `/etc/openvpn/` e utilize o comando:

```bash
cp -r /usr/share/easy-rsa /etc/openvpn/
cp /etc/openvpn/easy-rsa/vars.example /etc/openvpn/easy-rsa/vars
```

Edite o arquivo com o comando `nano /etc/openvpn/easy-rsa/vars`.
descomente as linhas e inclua o seguinte:

```
set_var KEY_COUNTRY "BRASIL"
set_var KEY_PROVINCE "MARANHAO"
set_var KEY_CITY "Sao Luis"
set_var KEY_ORG "UFMA"
set_var KEY_EMAIL "admin@exemplo.com"
set_var KEY_OU "Organizacao"
```

No diretório `easy-rsa`  utilize o comando para inicializar a PKI: 

```bash
./easyrsa init-pki
```

Utilize o comando para inicializar a CA, criando as suas chaves e certificado:

```bash
./easyrsa build-ca nopass
```

2. Temos agora as chaves da CA.
Vamos criar as chaves e certificados para o servidor e um cliente, não definiremos uma senha nesse exemplo.

```bash
./easyrsa gen-req server nopass
./easyrsa sign-req server server
./easyrsa gen-req client nopass
./easyrsa sign-req client client
```

3. Por fim, o último comando.

```bash
./easyrsa gen-dh
```

4. Comando parara gerar a chave.
```bash
openvpn --genkey --secret ta.key
cp ta.key /etc/openvpn
```

5. No dispositivo PC1 copie o arquivo de exemplo de configuração para a área de root:

```bash
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf ~
```

6. Copie os arquivos que foram gerados no servidor VPN que estão na pasta `/etc/openvpn/easy-rsa/` e suas subpastas para a área do root do PC1: `ca.crt`, `client.crt`, `client.key` e `ta.key`.
uma boa pratica de segurança e deletar o arquvivo `client.key` do servidor pois o mesmo contém a chave privada do cliente, o mesmo vale para o arquivo `server.key` que deve ficar bem protegido.


7. No dispositvo PC1, edite o arquivo`client.conf` e adicione o endereço do servidor VPN.

8. No servidor de VPN, faça uma copia dos arquivos`ca.crt` `dh.pem` `server.key` `server.crt` `ta.key` para a pasta `/etc/openvpn/`.

9. Abra o editor de texto no arquivo `server.conf`. Altere o seguinte:
* Mude `dh1024.pem` para `dh.pem`.
* Descomente a linha `tls-auth ta.key 0`.

10. Note que o arquivo contém a linha:

```bash
server 200.200.200.0 255.255.255.128
```

Isso mostra que a subrede da VPN é a rede `200.200.200.0/25`, ou seja, que um dispotivo de uma rede externa vai aparecer para a rede interna com um endereço IP dessa subrede.

11. Execute o servidor VPN no servidor, a partir do diretório `/etc/openvpn`:

```bash
openvpn server.conf
```

Em caso de erro, utilize os seguintes comandos:
```
mkdir -p /dev/net
mknod /dev/net/tun c 10 200
chmod 600 /dev/net/tun
```


12. Edite o arquivo `client.conf` e descomente a seguinte linha `comp-lzo`.

13. execute o cliente no PC1:

```bash
openvpn client.conf
```

14. Observe as mensagens de serviço iniciado tanto no servidor como no cliente.
Verifique que já estão ligados.

15. Verifique as rotas armazenadas na tabela do router 1 executando o comando `ip route`.
Repare que existe uma rota para a rede `200.200.200.0/25` através do servidor de VPN.
Esta é a rede com os IPs atribuídos aos clientes de VPN.

16. Coloque o openvpn em background com `Ctrl + z` seguido do comando `bg` com o número do processo.
tente a conexão do PC1 com a rede interna da empresa: `192.168.0.0/24`.


---

Referências:

-   *Kathará*, [https://github.com/KatharaFramework/Kathara/wiki]

-   OpenVPN, [https://openvpn.net/community-resources/]


  [1]: media/VPN.png 

