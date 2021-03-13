# Replit com Nginx

O [Replit](https://replit.com) é um IDE, editor, compilador, intérprete e REPL online. O Replit tem suport para diversas tecnologias, porém não tem suporte ao Nginx, o objetivo deste projeto é explorar esta limitação e fazer com que o Replit execute o Nginx.

O REPL deste projeto está disponível em: [https://replit.com/@GuilhermeRodri8/replitwithnginx](https://replit.com/@GuilhermeRodri8/replitwithnginx)

## Sobre o ambiente linux do Replit

### Sistema Operacional

Após criado um REPL com a linguagem Bash, podemos ir no Shell e digitar o comando `cat /etc/os-release` para descobrir qual a versão do Sistema Operacional. Para a data de escrita deste artigo a saída obtida foi:

```txt
~$ cat /etc/os-release
NAME="Ubuntu"
VERSION="18.04.5 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.5 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
```

Como podemos ver trata-se de um abiente com o Sistema Operacional Ubuntu.

### Limitações

Neste ambiente do Replit possuimos acesso apenas ao usuário `runner` que não tem permissões `root`. Estamos limitados também ao acesso apenas do diretório home (`home/$USER`) deste usuário e do diretório de aquirvos temporários do sistema `/tmp`.

Essas limitações podem ser visualizadas quando tentamos instalar um pacote no ambiente ou criar um arquivo fora do diretório home do usuário `runner`:

```txt
~$ sudo apt-get update
bash: sudo: command not found

~$ apt-get update
Reading package lists... Done
E: List directory /var/lib/apt/lists/partial is missing. - Acquire (30: Read-only file system)

~$ apt-get install zip
E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?

~$ cd /opt
/opt$ mkdir meu-software
mkdir: cannot create directory ‘meu-software’: Read-only file system
```

## Roteiro

- Gerar binário portatil do NGINX para o ubuntu
- Configurar o Replit para executar o binário do NGINX gerado 


### Compilando o NGINX a partir do código fonte

**Referências**

- http://nginx.org/en/docs/configure.html
- https://www.nginx.com/resources/wiki/start/topics/tutorials/installoptions/
- https://nginx.org/en/download.html

Vamos usar a versão estável do Nginx que no momento da escrita deste artigo é a nginx-1.18.0

Podemos fazer o download do código fonte do Nginx direto do site oficial (https://nginx.org/en/download.html)[https://nginx.org/en/download.html]

Selecionando a versão para linux, teremos um link para download parecido com este: (https://nginx.org/download/nginx-1.18.0.tar.gz)[https://nginx.org/download/nginx-1.18.0.tar.gz]

Como mencionado na seção **Limitações** o ambiente do Replit não possui permissões de `root`, e para realizar a compilação do Nginx precisamos instalar alguns pacotes adicionais necessários para o build. Então teremos que compilar o Nginx em um ambiente fora do Replit que tenhamos acesso ao root do Sistema. Para isso optei pelo uso da imagem do docker `ubuntu:18.04` que é o mesmo ambiente do Replit porém agora temos permissões `root`.

Para executar a imagem do docker é necessário que sua máquina possua o `docker` e o `docker-compose` instalados.

Na pasta `ubuntu-docker` deste projeto há um Dockerfile que gera uma imagem com todas as dependencias necessárias para compilarmos o Nginx. Para executa-la use o comando `docker-compose run ubuntu` dentro da pasta `ubuntu-docker` para abrir o shell do ubuntu.

_Dica: para um shell colorido, você pode usar o comando:_ `export PS1="\[\e[32m\][\[\e[m\]\[\e[31m\]\u\[\e[m\]\[\e[33m\]@\[\e[m\]\[\e[32m\]\h\[\e[m\]:\[\e[36m\]\w\[\e[m\]\[\e[32m\]]\[\e[m\]\[\e[32;47m\]\\$\[\e[m\] "`

Observe que existe uma pasta chamada `host`, esta pasta é um volume dentro do ubuntu no caminho `/host`, assim todos os arquivos dentro desta pasta estão acessivel dentro e fora do container do docker.

No shell do ubuntu vamos executar a seguinte sequencia de comandos para compilarmos o Nginx:

```sh
# Acessando o volume compartilhado
cd /host

# Download do Nginx e salvando no arquivo com nome de nginx.tar.gz
curl -L --output nginx.tar.gz https://nginx.org/download/nginx-1.18.0.tar.gz

# Descomprir o arquivo para um diretorio
tar -xvf nginx.tar.gz

# Acessando o diretorio que foi criado
cd ./nginx-1.18.0

# Configurando o build do Nginx
./configure --error-log-path=stderr --http-log-path=/dev/stdout --prefix=/tmp/local/nginx

# Compilando o Nginx. Arquivos `nginx` gerado no diretorio `objs`
make
```

Após a execução dos comandos acima com sucesso, teremos gerado o binário do Nginx. O binário é o arquivo `nginx` gerado no diretorio `objs`.

Antes de proseguirmos, vamos analisar os dois últimos comandas executados, `./configure` e `make`

O comando `./configure` está documentado aqui: [Installation and Compile-Time Options](https://www.nginx.com/resources/wiki/start/topics/tutorials/installoptions/). Para enteder os parâmentros que passamos para esse comando, precisamos lembrar das limitações do ambiente Replit. No ambiente Replit temos acesso apenas a pasta `/tmp` e a pasta do usuario `runner`, o Nginx por padrão usa os arquivos de configuração do diretório `/usr/local/nginx`, porém não temos permisão para escrever neste diretório, então basicamente o paramentro `--prefix=/tmp/local/nginx` diz ao Nginx que todos seus arquivos de configuração estarão no diretório `/tmp/local/nginx`. Os outros dois paramentros `--error-log-path=stderr` e `--http-log-path=/dev/stdout` dizem para o Nginx imprimir os logs no console/shell.

Após isso você pode digitar `exit` no shell do ubuntu para finalizar.

### Configurando o Replit para executar o binário do NGINX

Agora que já temos o binário do Nginx, vamos configurar o Replit para executá-lo.

Crie um novo REPL com a liguagem `Bash`, em seguida faça o upload do arquivo binário do Nginx, que é o arquivo `nginx` em `/host/nginx-1.18.0/objs` em seguida faça o upload dos diretorios `conf` e `html` em `/host/nginx-1.18.0`.

Após o arquivo `nginx` e os diretorios `conf` e `html` aparecerem no REPL vamos criar um script de inicialização do nginx, o `main.sh`:

```sh
# main.sh

# prefix configurando no build do Nginx
NGINX_PREFIX_PATH="/tmp/local/nginx"

# criando diretorios caso nao existam
mkdir -p $NGINX_PREFIX_PATH
mkdir -p $NGINX_PREFIX_PATH/sbin/nginx
mkdir -p $NGINX_PREFIX_PATH/modules
mkdir -p $NGINX_PREFIX_PATH/conf
mkdir -p $NGINX_PREFIX_PATH/logs

# Copiando arquivos do usuario para a pasta do Nginx
rm -rf $NGINX_PREFIX_PATH/conf $NGINX_PREFIX_PATH/html
cp -rp conf html $NGINX_PREFIX_PATH

# Executando o servidor Nginx
chmod +x nginx
./nginx
```

Agora vamos clicar no botão `RUN >` e subir nosso servidor. Opsss... recebemos um erro do REPL, dizendo:

```txt
[emerg] 84#0: bind() to 0.0.0.0:80 failed (13: Permission denied)
```

Por padrão no linux, as portas inferiores que 1024 portas são restritas apenas ao usuário root, o que aconteceu foi que como nosso usuário não tem privilégios `root` então ele não pode iniciar um servidor que escute a porta 80, para corrigir isso vamos no REPL alterar o arquivo `conf/nginx.conf` e na linha `listen 80;` trocar para a porta  8080 que temos acesso, então ficará assim: `listen 8080;`. Vamos aproveitar também que estamos alterando o arquivo de configuração e adicionar no inicio do arquivo a linha `daemon off;` para evitar que o Nginx execute em Background e o Replit não enxergue que ele está em execução.


## Liceça

O conteúdo deste projeto em si está licenciado sob a  [Creative Commons Attribution 3.0 Unported license](https://creativecommons.org/licenses/by/3.0/), e os códigos-fonte inclusos no projeto estão licenciado sob a [MIT license](LICENSE).
