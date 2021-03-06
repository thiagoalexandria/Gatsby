---
image: /assets/img/Apache.png
title: Configuração de um servidor Apache
description: Nesse tutorial será explicado como configurar o Apache em um
  servidor com CentOS 7, desde a sua instalação até a configuração de vhosts e
  instalação de módulos. Nesse artigo será utilizado a versão 2.4 do apache em
  uma instalação Minimal do CentOs 7.
date: 2021-03-15
category: linux
background: "#EE0000"
tags:
  - linux
---

Esse artigo será terá a sua leitura mais demorada, ele será mais detalhado voltado ao entendimento da configuração de um servidor Apache e suas principais funcionalidades, com as dicas repassadas poderá ser feito facilmente a configuração de um servidor web para hospedagem de sites e sistemas.


## 1. Configuração do hostname

Antes de seguir com a configuração do Apache, iremos configurar o hostname do servidor da seguinte forma "meu-site.com.br" no qual você deve substituir pelo teu domínio do teu servidor, para isso será utilizado o seguinte comando:

```bash
hostnamectl set-hostname meu-site.com.br
```

Após ajustar o hostname, verifique é o nome configurado é retornado após utilizar o comando abaixo:

```bash
hostnamectl status
```

Feito isso, edite o arquivo hosts em seu /etc/hosts com o editor de sua preferência

```bash
vi /etc/hosts
```

## 2. Instalação e configuração do Apache

Configurado o hostname seguiremos com a instalação e configuração do serviço web no servidor, para isso basta seguir com a instalação do serviço httpd e alguns utilitários através do gerenciador de pacotes:

```bash
yum install -y httpd wget htop
```

Feito a instalação utilizando o gerenciador de pacotes inicie o serviço para verificar o seu funcionamento

```bash
systemctl start httpd
systemctl status httpd
```

### 2.1 Ajustes de configuração do sistema operacional

Para que o servidor funcione sem maiores problemas é necessário seguir alguns passos para liberação de porta no firewall e desativar o selinux para não realizar bloqueios nas aplicações, dessa forma escolha a forma com que se sente mais familiarizado :

A: Desabilite o firewalld e utilize o iptables como firewall

```bash  
systemctl stop firewalld
systemctl disable firewalld
```

Nesse teste iremos utilizar apenas ipv4 então desativaremos as configurações de ipv6 do iptables e das configurações de rede:

```bash
service ip6tables stop
chkconfig ip6tables off
sed -i 's/IPV6INIT\=\"yes\"/IPV6INIT\=\"no\"/g' /etc/sysconfig/network-scripts/ifcfg-eth0
echo -e 'net.ipv6.conf.all.disable_ipv6 = 1\n' >> /etc/sysctl.conf
echo -e 'NETWORKING_IPV6=no\n' >> /etc/sysconfig/network
```

  Após feito, liberar a porta 80 e 443 no iptables:

```bash
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
iptables -I INPUT -p tcp --dport 443 -j ACCEPT
 /sbin/service iptables save 
```

B: Liberar a porta 80 e 443 no firewalld

```bash
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --reload
```

Configurado  o firewall será feito o ajuste dos limites de segurança e desativação do selinux no servidor, para isso siga com o passo determinado abaixo:

```bash
echo -e '* soft    nofile  1024\n' >> /etc/security/limits.conf
echo -e '* hard    nofile  65535\n' >> /etc/security/limits.conf
echo -e 'sesssion required /lib64/security/pam_limits.so\n' >> /etc/pam.d/login
echo -e 'fs.file-max = 65535\n' >> /etc/sysctl.conf
sed -i "s/SELINUX=.*/SELINUX=disabled/" /etc/sysconfig/selinux
```

### 2.2 Estrutura do Apache

O Apache, assim como todo outro serviço possui caminho para os seus arquivos de configuração, para acessar esses arquivos basta seguir com o acesso ao seguinte diretório:

```bash
/etc/httpd
```

Nesse diretório é possível verificar alguns outros diretórios responsáveis pela estrutura do apache, são esses os conf, conf.d, logs, modules e run, abaixo é possível verificar as configurações armazenadas:

```bash
# Arquivo de configuração principal, responsável por iniciar e determinar as configurações iniciais do serviço Web
/etc/httpd/conf
# Diretório para realização de includes de configuração
/etc/httpd/conf.d
# Diretório padrão para armazenamento de logs
logs -> ../../var/log/httpd
# Link simbólico para as bibliotecas de módulos instalados
modules -> ../../usr/lib64/apache2/modules
# Informações sobre o processo ativo
run -> ../../var/run/apache2
```

## 3. Configuração de aplicação Web

### 3.1 Instalar e configurar o modulo php

Nesse passo não se tem restrições, caso deseje instalar a versão disponibilizada pelo yum basta seguir com a instalação, para verificar qual a versão disponível basta verificar utilizando a opção info do gerenciador de pacotes:

```bash
yum info php
```

Se estiver de acordo, basta seguir com a instalação:

```bash
yum install php
```

Como a versão disponibilizada já encontra-se em end-of-life será instalado a versão 7.0 do php, para isso basta seguir os seguintes passos, ativando os repositórios Epel e Remi:

```bash
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

Em seguida é necessário instalar o yum-utils, para poder manusear os repositórios e pacotes que serão instalados:

```bash
yum install yum-utils
```

Instalado o yum-utils,  basta escolher a versão do PHP que deseja, para isso selecione o pacote:

````bash
yum-config-manager --enable remo-php70   [Install PHP 7.0]
yum-config-manager --enable remi-php71   [Install PHP 7.1]
yum-config-manager --enable remi-php72   [Install PHP 7.2]
yum-config-manager --enable remi-php73   [Install PHP 7.3]
```

Configurado a versão siga com a instalação dos módulos:

```bash
yum install php php-mcrypt php-cli php-gd php-curl php-mysql php-ldap php-zip php-fileinfo
````

Certifique-se de que a versão foi instalada corretamente:

```bash
php -v
```

Criaremos o arquivo Virtual Host o nosso domínio diretamente no /etc/httpd/conf.d/www.apachevm.conf

```bash
<VirtualHost *:80>
    ServerAdmin webmaster@apachevm.com
    DocumentRoot /var/www/apachevm/html/videopage
    ServerName www.apachevm.com
    ErrorLog logs/www.apachevm.com-error_log
    CustomLog logs/www.apachevm-access_log common
</VirtualHost>
```

## 4. Configuração de Logs

### 4.1 Funcionamento dos logs

É possível personalizar como as informações são armazenadas, os seus níveis de criticidade e o formato dos logs. 

* Log level

  * Define o nível de criticidade do log do Apache
  * Configurado no httpd.conf
  * Default: LogLevel warn
* LogFormat

  * Cria aliases para formatação de logs
  * Evita poluir as configurações de VirtualHosts
  * Possibilidade de criar diversos logs com formatos diferentes
* Custom log

  * *Configurável no Virtual Host*
  * É possível criar diversos arquivos de acordo com o LogFormat
* Rotate logs

  * *Permite gravar um arquivo através de outro comando*
  * rotatelogs: comando para gerenciar rotação de logs
  * Rotacionar por tamanho ou por tempo

### 4.2 Criticidade de logs

Existem diversas opções para configuração do LogLevel do apache, segue abaixo uma tabela com as opções e as suas descrições:

![Tabela](/assets/img/tabela.png)

Para configuração do LogLevel basa seguir com a edição direta do arquivo de configuração do apache na variável LogLevel:

```bash
vim /etc/httpd/conf/httpd.conf
```

Feito o ajuste, basta reiniciar o serviço web

```bash
service httpd restart
```

### 4.3 LogFormat

O log format pode ser ajustado de diversas formas, adaptando-se a sua necessidade sobre os itens que deve ser logados, abaixo segue alguns exemplos:

```bash
LogFormat "%h %I %u %t \"%r\" %>s %b" common
LogFormat "%{Referer}i -> %U" referer
LogFormat "%{User-Agent}i" agent
```

Para ajustar ou adicionar informações de LogFormat, basta seguir com a edição do arquivo de configuração do apache e buscar por LogFormat

```bash
vim /etc/httpd/conf/httpd.conf
```

Nesse caso, foi feito a criação de um LogFormat customizado

```bash
LogFormat "%h %t \"%r\" %>s %b \"%{User-Agent}i\" **%T/%D** newformat
# %h host
# %t time
# %r request
# %S status
# %b bytes
# Useagent browser
# %t tempo em segundos
# %D tempo em décimos de segundos
```

Salve o arquivo de configuração e seguiremos com o ajuste no Virtual Host que desejamos que o utilize

```
vim /etc/httpd/conf.d/www.apachevm.conf
CustomLog logs/www.apachevm-access_log newformat
```

Salvo o arquivo, basta reiniciar o serviço web.

```bash
service httpd restart
```

## 5. Boas práticas de segurança

Existem algumas ações básicas que podem ser adotadas para manter o seu ambiente mais seguro, os pontos abaixo são as medidas primordiais e quase sempre ignoradas, indico que siga com a aplicação das soluções que vamos abordar para evitar ataques por misconfiguration.

### 5.1 Desligar modo TRACE

Trace é um método que em conjunto com o XSS, dessa forma ele pode ter acesso a informações sensíveis do seu cliente, uma delas é o acesso aos cookies e dados de autenticação então é muito importante que o Trace esteja desativado. 

Para verificar se o trace esta habilitado, basta executar o comando abaixo:

```bash
$ curl -X TRACE 127.0.0.1
TRACE / HTTP/1.1
User-Agent: curl/7.24.0 (x86_64-apple-darwin12.0) libcurl/7.24.0 OpenSSL/0.9.8r zlib/1.2.5
Host: 127.0.0.1
Accept: */*
```

Observe que caso solicitado, os cookies são retornados através dessa solicitação:

```bash
$ curl -X TRACE -H "Cookie: name=value" 127.0.0.1
TRACE / HTTP/1.1
User-Agent: curl/7.24.0 (x86_64-apple-darwin12.0) libcurl/7.24.0 OpenSSL/0.9.8r zlib/1.2.5
Host: 127.0.0.1
Accept: */*
Cookie: name=value
```

Para desabilitar, basta ajustar a diretiva TraceEnable para "off" diretamente no arquivo de configuração, após feito basta reiniciar o Apache

```bash
TraceEnable off
```

Para verificar se os ajustes foram aplicados, basta executar o comando abaixo e terá o seguinte retorno

```bash
$ curl -X TRACE 127.0.0.1
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>405 Method Not Allowed</title>
</head><body>
<h1>Method Not Allowed</h1>
<p>The requested method TRACE is not allowed for the URL /.</p>
</body></html>
```

### 5.2 Ocultar informações do Apache

Ocultar a versão do apache do seu faz com que o seu servidor não retorne a versão instalada, com isso evitaremos alguns ataques direcionados a vulnerabilidades documentadas para a versão, pois uma vez que é sabido a versão utilizada torna-se mais fácil o direcionamento dos ataques. Ocultar essa informação. Para desabilitar isso, basta seguir com edição do arquivo de configuração do httpd e editar as seguinte variáveis dessa forma:

```bash
vim /etc/httpd/conf/httpd.conf
ServerTokens Prod
ServerSignature Off
```

Após feito, basta reiniciar o serviço do apache, verá que a versão do apache não será mais retornado.

### 5.3 Negar acesso a diretórios

Uma prática muito comum é a restrição de acesso a determinado tipos de conteudos ou diretórios, geralmente conteúdo sensível ou administrativo. Nesse tipo de configuração não adotaremos .htaccess, para melhor gerenciamento os ajustes serão aplicados no Virtual Host, uma vez que possuimos acesso direto ao arquivo de configuração.

Para bloqueio de diretórios, é adotado a seguinte estrutura, com essa configuração todo o acesso solicitado para o ambiente /admin será bloqueado, porém o acesso pode ser liberado a determinados endereços adicionando a linha ***Allow from IP*** abaixo do Deny from all

```bash
<Directory "var/www/apachevm/admin">
	Order allow, deny
	Deny from all
</Directory>
```

Existem diversos tipos de possibilidades para bloqueio e restrição de acesso, vou deixar um link direto para a documentação do [apache](https://httpd.apache.org/docs/2.4/mod/mod_access_compat.html#order) para mais detalhes.

### 5.6 Como proteger versão PHP

Seguindo a mesma linha de pensamento para ocultarmos a versão do apache, é interessante que realizemos o mesmo procedimento para o PHP, mesmo utilizando a versão mais recente e atualizada disponível.

Essas informações de versão PHP podem ser obtidas através do Header response através da diretiva ***x-powered-by***. Para realizar o ajuste, basta seguir com a seguinte configuração no arquivo ***/etc/php.ini*** e ajustaremos a seguinte diretiva:

```bash
expose_php = Off
```

Salvo as configurações, basta reiniciar o serviço web.

## 6. Configuração de SSL

Utilizar um certificado SSL para garantir a segurança de um domínio é impressionável uma vez que os navegadores já acusam como não seguro domínios sem esse recurso, a maioria dos seus visitantes ao visualizar o cadeado de segurança, terão a certeza de estar em um ambiente confiável e que poderão acessar seu sistema ou realizar suas compras e navegar tranquilamente.

### 6.1. Instalação openssl e mod_ssl

Para utilizar desse recurso, é necessário que possua o mod_ssl instalado, no nosso caso, como iremos utilizar um certificado auto assinado, apenas para exemplificação, instalaremos também o openssl.

```bash
yum install mod_ssl openssl
```

### 6.2. Geração de certificado SSL self signed

Para gerar os arquivos para o certificado SSL usaremos o openssl, na geração do arquivo .key o comando abaixo foi utilizado:

```bash
openssl genrsa -out ca.key 2048
``` 

O próximo passo é realizar a geração do CSR,para isso utilizaremos a nossa key para gerar a CSR:

```bash
openssl req -new -key ca.key -out ca.csr
```

Será solicitado algumas informações, basta preencher os dados conforme forem solicitados ao seu certificado. Com o csr e a key, geraremos o certificado auto assinado:

```bash
openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt 
```

Assim já temos o certificado gerado para realização dos testes.

### 6.3. Configuração de vHost e Teste

Com o certificado gerado, basta seguirmos com os ajustes no virtual host para adicionarmos as configurações para a porta 443 segura, viabilizando o acesso https. Acesse o vHost do seu domínio e siga com a adição da seguinte configuração.

```bash
<VirtualHost *:443>
	ErrorLog logs/www.apachevm.com_ssl-error_log
    CustomLog logs/www.apachevm-ssl_access_log common
	LogLevel warn
	SSLEngine on
	SSLCertificateFile /path/para/certificado.crt
	SSLCertificateKeyFile /path/para/privatekey.key
	SSLProtocol all -SSLv2
		SSLCipherSuite DEFAULT:!EXP:!SSLv2:!DES:!IDEA:!SEED:+3DES
	DocumentRoot /var/www/apachevm/
	ServerName www.apachevm.com
</VirtualHost>
```

## 7. Testes de carga

Testes de carga é sempre uma ótima escolha para saber se o servidor irá suportara a aplicação antes de hospeda-la em produção, o AB ( Apache Bench) é uma funcionalidade para realização de teste de carga que já vem durante a instalação do Apache que é bastante intuitiva no qual pode ser feito um teste de quantidade e usuários e máximo de requests a serem realizados. O ideal é que a carga seja executada em um servidor diferente do que será testado, nesse exemplo, o teste AB será executado no mesmo servidor apenas para titulo de  conhecimento. Segue um exemplo de teste no qual será executado 1000 requests e 50 usuários simultâneos:

```
ab -n 1000 -c 50 https://www.apachevm.com.br
```

É indicado que realize algumas verificações no servidor que esta sendo testado durante as requisições como verificar load, utilização de CPU e Memória. 

## 8. Módulos

Os módulos permitem que o Apache execute funções adicionais, dessa forma existem alguns módulos que podem ser instalados no seu servidor para auxiliar em alguns processo de administração e personalização.

### 8.1. mod_rewrite

O módulo mod_rewrite usa um mecanismo de reescrita baseado em regras, baseado em um analisador de expressão regular PCRE, para reescrever URLs solicitados na hora, sendo muito utilizado para a criação de url amigáveis e também redirecionamentos.

Antes de tudo, basta verificar se o modulo mod_rewrite encontra-se instalado e carregado no arquivo de configuração do Apache.

```bash
grep mod_rewrite /etc/httpd/conf/httpd.conf
LoadModule rewrite_module modules/mod_rewrite.so
```

Com o módulo devidamente configurado basta seguir com a implementação das regras de rewrite, nesse caso, iremos ajustar apenas o redirecionamento para https para o nosso vHost

```bash
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI} [R,L]
```

Após salvo, basta reiniciar o serviço httpd.

### 8.2. mod_sustitute

A diretiva mod_sustitute especifica um padrão de pesquisa e em seguida de substituição, sendo retornado o valor ajustado para o HTTP Response. Podemos utilizar esse modulo para evitarmos os erros de Mixed Content, no qual consiste em retornos http em uma sessão https, iremos ajustar todos as requests feitas em http forçando a substituição para https.

Antes de tudo, basta verificar se o modulo mod_sustitute encontra-se instalado e carregado no arquivo de configuração do Apache.

```bash
grep mod_sustitute /etc/httpd/conf/httpd.conf
LoadModule substitute_module modules/mod_sustitute.so
```

Seguiremos com a aplicação do seguinte trecho no vHost do nosso domínio:

```bash
AddOutputFilterByType SUBSTITUTE text/html
Substitute "s|http:|https:|"
```

Após salvo, basta reiniciar o serviço httpd.
