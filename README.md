# Como usar Plugins VST no linux

## Descrição do artigo

Este artigo descreve passo a passo como preparar a sua distribuição linux para trabalhar com produção musical, instalando e configurando todos componentes necessários no seu so linux para que você possa fazer suas produções em seu home studio. Escolhi aqui o [xubuntu](https://xubuntu.org/) por ser uma distribuição leve e fluida. Excelente para quem deseja ter um bom desempenho e pouco consumo de recursos da máquina.

## Escopo da Instalação

1. Instalação do kernel de baixa latência
2. Melhorando a o desempenho do sistema com zram (Opicional)
3. Instalação do repositório KxStudio
4. Instalação do Cadence (Jack)
5. Habilitando a inteface de audio e controlador midi o cadence
6. Adicionando o usuário ao grupo audio
7. Instalação do wine
8. Instalando os prefixos para rodar os plugins
9. Instalação do Reaper
10. Instalação do yabridge
11. Instalação dos Plugins
12. Habilitando controlador midi no Reaper

## 1. Instalação do kernel de baixa latência

Para entender melhor o porque do uso do kernel de baixa latência acesse o seguinte artigo no [link](https://sempreupdate.com.br/o-que-e-um-kernel-de-baixa-latencia-real-time-preemptividade/).
O kernel usado aqui foi o xanmod. Você pode conhecer mais o projeto acessando o [link](https://xanmod.org/).

Adicione o repositório:

```bash
echo 'deb http://deb.xanmod.org releases main' | sudo tee /etc/apt/sources.list.d/xanmod-kernel.list && wget -qO - https://dl.xanmod.org/gpg.key | sudo apt-key add -
```

Atualize o sistema e instale o kernel:

```bash
sudo apt update && sudo apt install linux-xanmod-rt-edge
```

### 1.2 Instalação de CPU Microcodes:

Para processadores Intel:

```bash
sudo apt install intel-microcode iucode-tool
```

Para processadores AMD:

```bash
sudo apt install amd64-microcode
```

## 2. Melhorando o desempenho do sistema com zram

O ZRAM é um utilitário relativamente conhecido dos usuários Linux, ele permite um melhore gerenciamento de memória em relação a partição de SWAP. Para entender melhor o uso do zram acesse o [link](https://diolinux.com.br/2016/08/como-instalar-o-zram-no-ubuntu-e-outras-dicas-para-melhorar-o-desempenho.html).

```bash
sudo apt-get install zram-config
```

## 3. Instalação do repositório KxStudio

O KXStudio é uma coleção de aplicativos e plugins para produção de áudio. O KXStudio também fornece repositórios compatíveis com o Debian (e Ubuntu). Para conhecer mais sobre o projeto acesse o [link](https://kx.studio/).

Instale as dependências necessárias:

```bash
sudo apt-get install apt-transport-https gpgv
```

Remover repositórios instalados:

```bash
sudo dpkg --purge kxstudio-repos-gcc5
```

Baixar arquivo de pacote para instalação:

```bash
wget https://launchpad.net/~kxstudio-debian/+archive/kxstudio/+files/kxstudio-repos_10.0.3_all.deb
```

Atualizar o sistema

```bash
sudo apt update && sudo apt upgrade
```

Instalar o pacote:

```bash
sudo dpkg -i kxstudio-repos_10.0.3_all.deb
```

## 4. Instalação do Cadence (Jack)

Cadence é um conjunto de ferramentas úteis para produção de áudio. Ele realiza verificações do sistema, gerencia o JACK, chama outras ferramentas e faz ajustes no sistema.

```bash
sudo apt-get update && sudo apt-get install cadence
```

## 5. Habilitando a inteface de audio e controlador midi o cadence

Nesse passo vamos definir qual interface de audio será usada no jack e abilitar o driver para o controlador midi. A partir daí o Reaper poderá encontrar tanto a interface de audio como o controlador midi em suas configurações.

![](https://i.imgur.com/GS8jtDn.gif)

## 6. Adicionando o usuário ao grupo audio

Para que o seu usuário tenha acesso ao cadence e as configurações de audio adicione ele ao grupo audio.

```bash
sudo gpasswd -a ${USER} audio
```

## 7. Instalação do wine

```bash
wget -nc https://dl.winehq.org/wine-builds/winehq.key
sudo apt-key add winehq.key
sudo apt-add-repository 'https://dl.winehq.org/wine-builds/ubuntu/'
sudo apt update
sudo apt install wine-stable
sudo apt install winetricks
```

## 8. Instalando os prefixos para rodar os plugins

```bash
winetricks dotnet40 mfc40 mfc42 vcrun2008 vcrun2010 vcrun2013 vcrun2017
```

## 9. Instalação do Reaper

```bash
wget -c http://reaper.fm/files/6.x/reaper611_linux_x86_64.tar.xz #update_version
sudo apt install xz-utils
tar -xvf reaper611_linux_x86_64.tar.xz
cd reaper_linux_x86_64/
sh install-reaper.sh
wget -c https://landoleet.org/old/reaper_sws_x86_64_fc28caa7.tar.xz
tar -xvf reaper_sws_x86_64_fc28caa7.tar.xz
mkdir -p ~/.config/REAPER/UserPlugins/
cp reaper_sws64.so ~/.config/REAPER/UserPlugins/
wget -c https://github.com/cfillion/reapack/releases/download/v1.2.2/reaper_reapack64.so
cp reaper_reapack64.so ~/.config/REAPER/UserPlugins/
```

## 10. Instalação do yabridge

O yabridge faz a conversão dos plugins .dll para .so, permitindo que os mesmos sejam visualizados nativamente no reaper.

link do repositório do [yabridge](https://github.com/robbert-vdh/yabridge)

Baixe e instale o pacote

```bash
wget -c https://github.com/robbert-vdh/yabridge/releases/download/3.8.0/yabridge-3.8.0-ubuntu-18.04.tar.gz

```

Adicione ao final do seu arquivo `~/.zshrc` ou `~/bashrc` a seguinte linha:
`PATH="$PATH:$HOME/.local/share/yabridge"`

Adicione o path do direrótio onde estão os plugins, como no exemplo abaixo:

```bash
yabridgectl add "$HOME/.wine/drive_c/Program Files/Steinberg/VstPlugins"
```

E finalmente sincronize a pasta:

```bash
yabridgectl sync
```

## 11. Instalação dos Plugins

Para exemplificar a instalação de um instrumento virtual vou usar um piano free 4Front Piano que pode ser baixado no [site](http://www.vst4free.com/free_vst.php?plugin=CVPiano&id=382).

Primeiro vamos criar um diretório para armazenar nossos plugins. Será nesse diretório que o REAPER fará a busca e carregará VST's dentro da dawn.

```bash
mkdir ~/.vst3
```

Agora vamos baixar e instalar o plugin para convertermos e importarmos no reaper.

```bash
wget -c http://www.vst4free.com/get_plug.php\?win64\=4Front_Piano_x64.zip
unzip get_plug.php\?win64=4Front_Piano_x64.zip
```

Copie o arquivo .dll baixado para o diretótio .vst3 que criamos anteriormente. Esse processo pode ser feito tanto via linha de comando como pelo gerenciador de arquivos.

```bash
cp 4Front\ Piano\ x64.dll ~/.vst3
```

Agora vamos fazer com que o reaper reconheça o plugin como se fosse nativo linux usando o linvst.
Vou mostrar como fazer primeiramento via linha de comando.

```bash
cp /opt/linvst.so ~/.vst3/4Front\ Piano\ x64.so
```

Uma outra maneira é usando o linvst com o lançador que criamos:

![](https://i.imgur.com/yVZKCC2.gif)

## 12. Habilitando controlador midi no Reaper

Para que haja comunicação entre a dawn e o teclado controlador precisamos ir até o menu de configurações do Reaper e habilitar o controle de mensagens.

![](https://imgur.com/jNK0jyI.gif)

Voltarei em breve com mais conteúdo, fico a disposição a perguntas e dúvidas, abraços!
