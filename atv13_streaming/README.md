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


## 1. Configuração prévia da VM

Para esta atividade, utilizaremos apenas uma placa de rede, que deverá ser configurada em _Modo Bridge_

## 2. Logando na VM

_login: osboxes_

_senha: osboxes.org_

Essa máquina não permite login no modo root. Então use o comando `sudo -i` (senha: osboxes.org) para evitar a necessidade de adicionar _sudo_ e digitar senha em todos os comandos.

Para acessá-la via SSH, será necessário fazer uma breve configuração antes para autorizar o acesso. Na VM:

```bash
apt update
apt install ssh
ufw allow 22
```
Depois, confira o IP com `ip a`. Na máquina host:

```bash
ssh osboxes@<ip_da_vm> 
```

2.2. Atualize a lista de pacotes e instale o Nginx, ffmpeg e suas
dependências:

```bash
sudo apt-get update
sudo apt install nginx libnginx-mod-rtmp ffmpeg
```

2.3. Verifique a instalação:

```bash
sudo systemctl status nginx
```

```bash
ls /usr/lib/nginx/modules/ | grep rtmp
```

2.4. Configure o nginx:

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

2.4. Reinicie o nginx:

```bash
sudo systemctl restart nginx
```
## 3 Salvando um vídeo na VM

Para criar um servidor streaming, precisaremos de arquivo de vídeo salvo na VM. Para isso:

* Baixe um vídeo qualquer da internet na sua máquina host. 
* Siga o passo a passo do tutorial da [atividade de FTP](https://github.com/boaventuravh/inf017_redes_ii/tree/main/atv07_ftp) para transferir arquivos da máquina host para a VM. Porém, não execute o comando `chmod a-w /home/ftpuser`, pois ele remove a permissão de escrita.
* Estabeleça a conexão FTP no Filezilla.
* Arraste o vídeo salvo na máquina host para a pasta criada para o usuário `ftpuser`.

Para verificar se o vídeo foi salvo na VM, entre na pasta ftpuser e use o comando `ls`.

## 4. Transmitindo e consumindo o vídeo:


4.1. Transmita o vídeo com ffmpeg:

```bash
ffmpeg -re -i "nome_do_video.mp4" -c:v copy -c:a aac -ar
44100 -ac 1 -f flv rtmp://localhost/live/stream
```

4.2. Na máquina host, instale o ffmpeg:


```powershell
winget install "FFmpeg (Essentials Build)"
```

4.3. Abra a stream usando ffplay:

```
ffplay rtmp://<ip_da_vm>/live/stream
```

Alternativamente, é possível utilizar o reprodutor de mídia VLC, clicar na opção _Mídia_, depois _abrir transmissão de rede_ e colar o link _rtmp://<ip_da_vm>/live/stream_.

## 5. Utilizando Wireshark:


5.1. Na máquina host, instale o Wireshark. Caso não possa instalar, utilize
a versão portátil. Se estiver no linux, instale a seguinte maneira:

```bash
sudo apt install wireshark
```

5.2. Após instalado, abra o wireshark como adm:

```bash
sudo wireshark
```
5.3. Ao abrir, escreva na barra de filtro “port 1935” e aperte em iniciar no canto superior esquerdo(simbolo de barbatana azul), ele começará a capturar os pacotes da stream.

