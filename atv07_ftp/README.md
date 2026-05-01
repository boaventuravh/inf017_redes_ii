# Tutorial atividade 07 - FTP

### Vitor Hugo Boaventura da Silva


Adaptado do [tutorial produzido por Caian Santana
 ](https://docs.google.com/document/d/1qwJCIU-pSPB7-klefBeOGk3s-FQBea6X1xyu8QhZEyM/edit?tab=t.0#heading=h.3cod0n8abz08) para as [máquinas virtuais Debian 12](https://drive.google.com/drive/folders/1nGR5_DV5u7J-jm3upk0B82C3EsvlfjlT?usp=sharing).


# Objetivo

Instalar e configurar o vsftpd (servidor FTP) na VM, permitindo acesso remoto a arquivos através do File Transfer Protocol (FTP, Protocolo de Transferência de Arquivo).

---

## Configuração prévia da VM

Anstes de iniciar a máquina virtual, realize as seguintes configurações de rede.

1. Vá em Configurações  
2. Aba Rede  
3. Configure como:

~~~
Adaptador em modo Bridge (Bridge Adapter)
~~~
Se desejar acessar a VM via conexão ssh, siga os dois passos a seguir.

---
### Descobrir IP

Com a máquina virtual funcionando, descubra o ip com o seguinte comando.

~~~bash
ip a
~~~

### SSH

Na máquina host, estabeleça a conexão via ssh com a máquina virtual.

~~~bash
ssh root@<IP_DA_VM>
~~~

# Atividade

---

## 1. Instalação do VSFTPD

Primeiramente, vamos instalar o servidor FTP:

~~~bash
apt install -y vsftpd
~~~

---

## 2. Configuração do VSFTPD

~~~bash
nano /etc/vsftpd.conf
~~~

Acrescente as seguintes informações no arquivo:

~~~
anonymous_enable=NO
local_enable=YES
chroot_local_user=YES
pasv_enable=YES
~~~

Salve as alterações.

Depois, reinicie o servidor FTP:

~~~bash
systemctl restart vsftpd
~~~

---

## 3. Criar usuário FTP

~~~bash
groupadd ftpusers
useradd -g ftpusers ftpuser
passwd ftpuser
mkdir -p /home/ftpuser/public
chown -R ftpuser:ftpusers /home/ftpuser
chmod a-w /home/ftpuser
ls -l /home
touch /home/ftpuser/public/teste.txt
~~~

---

## 4. Teste com FileZilla

Preencha:

- Host: IP da VM  
- Usuário: ftpuser  
- Senha: definida  
- Porta: 21  

Resultado esperado:

~~~
Status: Connected
Status: Logged in
Status: Directory listing successful
~~~

Verifique o arquivo:

~~~
/public/teste.txt
~~~

---

# Conclusão

Servidor FTP funcionando com:

- Autenticação local
- Sem acesso anônimo
- Diretório isolado
- Modo passivo ativo
