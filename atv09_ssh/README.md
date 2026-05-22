# ATV10 - SSH

### Vitor Hugo Boaventura da Silva

Adaptado do [tutorial de Caian Santana](https://docs.google.com/document/d/1tvqAz4UjqcFmk0kTczYavVjqnMM86z54FhlmwiPCljU/edit?usp=sharing) para as [máquinas virtuais Debian 12](https://drive.google.com/drive/folders/1nGR5_DV5u7J-jm3upk0B82C3EsvlfjlT?usp=sharing).

## Objetivo

A atividade proposta tem como objetivo configurar e testar a comunicação entre duas máquinas virtuais utilizando o protocolo SSH, onde a máquina 1 atuará como servidor SSH e a máquina 2 será o cliente SSH que acessará o servidor para realizar a autenticação e a troca de dados. Para isso, será necessário instalar e configurar o OpenSSH em ambas as máquinas, além de garantir que a máquina 2 consiga estabelecer a comunicação com a máquina 1 via SSH.

_A máquina host pode assumir a função de cliente. Nesse caso, apenas uma máquina virtual é necessária._

## Tutorial da Atividade

_IMPORTANTE: NÃO É NECESSÁRIO UTILIZAR A CONEXÃO SSH COM FINS DE COPIAR E COLAR TEXTO NESSA ATIVIDADE, POIS NÃO HÁ GRANDES TRECHOS DE CÓDIGO OU CONFIGURAÇÃO PARA SEREM DIGITADOS!_


As instruções a seguir foram pensadas para conectar duas máquinas virtuais via SSH. Alternativamente, porém, você pode simplesmente utilizar a máquina host para acessar sua VM via SSH, conforme já mencionado e como aprendemos em atividades anteriores. Nesse caso, a máquina host fará o papel da segunda máquina virtual, que será o cliente.

### 1. Configuração prévia das VMs

Nas configurações de rede, ative apenas um adaptador e configure-o no modo Bridge. Se for utilizar duas máquinas virtuais, as duas devem ter essa configuração.

Inicie as máquinas (ou a máquina) e faça o login.

### 2. Instalando o servidor SSH na Máquina 1

2.1 Instalação do servidor SSH

Se você já acessou essa máquina via SSH em atividades anteriores, o `openssh` já deve estar instalado. Mas, por via das dúvidas, execute o comando:

```bash
apt-get install -y openssh-server
```

2.2 Crie um arquivo com seu nome na `/home` da sua máquina:

```bash
nano /home/seunome.txt
```

Digite um texto qualquer e salve o arquivo (`CTRL + X`, `Y`, `ENTER`).

2.3 Verifique o endereço IPv4 da máquina 1 e guarde-o para usar posteriormente:

```bash
ip a
```

ou:

```bash
ifconfig -a
```

### 3. Instalando o cliente SSH na Máquina 2

Se você resolveu utilizar a máquina host como client, pule para o item 4. Caso contrário, siga as instruções a seguir.

3.1 Instalação do cliente SSH:

```bash
apt-get install -y openssh-client
```

### 4. Acessando a máquina 1 através da máquina 2 via SSH

4.1 Acessando a máquina 1:

```bash
ssh root@<ip_da_maq1>
```

4.2 Agora vá para a `/home` da máquina 1:

```bash
cd /home
```

4.3 Liste os itens deste diretório, você deve ser capaz de ver o arquivo criado com seu nome anteriormente:

```bash
ls
```

4.4 Leia o conteúdo do arquivo:

```bash
cat seunome.txt
```

Se o arquivo existir e contiver o texto que você escreveu, a atividade foi realizada com sucesso.