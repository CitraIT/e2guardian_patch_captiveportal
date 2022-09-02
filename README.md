# E2Guardian - Integração de Autenticação com Captive Portal
Patch para permitir o Proxy E2Guardian identificar o usuário logado através do Captive Portal no pfSense


!! Importante: Este procedimento foi homologado para pfSense CE na versão 2.6  
Última atualização: 01/09/2022  
Responsável: Luciano Rodrigues - luciano@citrait.com.br

## Instalação:


### Etapas:  
1- Ajustar hostname e nome de domínio do firewall de acordo com o seu ambiente:  
1.1- A configuração é feita pelo menu System -> General Setup.  

2- Criar o arquivo de configuração externa do serviço DNS:  
2.1- Criar o arquivo /var/unbound/dnsauth.conf vazio com o comando abaixo (executar através do menu Diagnostics -> Command Prompt):  
```
echo> /var/unbound/dnsauth.conf
```
3- Configure o serviço de DNS do pfSense para incluir o arquivo dnsauth.conf.  
3.1- Acesse o menu Services -> DNS Resolver. Clique em Display Custom Options ao final da página e inclua a linha seguinte no campo de texto:  
```
include: /var/unbound/dnsauth.conf
```
4- Realize a instalação do E2Guardian:  
4.1- Vá no menu System -> Package Manager -> Available Packages e instale o pacote System_Patches.  
4.2- Acesse a página abaixo e copie todo o código:  
```
https://raw.githubusercontent.com/marcelloc/Unofficial-pfSense-packages/master/25_unofficial_packages_list.patch
```  
4.3- Acesse o menu System -> Patches. Clique em Add New Patch.  
4.4- Cole o código no campo Patch Contents.  
4.5- Em Description preencha com: Unnoficial_packages.  
4.6- No campo Path Strip Count defina com valor 1 e salve o patch (botão save ao final da página).  
4.7- Clique em apply no patch recem registrado.  
4.8- No menu Diagnostics -> Command Prompt execute o comando abaixo:  
```
fetch -q -o /usr/local/etc/pkg/repos/Unofficial.conf 
https://raw.githubusercontent.com/marcelloc/Unofficial-pfSense-packages/master/Unofficial_25.conf
```
4.9- Ainda no menu Diagnostics -> Command Prompt execute o comando abaixo:
```
pkg update
```
4.10- Acesse o menu System -> Package Manager -> Available Packages e instale o pacote E2Guardian.

5- Realize a configuração do captive portal (adicione uma zona ao captive portal).  
5.1- Acesse Zones -> Captive Portal e clique em Add.  
5.2- Dê um nome e uma descrição para a zona e clique em Add & Continue.  
5.3- Clique em Enable, selecione a interface (LAN).  
5.4- No campo Authentication Server selecione Local database.  
5.5- Clique em Save ao final da página.  

6- Instale o patch do captive portal.  
6.1- Acesse a página abaixo e copie todo o código:  
```
https://raw.githubusercontent.com/CitraIT/e2guardian_patch_captiveportal/main/patches/captiveportal.patch
```
6.2- Acesse o menu System -> Patches e clique em Add Patch.  
6.3- Cole o código no campo Patch Contents.  
6.4- Em Description preencha com: CitraIT_Patch_CaptivePortal_Index_PHP.  
6.5- No campo Path Strip Count defina com valor 1 e salve o patch (botão save ao final da página).  
6.6- Clique em apply no patch recem registrado.  

7- Ajustar o plugin de autenticação do E2Guardian.  
7.1- Acesse o menu Diagnostics -> Edit File.  
7.2- Insira no caminho do arquivo o texto abaixo e clique em Load:  
```
/usr/local/etc/e2guardian/authplugins/dnsauth.conf
```
7.3- Apague todo o texto e cole o texto abaixo:  
```
# IP/DNS-based auth plugin
#
# Obtains user and group from domain entry maintained by separate authentication# program.

plugname = 'dnsauth'

# Base domain
basedomain = "citrait.local"

# Authentication URL
authurl = "https://192.168.1.1:8002/?redirurl"

# Prefix for auth URLs
prefix_auth = "https://192.168.1.1:8002/"

# Redirect to auth (i.e. log-in)
#  yes - redirects to authurl to login
#  no - drops through to next auth plugin
redirect_to_auth = "yes"
```
7.4- Substitua citrait.local pelo nome de domínio do firewall (ex.: empresa.corp).  
7.5- Substitua a authurl pelo endereço no qual o pfSense te redireciona para o Captive Portal.  
7.6- Em prefix_auth ajuste conforme a variável authurl, mas terminando na barra após a porta.  
7.7- Clique em Save para salvar o arquivo.  

8- Crie um usuário de teste.  
8.1- Acesse o menu System -> User Manager e clique em Add.  
8.2- Preencha o nome e senha do usuário e clique em Save.  
8.3- Edite o usuário criado acima e atribua a ele a permissão "User - Services: Captive Portal login".  

9- Criar uma CA (Autoridade Certificadora) para usar na interceptação SSL/HTTPS do E2Guardian.   
9.1- Acesse o menu System -> Cert. Manager, clique em Add.
9.2- Dê um nome descritivo para a CA (ex.: CA-E2GUARDIAN).  
9.3- Marque a caixa "Trust Store".  
9.4- Em "Common Name" preencha com um nome significativo (ex.: usar o mesmo da descrição).  
9.5- Preencha as informações organizacionais (country code, state, city...) e clique em Save.

10- Configurar o E2Guardian.  
10.1- Acesse o menu Services -> E2Guardian Proxy.  
10.2- Marque a caixa "Enable e2guardian".  
10.3- Selecione as interfaces LAN e Loopback.  
10.4- Marque a caixa "Transparent HTTP Proxy".  
10.5- Marque a opção "Bypass Proxy for Private Address Destination".  
10.6- Marque a opção "Enable SSL support".  
10.7- Selecione a CA que criou na etapa acima.  
10.8- Clique em Save.  

11- Habilitar a autenticação no E2Guardian.  
11.1- Acesso o menu Services -> E2Guardian Proxy -> Guia General.  
11.2- No campo "Auth Plugins" selecione apenas DNS.  
11.3- Clique em Save ao final da Página.  

12- Configurar um usuário e grupo de teste.  
12.1- Acesso o menu Services -> E2Guardian Proxy -> Guia Grups e clique em Add.  
12.2- Dê um nome e uma descrição para o grupo (ex.: ti / ti).  
12.3- Clique em Save ao final da página.  
12.4- Acesse a aba Users.  
12.5- No campo com o nome do grupo (ex.: ti) referente ao grupo ti, insira o login do usuário (ex.: luciano).  
12.6- Clique em Save ao final da página.  
12.7- Clique em Apply Changes (botão verde que aparece no topo após salvar a configuração).  

13- Testar  
13.1- Usando uma máquina que está dentro da zona do captive portal habilitado (LAN), navegue em algum site.  
13.2- Você deve ser redirecionado para a tela do captive portal.  
13.3- Após autenticar, deverá ser redirecionado para o site que tentou navegar.  
13.4- Acesse o menu Services -> E2Guardian Proxy.  
13.5- Clique na guia Real Time.  
13.6- Deverá aparecer nos logs o nome do usuário e o grupo que ele foi identificado semelhante a imagem abaixo:  

![image](https://user-images.githubusercontent.com/91758384/188039740-0e3cbd25-b9ae-4c37-8636-5a2e051f5ad5.png)




