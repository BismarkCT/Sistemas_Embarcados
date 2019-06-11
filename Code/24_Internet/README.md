# Conexão à internet

## Modelos do Raspberry Pi sem WiFi

Conecte o cabo ethernet ao RPi. Ele deve se configurar automaticamente para acessar a internet.

## Modelos do Raspberry Pi com WiFi

Execute ``` sudo iwlist wlan0 scan ``` para conferir todas as redes WiFi disponíveis. (Execute ``` sudo iwlist wlan0 scan | grep SSID``` para obter somente os nomes das redes disponíveis.)

Abra o arquivo ``` /etc/wpa_supplicant/wpa_supplicant.conf ```, e acrescente o seguinte texto ao final dele:

```
network={
    ssid="NOME_DA_REDE"
    psk="SENHA_DA_REDE"
}
```
onde ```ssid``` é o nome da rede, e ```psk``` é a senha da rede.

Se quiser criptografar a senha, execute ```wpa_passphrase NOME_DA_REDE``` e digite a senha da rede. Escreva no arquivo ``` /etc/wpa_supplicant/wpa_supplicant.conf ``` o texto retornado no terminal:

```
network={
      ssid="testing"
      #psk="testingPassword"
      psk=131e1e221f6e06e3911a2d11ff2fac9182665c004de85300f9cac208a6a80531
  }
```

(Retire o texto precedido pelo símbolo ```#``` para maior segurança.)

Na UnB, utilize a rede ```eduroam``` com a seguinte configuração:

```
network={
   ssid="eduroam"
   proto=RSN
   key_mgmt=WPA-EAP
   eap=PEAP
   identity="SEU_EMAIL@unb.br"
   password="SUA_SENHA"
   phase1="peaplabel=0"
   phase2="auth=MSCHAPV2"
}
```

onde os campos ```identity``` e ```password``` contém seu e-mail unb.br e sua senha.

# _Download_ de arquivos, sites etc.

A maneira mais simples de baixar arquivos da internet pelo RPi é utilizando o comando ```wget```. Por exemplo,

```
wget https://raw.githubusercontent.com/DiogoCaetanoGarcia/Sistemas_Embarcados/master/README.md
```

baixa o Plano de Ensino desta disciplina, e

```
wget www.unb.br
```

baixa o arquivo HTML da página principal da UnB.

O comando ```curl``` oferece muito mais opções do que o ```wget```, sendo bastante interessante no que veremos mais à frente. Execute 

```
curl -O https://raw.githubusercontent.com/DiogoCaetanoGarcia/Sistemas_Embarcados/master/README.md
curl -O www.unb.br
```

para obter os mesmos resultados, ou

```
curl https://raw.githubusercontent.com/DiogoCaetanoGarcia/Sistemas_Embarcados/master/README.md
curl www.unb.br
```

para que os arquivos baixados não sejam salvos localmente, e sim apresentados no terminal. Alternativamente, execute

```
curl -o saida1.html https://raw.githubusercontent.com/DiogoCaetanoGarcia/Sistemas_Embarcados/master/README.md
curl -o saida2.html www.unb.br
```

para salvar os resultados nos arquivos locais ```saida1.html``` e ```saida2.html```, respectivamente.

Acrescente a opção ```-v``` para conferir o pedido HTTP feito no pedido da página (por exemplo, ```curl -v -o saida2.html www.unb.br```). Se você quiser diminuir a quantidade de informação apresentada na tela, acrescente a opção ```-s``` (por exemplo, ```curl -s -o saida2.html www.unb.br```).

## _Download_ de arquivos em serviços em nuvem (Dropbox, Google Drive, Microsoft OneDrive etc.)

Para fazer o _download_ de arquivos em serviços de armazenamento em nuvem, tais como o Dropbox, o Google Drive e o Microsoft OneDrive, pode ser necessário realizar alguns passos extra. Antes de mais nada, é necessário liberar o acesso ao arquivo no serviço em nuvem.

Considere o arquivo ```Arquivo_na_nuvem.pdf```, disponível em https://drive.google.com/file/d/11uI5-BAXzIhKO14clUwmQlAo8LyqS-Ui/view?usp=sharing. É necessário separar nesta URL o identificador do arquivo (uma sequência aparentemente aleatória de caracteres) para fazer o download adequadamente:

```
filename="Arquivo_na_nuvem1.pdf"
fileid="11uI5-BAXzIhKO14clUwmQlAo8LyqS-Ui"
curl -L -o ${filename} "https://drive.google.com/uc?export=download&id=${fileid}"
```

Se o arquivo tiver mais de 100MB, o Google Drive pede uma espécie de autenticação antes de permitir o _download_ do mesmo (por exemplo, em https://drive.google.com/file/d/1EPVcIVl0UiNtjw9S751ZRiw_FQdTLHf0/view?usp=sharing). Nesse caso, execute:

```
filename="Arquivo_na_nuvem2.pdf"
fileid="1EPVcIVl0UiNtjw9S751ZRiw_FQdTLHf0"
curl -L -c cookies.txt 'https://docs.google.com/uc?export=download&id='$fileid | sed -rn 's/.*confirm=([0-9A-Za-z_]+).*/\1/p' > confirm.txt
curl -L -b cookies.txt -o $filename 'https://docs.google.com/uc?export=download&id='$fileid'&confirm='$(<confirm.txt)
rm -f confirm.txt cookies.txt
```

Procedimento semelhante pode ser necessário com o Dropbox e o OneDrive.

## Buscas pelo Google

Para buscar pelo termo ```raspberry pi``` em ```www.google.com```, por exemplo, execute

```
curl -o saida.html https://www.google.com/search?q=raspberry+pi
```

e abra em um _browser_ o arquivo salvo ```saida.html```.

Provavelmente você verá uma mensagem dizendo que você não teve permissão para fazer esta busca (por exemplo, _Your client does not have permission to get URL /search?q=raspberry+pi from this server._). Isso acontece porque o Google procura impedir buscas automatizadas como esta que você acabou de tentar.

Execute ```curl -v www.unb.br > /dev/null```. No pedido HTTP, você verá um campo indicando qual é o agente que está fazendo o pedido. O ```curl``` geralmente preenche este campo como ```curl\VERSAO_DO_CURL``` (por exemplo, ```User-Agent: curl\7.58.0```). O Google provavelmente colocou quaisquer agentes que contenham o nome ```curl``` na sua 'lista negra'. Para conseguir sair da 'lista negra', acrescente a opção ```-A USER-AGENT``` ao comando anterior:

```
curl -A USER-AGENT -o saida.html https://www.google.com/search?q=raspberry+pi
```

e abra em um _browser_ o arquivo salvo ```saida.html``` para ver a busca disponibilizada. (Se você não conseguir permissão novamente, provavelmente o agente ```USER-AGENT``` também entrou na lista negra do Google. Troque o nome do agente até conseguir.)

## Hora mundial

Para conferir a hora atual via IP da máquina, execute ```curl http://worldtimeapi.org/api/ip.txt```.

Para conferir a hora atual via o fuso-horário de Brasília/São Paulo, execute ```curl http://worldtimeapi.org/api/timezone/America/Sao_Paulo.txt```.

Para conferir os fuso-horários disponíveis, execute ```curl http://worldtimeapi.org/api/timezone.txt```.

# Envio de informações (formulários, arquivos etc.)

Páginas web possuem diversos campos destinados a formulários. Por exemplo, a página de _login_ de uma página geralmente possui campos para usuário e senha, além de um botão para envio destas informações. Barras de busca são outro exemplo relevante.

Quando se baixa uma página na internet, é feita uma requisição HTTP GET. No caso do envio destas informações de formulários, é feita uma requisição HTTP POST. O comando ```curl``` permite automatizar este tipo de requisição.

Por exemplo, entre na página https://fga.unb.br, e faça uma busca na barra correspondente pela palavra "eletronica". Você será redirecionado para a página https://fga.unb.br/search/articles?query=eletronica. Uma forma simples de automatizar essa busca é executando

```
curl -v -o exemplo_POST.txt -d query=eletronica https://fga.unb.br/search/articles
cat exemplo_POST.txt
```

Repare na saída do comando ```curl``` que foi feita uma requisição POST.

Pelo resultado da busca, pode-se perceber que uma requisição GET também é suficiente:

```
curl -v -o exemplo_POST.txt https://fga.unb.br/search/articles?query=eletronica
```

## Atualização de serviços em nuvem

A instalação de aplicativos de serviços em nuvem pode ser uma solução simples, mas custosa computacionalmente, já que estes aplicativos continuamente conferem se houve mudanças em suas pastas principais e secundárias. Para fazer o _upload_ de arquivos pelo terminal, cada serviço oferece uma solução diferente:

* https://www.dropbox.com/developers/documentation/http/documentation
* https://riptutorial.com/dropbox-api/example/1356/uploading-a-file-via-curl
* https://olivermarshall.net/how-to-upload-a-file-to-google-drive-from-the-command-line/



# E-mail

O comando ```curl``` trabalha com diversos protocolos, incluindo SMTP. Assim, ele permite o envio de e-mails. Por exemplo,

```
curl smtp://mail.example.com --mail-from myself@example.com --mail-rcpt
receiver@example.com --upload-file email.txt
```

envia um e-mail do endereço ```myself@example.com``` para o endereço ```receiver@example.com```, onde o corpo do e-mail se encontra no arquivo ```email.txt```:

```
From: John Smith <john@example.com>
To: Joe Smith <smith@example.com>
Subject: an example.com example email
Date: Mon, 7 Nov 2016 08:45:16
Dear Joe,
Welcome to this example email. What a lovely day.
```

## Instalação

```
sudo apt-get install mailutils
sudo apt-get install ssmtp
```

## Referências

* https://weworkweplay.com/play/automatically-connect-a-raspberry-pi-to-a-wifi-network/
* https://curl.haxx.se/book.html
* https://www.hostinger.com.br/tutoriais/comando-curl-linux/
* https://www.matthuisman.nz/2019/01/download-google-drive-files-wget-curl.html
* http://worldtimeapi.org/