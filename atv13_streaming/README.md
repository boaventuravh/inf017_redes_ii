# Atividade Serviços de Streaming, QoS e QoE

### Vitor Hugo Boaventura da Silva e [Caian Santana](https://github.com/CaianSantana)

Versão adaptada [deste tutorial](https://docs.google.com/document/d/1B6aRyR8DT_dwf3-QA0XwpbW0eJUVSqF0oGoOoP_bQ5k/edit?usp=sharing).

# Objetivo

A atividade proposta consiste em utilizar o FFmpeg para realizar o streaming de um vídeo
através do NGINX utilizando o protocolo RTMP. O NGINX será configurado para atuar como
servidor RTMP, permitindo que o FFmpeg envie o conteúdo de vídeo para o servidor, que
estará disponível para consumo remoto. O fluxo de vídeo poderá ser visualizado por meio
do FFplay ou VLC, garantindo a flexibilidade na escolha do reprodutor. A configuração do
NGINX envolverá a instalação do módulo RTMP e a criação de uma estrutura adequada de
diretórios para armazenar os dados do stream.

A atividade também inclui a verificação da conectividade do servidor de streaming, tanto
localmente quanto de máquinas externas, testando a qualidade do stream e a latência da
rede. Como parte do processo de análise e diagnóstico, opcionalmente será realizada a
captura de pacotes utilizando o Wireshark para monitorar a transmissão de dados do RTMP,
permitindo a análise de possíveis problemas de rede e o comportamento do protocolo
durante a transmissão. Todo o processo ocorrerá dentro de uma VM rodando Ubuntu 20.04,
com o Wireshark e o vídeo sendo visualizados na máquina host.


Para esta atividade, utilizaremos a Máquina Virtual 04(ubuntu 20.4) :
https://www.dropbox.com/scl/fi/pltkqimkjzgd8d6f0p8nj/maq4.ova?rlkey=3kojkxn3cs0l
bt1xxq4aeiik3&e=1&st=wa8yu3lv&dl=

# Revisão de conceitos e ferramentas

**RTMP (Real-Time Messaging Protocol):** Protocolo desenvolvido pela Adobe para
transmissão de áudio, vídeo e dados em tempo real. É utilizado para streaming ao vivo,
permitindo que o conteúdo seja transmitido de um servidor para um cliente de forma
contínua e interativa.

**NGINX:** Servidor web de código aberto que, com o módulo RTMP, pode ser configurado
para gerenciar e distribuir streams de áudio e vídeo via RTMP. Ele recebe o conteúdo
enviado pelo FFmpeg e o disponibiliza para os clientes que consomem o stream.

**FFmpeg:** Ferramenta de linha de comando que permite capturar, converter e transmitir
conteúdo de mídia. No contexto da atividade, o FFmpeg é utilizado para codificar e enviar o
fluxo de vídeo para o servidor NGINX via RTMP.

**FFplay** : Player multimídia de código aberto que pode ser usado para consumir o stream
transmitido pelo servidor NGINX, permitindo a visualização do conteúdo em tempo real.

**VLC:** Player de mídia popular e de código aberto que também pode ser utilizado para
consumir streams RTMP, proporcionando uma interface amigável para assistir ao conteúdo
transmitido.

**Wireshark:** Ferramenta de captura e análise de pacotes de rede, utilizada para monitorar o
tráfego gerado pelo RTMP e diagnosticar possíveis problemas de rede ou desempenho
durante a transmissão do vídeo.

# Tutorial da Atividade


## 2. Configuração prévia da VM

Para esta atividade, utilizaremos apenas uma placa de rede, que deverá ser configurada em _Modo Bridge_

## 3. Logando na VM

_login: osboxes_

_senha: osboxes.org_

Para acessar esta máquina via SSH, será necessário fazer uma breve configuração antes para autorizar o acesso. Na VM:

```bash
apt update
apt install ssh
ufw allow 22
```
Depois, confira o IP da VM com `ip a`. Na máquina host:

```bash
ssh osboxes@192.168.0.15 
```
Essa máquina não permite login no modo root. Então use o comando `sudo -i` (senha: osboxes.org) para evitar a necessidade de adicionar _sudo_ e digitar senha em todos os comandos.

## 4. Pré-requisitos

4.2. Atualize a lista de pacotes e instale o Nginx, ffmpeg e suas
dependências:

```bash
sudo apt-get update
sudo apt install nginx libnginx-mod-rtmp ffmpeg
```

4.3. Verifique a instalação:

```bash
sudo systemctl status nginx
ls /usr/lib/nginx/modules/ | grep rtmp
```
4.4. Configure o nginx:

```bash
sudo nano /etc/nginx/nginx.conf
```

Adicione no final do arquivo:

```
rtmp {
    server {
        listen 1935;
        chunk_size 4096;
        allow publish 127.0.0.1;
        deny publish all;

        application live {
            live on;
            record off;
        }
    }
}
```

4.5. Reinicie o nginx:

```bash
sudo systemctl restart nginx
```

Instação do python:

```bash
sudo apt-get install -y build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev libsqlite3-dev wget libbz2-dev
```

Baixar versão específica 
```bash
wget https://www.python.org/ftp/python/3.10.0/Python-3.10.0.tgz
```

Descompactar

```bash
tar -xf Python-3.10.0.tgz
```
acesse a pasta

```bash
cd Python-3.10.0
```

Rode o comando `configure` para habilitar otimizações:
```bash
./configure --enable-optimizations
```

Utilize o comando make:

```bash
make -j $(nproc)
```

Para manter a instalação pré-instalada do Python no Ubuntu:

```bash
make altinstall
```

Agora retorne ao diretório anterior e cheque se a versão instalada é a desejada:

```bash
cd ..
python3.10 --version
```

O retorno esperado é `Python 3.10.0`.

Agora, instale o downloader de vídeos `yt-dlp` com o comando a seguir para garantir que a versão seja compatível com o python:

```bash
python3.10 -m pip install -U yt-dlp

```
4.6. Baixe o video(pode ser qualquer um, link meramente de exemplo):

```bash
yt-dlp https://www.dailymotion.com/video/x9e4gdi
```

## 5. Streamando e consumindo o video:

```
5.1. Streame o video com ffmpeg:
ffmpeg -re -i "nome_do_video.mp4" -c:v copy -c:a aac -ar
44100 -ac 1 -f flv rtmp://localhost/live/stream
5.2. Na máquina host, instale o ffmpeg:
winget install "FFmpeg (Essentials Build)"
5.3. Abra a stream usando ffplay:
ffplay rtmp://<ip_da_vm>/live/stream
```

## 6. Utilizando Wireshark:

```
6.1. Na máquina host, instale o Wireshark. Caso não possa instalar, utilize
a versão portátil. Se estiver no linux, instale a seguinte maneira:
sudo apt install wireshark
6.2. Após instalado, abra o wireshark como adm:
sudo wireshark
6.3. Ao abrir, escreva na barra de filtro “port 1935” e aperte em iniciar no
canto superior esquerdo(simbolo de barbatana azul), ele começará a
capturar os pacotes da stream:
```
