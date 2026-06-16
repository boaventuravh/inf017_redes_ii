# Atividade Zabbix

### Vitor Hugo Boaventura da Silva

Adaptado do [tutorial de Caian Santana](https://docs.google.com/document/d/19EaYKcBe11STBm0BLIbhcKSmllEtAZZUojjgXgMzMqc/edit?usp=sharing) para as [máquinas virtuais Debian 12](https://drive.google.com/drive/folders/1nGR5_DV5u7J-jm3upk0B82C3EsvlfjlT?usp=sharing).


## Objetivo

A atividade proposta tem como objetivo instalar e configurar o Zabbix em um ambiente de monitoramento, proporcionando um breve overview sobre a ferramenta e suas funcionalidades. A tarefa consistirá em instalar o servidor Zabbix, o agente Zabbix e a interface web em uma máquina virtual, a fim de monitorar o desempenho e a disponibilidade de dispositivos em uma rede.

## Tutorial da Atividade

### Configuração prévia das VMs

Nas configurações de rede, ative apenas um adaptador e configure-o no modo Bridge.

Inicie a máquina virtual.

Se desejar acessar a máquina virtual via conexão SSH (recomendado), descubra o IP da VM com `ip a` ou `ifconfig -a`. Na máquina host:

```bash
ssh root@<IP_DA_VM>
```


### 1. Instalando o Zabbix

1.1 Mude os repositórios, incluindo o do Zabbix:

```bash
wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.0+debian12_all.deb

dpkg -i zabbix-release_latest_7.0+debian12_all.deb

apt-get update
```

1.2 Instale o servidor Zabbix, o frontend,o agente e o SGBD MariaDB:

```bash
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-agent mariadb-server
```

### 2. Configurando o MySQL

2.1 Inicie e proteja o MariaDB:

```bash
systemctl enable --now mariadb
mysql_secure_installation
```

Quando executar mysql_secure_installation, responda:

_Switch to unix_socket authentication? Y_

_Change the root password? N_

_Remove anonymous users? Y_

_Disallow root login remotely? Y_

_Remove test database and access to it? Y_

_Reload privilege tables now? Y_


2.2 Logue como root no MySQL:

```bash
mysql -u root -p
```

2.3 Dentro do MySQL, crie um banco para o Zabbix:

```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;

CREATE USER 'zabbix'@'localhost' IDENTIFIED BY '1234';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

2.4 Importe o esquema do banco de dados:

```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -p zabbix
```

### 3. Configure o Zabbix

3.1 Edite o arquivo de configuração:

```bash
nano /etc/zabbix/zabbix_server.conf
```

3.2 Dentro do arquivo, altere os parâmetros (para procurar itens dentro do arquivo, aperte Ctrl+W no nano):

```text
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=1234
```

### 4. Configure o PHP

4.1 Edite o arquivo de configuração do PHP:

```bash
nano /etc/php/8.2/apache2/php.ini
```

4.2 Dentro do arquivo, procure por `date.timezone` com o comando `Ctrl + W` e defina-o da seguinte maneira:

```text
date.timezone = America/Bahia
```

### 5. Reiniciando os serviços

5.1 Reinicie e habilite os serviços Apache, Zabbix Server e Agent:

```bash
systemctl restart apache2
systemctl restart zabbix-server
systemctl restart zabbix-agent
```

```bash
systemctl enable apache2
systemctl enable zabbix-server
systemctl enable zabbix-agent
```

### 6. Acessando o Zabbix

6.1 Descubra o endereço IPv4 da sua máquina:

```bash
ip a
```

ou

```bash
ifconfig -a
```

6.2 Em um navegador, utilize o IP na seguinte URL:

```text
http://ip_da_maquina/zabbix
```

6.3 Prossiga, introduza a senha definida para o MySQL (`1234`) e logue com as seguintes credenciais:

```text
login: Admin
senha: zabbix
```

Se você conseguir acessar a interface de usuário do Zabbix, a atividade foi concluída com sucesso.
