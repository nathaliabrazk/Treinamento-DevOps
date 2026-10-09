# Laboratório SEI/SIP 4.0 (4.0.12.15) no Amazon Linux 2

Guia de instalação do SEI e do SIP em uma instância EC2 (Amazon Linux 2), baseado no manual **INSTALACAO 4.0.12.15** e nos problemas encontrados durante o laboratório.

> **Convenções**
> - `SEU_ENDERECO`: DNS público ou IP da instância EC2.
> - `SENHA_DO_BANCO`: senha do usuário `sei_user` (nunca versionar senhas reais).
> - Prints das etapas ficam na pasta `imagens/`.
> - Blocos marcados com **Problema** descrevem erros reais e a correção aplicada.

---

## Índice

1. [Requisitos](#requisitos)
2. [Etapa 0: Preparação do ambiente](#etapa-0--preparação-do-ambiente)
3. [Etapa 1: Descompactação dos pacotes](#etapa-1--descompactação-dos-pacotes)
4. [Etapa 2: Apache](#etapa-2--apache)
5. [Etapa 3: PHP 7.3 e extensões](#etapa-3--php-73-e-extensões)
6. [Etapa 4: Memcached](#etapa-4--memcached)
7. [Etapa 5: Banco de dados (MariaDB)](#etapa-5--banco-de-dados-mariadb)
8. [Etapa 6: Permissões, SELinux e publicação no Apache](#etapa-6--permissões-selinux-e-publicação-no-apache)
9. [Etapa 7: Configurar o SIP](#etapa-7--configurar-o-sip)
10. [Etapa 8: Configurar o SEI](#etapa-8--configurar-o-sei)
11. [Etapa 9: Primeiro login e correção de erros](#etapa-9--primeiro-login-e-correção-de-erros)
12. [Etapa 10: Solr](#etapa-10--solr)
13. [Etapa 11: wkhtmltopdf](#etapa-11--wkhtmltopdf)
14. [Etapa 12: Agendamentos (cron)](#etapa-12--agendamentos-cron)
15. [Conferência final](#conferência-final)
16. [Problemas conhecidos (resumo)](#problemas-conhecidos-resumo)
17. [Pendências](#pendências)

---

## Requisitos

| Componente | Versão / observação |
|---|---|
| Apache | 2.4.6 |
| PHP | 7.3.x |
| Framework | InfraPHP |
| Banco (MySQL/MariaDB) | MySQLi 5.0.12 ou superior |
| Memcache (extensão PHP) | 4.0.4 |
| Memcached (serviço) | 1.4.25 |
| Java Runtime | 1.8 |
| wkhtmltopdf | 0.12.6 |
| Solr | 8.2.0 |
| Fontes TrueType | instaladas no servidor |
| ffmpeg | comando disponível |
| Uploadprogress | 1.1.3 (versão para PHP 7.3 ou compilar de `/opt/sei/config/uploadprogress`) |

**Extensões PHP exigidas:** bcmath, bz2, calendar, ctype, curl, date, dom, exif, fileinfo, filter, gd, gettext, hash, iconv, intl, json, libxml, mbstring, openssl, pcre, Phar, SimpleXML, soap, sodium, zip, zlib, uploadprogress e memcache.

**Ambiente do laboratório:** EC2 Amazon Linux 2, usuário `ec2-user`, SEI e Solr na mesma máquina.

---

## Etapa 0: Preparação do ambiente

### Acesso à máquina e leitura da documentação

A leitura da documentação foi feita pelo VS Code, com acesso à EC2 pelo MobaXterm:

1. Session → SSH.
2. Remote host: IP público da EC2. Username: `ec2-user`.
3. Advanced SSH settings → *Use private key* → selecionar a chave `lab.pem`.

O arquivo principal é `INSTALACAO 4.0.12.15.md`, baixado e lido no VS Code.

### Conhecer a máquina

```bash
cat /etc/os-release
free -h        # memória RAM total e livre
df -h /        # espaço livre no disco raiz
nproc          # número de CPUs
```

### Atualizar o sistema e instalar utilitários

```bash
sudo yum update -y
sudo yum install -y unzip tar vim curl wget tree nmap-ncat policycoreutils-python-utils
```

| Pacote | Uso |
|---|---|
| `unzip`, `tar` | descompactar arquivos |
| `vim` | editor |
| `curl`, `wget` | testar URLs e baixar arquivos |
| `tree` | visualizar a árvore de pastas |
| `nmap-ncat` | comando `nc` para testar portas |
| `policycoreutils-python-utils` | comando `semanage` (SELinux) |

### Verificar SELinux e firewall

```bash
getenforce                      # modo do SELinux
systemctl is-active firewalld   # firewall local
```

![Etapa 0](imagens/etapa-0.0.png)

---

## Etapa 1: Descompactação dos pacotes

### Listar o conteúdo do pacote (sem extrair)

```bash
unzip -l /opt/sei-lab/sei-4.0.12.15.zip | less
```

### Extrair o pacote principal

```bash
mkdir -p ~/sei-pacote
unzip -q /opt/sei-lab/sei-4.0.12.15.zip -d ~/sei-pacote
ls -R ~/sei-pacote | head -40
```

### Extrair o ZIP de fontes

O pacote principal traz um segundo ZIP (`sei-Fontes-4.0.12.15.zip`) com as pastas `sei`, `sip` e `infra`. É preciso extraí-lo também:

```bash
unzip -l ~/sei-pacote/sei-Fontes-4.0.12.15.zip | less
unzip -q ~/sei-pacote/sei-Fontes-4.0.12.15.zip -d ~/sei-pacote/
find ~/sei-pacote -maxdepth 3 -type d \( -name "sei" -o -name "sip" -o -name "infra" \)
```

### Copiar para `/opt`

```bash
sudo cp -r ~/sei-pacote/sei ~/sei-pacote/sip ~/sei-pacote/infra /opt/
ls /opt/sei /opt/sip /opt/infra
```

![Etapa 1](imagens/etapa-1.7.png)

---

## Etapa 2: Apache

Referência no manual: seção 2 (Servidores).

```bash
sudo yum install -y httpd
sudo systemctl enable --now httpd
sudo systemctl status httpd
curl -I http://localhost
```

**Resultado esperado:** o Apache responde. Um `403 Forbidden` neste ponto é normal, pois o `/var/www/html` está vazio. O que **não** pode aparecer é `Connection refused`.

```bash
ls -ld /var/www/html
ls -la /var/www/html
```

---

## Etapa 3: PHP 7.3 e extensões

Referência no manual: seção 2 (tabelas de nós de aplicação e configuração do PHP).

### Instalação

O PHP 7.3 não existe nos repositórios padrão atuais. A tentativa pelo repositório Remi falhou:

```bash
rpm -E %rhel
sudo yum install -y https://rpms.remirepo.net/enterprise/remi-release-7.rpm
# Erro: Requires: epel-release = 7
```

> **Problema:** o pacote Remi 7 depende do `epel-release`, que não existe no Amazon Linux 2.
> **Solução:** usar o `amazon-linux-extras`.

```bash
sudo amazon-linux-extras enable php7.3
sudo yum clean metadata
sudo yum install -y php php-cli php-pdo php-fpm php-json php-mysqlnd
php -v
```

### Conferir as extensões exigidas

```bash
for ext in bcmath bz2 calendar ctype curl date dom exif fileinfo filter gd gettext hash iconv igbinary intl json ldap libxml mbstring memcache mysqli openssl pcre Phar SimpleXML soap sodium zip zlib uploadprogress; do
  php -m | grep -qix "$ext" || echo "FALTA: $ext"
done; echo "verificação concluída"
```

Na primeira execução faltavam: bcmath, dom, gd, igbinary, intl, ldap, mbstring, memcache, SimpleXML, soap, sodium e uploadprogress.

### Instalar as extensões que faltam

```bash
sudo yum install -y php-bcmath php-gd php-mbstring php-soap php-intl php-zip php-xml
sudo yum install -y php-devel php-pear gcc make zlib-devel
```

#### Extensão `memcache`

O pacote `php-pecl-memcache` não estava disponível para o PHP 7.3. A extensão foi compilada pelo PECL:

```bash
printf "yes\n" | sudo pecl install memcache-4.0.5.2
echo "extension=memcache.so" | sudo tee /etc/php.d/50-memcache.ini
sudo systemctl restart httpd
php -m | grep -i memcache
```

### Configuração do `php.ini`

Em vez de editar o `php.ini` principal, os ajustes ficam em um arquivo próprio em `/etc/php.d/` (atualizações de pacote não o apagam):

```bash
sudo tee /etc/php.d/99-sei.ini > /dev/null << 'EOF'
; Ajustes do SEI/SIP - manual seção 2, "Configuração do PHP"
include_path = ".:/usr/share/pear:/usr/share/php:/opt/infra/infra_php"
default_charset = "ISO-8859-1"
session.gc_maxlifetime = 28800
short_open_tag = On
default_socket_timeout = 60
max_input_vars = 1000
html_errors = 0
post_max_size = 201M
upload_max_filesize = 200M
session.cookie_secure = 0
date.timezone = America/Sao_Paulo
EOF

cat /etc/php.d/99-sei.ini
```

| Parâmetro | Motivo |
|---|---|
| `include_path` | acrescenta o diretório do InfraPHP ao final |
| `short_open_tag = On` | sem isso o navegador exibe código PHP em vez da página (Problema nº 1 do manual) |
| `session.cookie_secure = 0` | enquanto o lab estiver em HTTP. Com `1` sem HTTPS, o login só recarrega a tela (Problema nº 9) |
| `date.timezone` | não está no manual, mas evita avisos de fuso horário |

---

## Etapa 4: Memcached

Referência no manual: seção 2 (Memcached 1.4.25).

```bash
sudo yum install -y memcached
cat /etc/sysconfig/memcached          # MAXCONN e CACHESIZE (MB)
sudo systemctl enable --now memcached
systemctl status memcached
printf "stats\nquit\n" | nc 127.0.0.1 11211
```

**Resultado esperado:** o comando `nc` devolve as estatísticas do memcached.

---

## Etapa 5: Banco de dados (MariaDB)

Referência no manual: seção 4 (Bases de Dados) e Problemas nº 7, 10, 21 e 32.

### Instalação e configuração

```bash
sudo yum install -y mariadb-server

sudo tee /etc/my.cnf.d/sei.cnf > /dev/null << 'EOF'
[mysqld]
character-set-server = latin1
collation-server = latin1_swedish_ci
max_allowed_packet = 64M

[client]
default-character-set = latin1
EOF

sudo systemctl enable --now mariadb
systemctl status mariadb
sudo mysql_secure_installation
```

Verificar o charset (Problema nº 10 do manual):

```bash
mysql -u root -p -e "SHOW VARIABLES WHERE Variable_name IN ('character_set_client','character_set_server','character_set_database','character_set_connection');"
```

### Criar bases e usuário da aplicação

```sql
CREATE DATABASE sei CHARACTER SET latin1 COLLATE latin1_swedish_ci;
CREATE DATABASE sip CHARACTER SET latin1 COLLATE latin1_swedish_ci;
CREATE USER 'sei_user'@'localhost' IDENTIFIED BY 'SENHA_DO_BANCO';
GRANT SELECT, INSERT, UPDATE, DELETE ON sei.* TO 'sei_user'@'localhost';
GRANT SELECT, INSERT, UPDATE, DELETE ON sip.* TO 'sei_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

O manual exige um usuário **sem permissão de DDL** (CREATE, ALTER, DROP).

### Restaurar os dumps

```bash
find ~/sei-pacote -iname '*.sql*'

mysql -u root -p sei < ~/sei-pacote/CAMINHO/DUMP_DO_SEI.sql
mysql -u root -p sip < ~/sei-pacote/CAMINHO/DUMP_DO_SIP.sql

mysql -u root -p -e "SELECT table_schema, COUNT(*) AS tabelas FROM information_schema.tables WHERE table_schema IN ('sei','sip') GROUP BY table_schema;"
```

### Ajustar órgão e URLs no SIP

```sql
-- mysql -u root -p sip
UPDATE orgao
SET sigla='LAB', descricao='Laboratorio Trainee'
WHERE id_orgao=0;

UPDATE sistema
SET pagina_inicial='http://SEU_ENDERECO/sip'
WHERE sigla='SIP';

UPDATE sistema
SET pagina_inicial='http://SEU_ENDERECO/sei/inicializar.php',
    web_service='http://SEU_ENDERECO/sei/controlador_ws.php?servico=sip'
WHERE sigla='SEI';   -- no lab foi usada a sigla cadastrada para a instância

SELECT sigla, pagina_inicial, web_service FROM sistema;
EXIT;
```

### Mesmo ajuste de órgão na base `sei`

A sigla do órgão **deve** ser a mesma nas duas bases:

```bash
mysql -u root -p sei -e "UPDATE orgao SET sigla='LAB', descricao='Laboratorio Trainee' WHERE id_orgao=0; SELECT id_orgao, sigla, descricao FROM orgao;"
```

### Testar o usuário da aplicação (não o root)

```bash
mysql -u sei_user -p -e "SELECT COUNT(*) FROM sip.usuario; SELECT COUNT(*) FROM sei.orgao;"
```

> **Problema:** `Access denied` para o `sei_user`.
> **Solução:** redefinir a senha pelo console como root.

```sql
SET PASSWORD FOR 'sei_user'@'localhost' = PASSWORD('NOVA_SENHA');
FLUSH PRIVILEGES;
```

---

## Etapa 6: Permissões, SELinux e publicação no Apache

Referência no manual: seção 3 (permissões, cron do temp, `sei.conf`), seção 2 (SELinux e `robots.txt`) e seção 6 (`RepositorioArquivos`).

### Permissões dos diretórios

O manual escreve `root.apache` (sintaxe antiga). Use `root:apache`.

```bash
# SEI
sudo chown -R root:apache /opt/sei
sudo find /opt/sei -type d -exec chmod 2750 {} \;
sudo find /opt/sei -type f -exec chmod 0640 {} \;
sudo find /opt/sei/temp -type d -exec chmod 2570 {} \;

# SIP
sudo chown -R root:apache /opt/sip
sudo find /opt/sip -type d -exec chmod 2750 {} \;
sudo find /opt/sip -type f -exec chmod 0640 {} \;
sudo find /opt/sip/temp -type d -exec chmod 2570 {} \;

# InfraPHP (não tem pasta temp)
sudo chown -R root:apache /opt/infra
sudo find /opt/infra -type d -exec chmod 2750 {} \;
sudo find /opt/infra -type f -exec chmod 0640 {} \;

sudo ls -ld /opt/sei /opt/sei/temp /opt/sip/temp /opt/infra
```

| Modo | Significado |
|---|---|
| `2750` (diretórios) | setgid (arquivos novos herdam o grupo `apache`); dono tudo, grupo lê/entra, outros nada |
| `0640` (arquivos) | dono lê e grava; grupo `apache` só lê. O servidor executa o código, mas não altera |
| `2570` (temp) | o grupo tem permissão total: o Apache grava uploads, PDFs e ZIPs ali |

> **Problema:** durante o primeiro login o Apache não conseguia gravar no `temp`.
> **Solução aplicada:** `sudo chmod -R 2770 /opt/sei/temp /opt/sip/temp`.

### Repositório de arquivos

```bash
sudo mkdir -p /var/sei/arquivos
sudo chown -R apache:apache /var/sei/arquivos
sudo chmod -R 750 /var/sei/arquivos
```

Durante o laboratório também foram criados `/dados` e `/sei-nfs/dados`, com dono `apache:apache`, para testes de `RepositorioArquivos`. Confirme qual caminho está em `ConfiguracaoSEI.php`:

```bash
sudo grep -n RepositorioArquivos /opt/sei/config/ConfiguracaoSEI.php
```

> **Atenção:** `/dados` também é usado pelo Solr (Etapa 10), com dono `solr`. Não reutilize `/dados` como repositório de arquivos do SEI.

### SELinux

Booleans do manual:

```bash
sudo setsebool -P httpd_can_network_connect 1      # Apache/PHP abrem conexões (web service do SIP, Solr)
sudo setsebool -P httpd_can_network_memcache 1     # fala com o memcached
sudo setsebool -P httpd_execmem 1                  # executa código em memória
sudo setsebool -P httpd_can_network_connect_db 1   # conecta ao banco
```

Para o modo enforcing, o manual orienta editar `/etc/selinux/config` (`SELINUX=enforcing`). Se o SELinux ficar em **enforcing**, aplique também:

```bash
getsebool -a | grep -E 'httpd_can_network|httpd_execmem'    # os quatro devem estar "on"
sudo semanage fcontext -a -t httpd_sys_content_t "/opt/(sei|sip|infra)(/.*)?"
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/opt/(sei|sip)/temp(/.*)?"
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/sei(/.*)?"
sudo restorecon -Rv /opt/sei /opt/sip /opt/infra /var/sei
```

> **Estado no laboratório:** durante o diagnóstico do login o SELinux foi colocado em permissive (`sudo setenforce 0`). Os comandos `semanage`/`restorecon` não foram necessários enquanto ele ficou desativado. Registre o estado final com `getenforce`.

### Publicar no Apache (`sei.conf`)

```bash
sudo tee /etc/httpd/conf.d/sei.conf > /dev/null << 'EOF'
Alias "/sei" "/opt/sei/web"
Alias "/sip" "/opt/sip/web"
Alias "/infra_css" "/opt/infra/infra_css"
Alias "/infra_js" "/opt/infra/infra_js"

<Directory />
  AllowOverride None
  Require all denied
</Directory>

<Directory ~ "(/opt/sei/web|/opt/sip/web|/opt/infra/infra_css|/opt/infra/infra_js)" >
  AllowOverride None
  Options None
  Require all granted
</Directory>
EOF
```

Só a pasta `web` fica exposta; `config` e `scripts` não são acessíveis pelo navegador. O primeiro `<Directory />` nega tudo; o segundo libera apenas as quatro pastas públicas.

### `robots.txt` e reinício do Apache

```bash
printf "User-agent: *\nDisallow: /\n" | sudo tee /var/www/html/robots.txt
sudo apachectl configtest
sudo systemctl restart httpd
```

> **Problema:** o comando do manual veio com `systemctl restart http`. O nome correto do serviço é `httpd`.

### Limpeza do `temp` (cron)

O manual traz as linhas de limpeza em `/etc/cron.d/sei-limpeza` (seção 3). Copie as linhas do manual (formato com a coluna `root`, válido em `/etc/cron.d`, não em `crontab -e`) e ajuste dono e modo:

```bash
sudo chown root:root /etc/cron.d/sei-limpeza
sudo chmod 644 /etc/cron.d/sei-limpeza
cat /etc/cron.d/sei-limpeza
```

---

## Etapa 7: Configurar o SIP

Referência no manual: seção 5.

```bash
sudo ls -la /opt/sip/config/
sudo cp -p /opt/sip/config/ConfiguracaoSip.exemplo.php /opt/sip/config/ConfiguracaoSip.php
sudo cp -p /opt/sip/config/ConfiguracaoSip.php /opt/sip/config/ConfiguracaoSip.php.original   # backup do manual
sudo vim /opt/sip/config/ConfiguracaoSip.php
```
| Bloco / chave | Valor no lab | Por quê |
|---|---|---|
| `SIP / URL` | `http://SEU_ENDERECO/sip` | Endereço pelo qual o SIP é acessado. |
| `SIP / Producao` |  false durante a montagem, true no fim | Com false o sistema mostra o detalhe dos erros, o que ajuda a aprender. O manual exige true em produção. |
| PaginaSip / NomeSistema | `SIP` | Título das janelas |
| SessaoSip / SiglaOrgaoSistema | `LAB` | A mesma sigla dos updates da Etapa 5 |
| SessaoSip / SiglaSistema | `SIP` | Valor fixo do manual |
| SessaoSip / PaginaLogin | `http://SEU_ENDERECO/sip/login.php` | Tela de login |
| SessaoSip / SipWsdl | `http://SEU_ENDERECO/sip/controlador_ws.php?servico=sip` | Endereço do web service do SIP (Problema nº 3 se estiver errado) |
| SessaoSip / ChaveAcesso | Manter a do pacote | O manual manda usar a chave que veio no arquivo para a base padrão. |
| SessaoSip / https | False (muda na Etapa 12) | Ainda sem certificado |
| BancoSip / Servidor, Porta | `localhost, 3306` | Banco na mesma máquina |
| BancoSip / Banco | sip | |
| BancoSip / Usuario, Senha | sei_user, a senha criada | Usuário SEM DDL, como pede o manual|
| BancoSip / Tipo | MySql |  Também vale para MariaDB |
| CacheSip / Servidor, Porta | localhost, 11211 | O memcached da Etapa 4 |

Campos alterados:

```php
'BancoSip' => array(
    'Servidor' => 'localhost',
    'Porta'    => '3306',
    'Banco'    => 'sip',
    'Usuario'  => 'sei_user',
    'Senha'    => 'SENHA_DO_BANCO',
    'Tipo'     => 'MySql'),
'CacheSip' => array(
    'Servidor' => 'localhost',
    'Porta'    => '11211'),
```

Também é preciso ajustar a **URL** do SIP (`PaginaSip`/`URL`) para `http://SEU_ENDERECO/sip` e a sigla do órgão para `LAB`.

Validar:

```bash
sudo php -l /opt/sip/config/ConfiguracaoSip.php     # sem sudo dá "Could not open input file"
sudo ls -l /opt/sip/config/ConfiguracaoSip.php      # conferir dono/permissões da Etapa 6
```

---

## Etapa 8: Configurar o SEI

Referência no manual: seção 6.

```bash
sudo ls -la /opt/sei/config/
sudo cp -p /opt/sei/config/ConfiguracaoSEI.exemplo.php /opt/sei/config/ConfiguracaoSEI.php
sudo cp -p /opt/sei/config/ConfiguracaoSEI.php /opt/sei/config/ConfiguracaoSEI.php.original
sudo vim /opt/sei/config/ConfiguracaoSEI.php
```

| Bloco / chave | Valor no lab | Observação |
|---|---|---|
| `SEI` / `URL` | `http://SEU_ENDERECO/sei` | |
| `SEI` / `RepositorioArquivos` | `/var/sei/arquivos` | pasta criada na Etapa 6 |
| `SessaoSEI` / `SiglaOrgaoSistema` | `LAB` | mesma sigla nas duas bases |
| `SessaoSEI` / `SiglaSistema` | `SEI` | |
| `SessaoSEI` / `PaginaLogin` | `http://SEU_ENDERECO/sip/login.php` | o login do SEI é feito no SIP |
| `SessaoSEI` / `SipWsdl` | `http://SEU_ENDERECO/sip/controlador_ws.php?servico=sip` | por aqui o SEI conversa com o SIP |
| `SessaoSEI` / `ChaveAcesso` | manter a do pacote | Problema nº 6 se estiver errada |
| `SessaoSEI` / `https` | `false` | muda na etapa de HTTPS |
| `BancoSEI` / `Servidor`, `Porta`, `Banco` | `localhost`, `3306`, `sei` | |
| `BancoSEI` / `Usuario`, `Senha`, `Tipo` | `sei_user`, senha, `MySql` | o pacote vem com usuário `teste` |
| `CacheSEI` / `Servidor`, `Porta` | `localhost`, `11211` | o **mesmo** memcached do SIP |
| `Solr` | ver Etapa 10 | |

Para desativar HTTPS (linha 45 no arquivo do lab):

```bash
sudo sed -i "45s/'https' => true/'https' => false/" /opt/sei/config/ConfiguracaoSEI.php
sudo sed -n '45p' /opt/sei/config/ConfiguracaoSEI.php
```

Validar:

```bash
sudo php -l /opt/sei/config/ConfiguracaoSEI.php
sudo ls -l /opt/sei/config/ConfiguracaoSEI.php
```

---

## Etapa 9: Primeiro login e correção de erros

Referência no manual: seção 7 (usuário e senha `teste`).

```bash
curl -I http://SEU_ENDERECO/sip/
curl -I http://SEU_ENDERECO/sei
sudo ss -lntp | grep ':80'
```

Ferramentas de diagnóstico usadas:

```bash
sudo tail -n 50 /var/log/httpd/error_log
sudo tail -f /var/log/httpd/error_log | grep -v "not found or unable to stat"
sudo tail -n 30 /var/log/php-fpm/www-error.log
php -i | grep -E "^error_log|^log_errors|^display_errors"
mysql -u sei_user -p -e "SELECT dth_log, texto FROM sei.infra_log ORDER BY dth_log DESC LIMIT 3\G"
```

Habilitar exibição de erros temporariamente (**desfazer depois do diagnóstico**):

```bash
sudo sed -i 's/^display_errors = .*/display_errors = On/' /etc/php.ini
sudo systemctl restart httpd
```

### Registro dos erros (prints)

#### Erro 1: ao acessar o SIP pelo navegador

- **URL testada:** `http://ec2-3-85-233-158.compute-1.amazonaws.com/sip`
- **Mensagem exibida:** Erro Falha ao abrir conexão com o banco de dados
- **Causa:** Foi verificado o arquivo de configuração Sip estava com o usuário errado.
- - **Correção:** O acesso foi realizado e o usuário ‘localhost’ foi substituido por ‘sei_user’

![Erro 1: acesso ao SIP no navegador](imagens/ERRO-SIP-1.png)

#### Erro 2: erro de login ao acessar o SIP pelo navegador

- **URL testada:** `(http://ec2-3-85-233-158.compute-1.amazonaws.com/sip)`
- **Mensagem exibida ao realizar login:** Esta página não está funcionando no momento - ec2-3-85-233-158.compute-1.amazonaws.com não pode lidar com esta solicitação no momento.
- **Causa:** Foi verificado que era necessário instalar pendências do módulo SOAP para o PHP
- **Correção:** Instalar pendências e reiniciar o apache.
![Erro 2: acesso ao SEI no navegador](imagens/ERRO-SIP-2.png)

#### Erro 3: erro de login ao acessar o SIP pelo navegador

- **URL testada:** `(http://ec2-3-85-233-158.compute-1.amazonaws.com/sip)`
- **Mensagem exibida:** ec2-3-85-233-158.compute-1.amazonaws.com diz Class “ not found 
- **Causa:** ` Ausência da extensão memcache,  se CacheSEI/CacheSip estiverem ativos no arquivo, o login falha com “Erro acessando o Sistema de Permissões” caso o memcached não esteja rodando)
- **Correção:** .

![Erro 3: falha de login no SIP](imagens/ERRO-SIP-3.png)

#### Erro ao gerar PDF de processo SEI

- **Causa:** O servidor não tinha o wkhtmltopdf instalado. O SEI usa essa ferramenta para converter o HTML dos documentos em PDF, e sem o binário no servidor a geração falha.
- **Correção:** Instalar as dependências, a instalação foi feita em uma instância EC2 (Amazon Linux 2, usuário ec2-user), usando o pacote RPM oficial do projeto.

### Problemas encontrados e soluções

| # | Sintoma | Causa | Solução |
|---|---|---|---|
| 1 | SIP abre com URL errada | `URL` do `ConfiguracaoSip.php` sem alteração | Ajustar para `http://SEU_ENDERECO/sip` |
| 2 | Erro de acesso ao banco | `Usuario` do SIP estava `localhost` | Trocar para `sei_user` |
| 3 | Erro após abrir a tela de login | extensão `soap` ausente (erro no `error_log`) | `sudo yum install -y php-soap` e `sudo systemctl restart httpd` |
| 4 | `Class "" not found` no login, sem rastro no log | extensão `memcache` ausente e outras extensões faltando (bcmath, gd, mbstring, intl, zip) | Instalar as extensões (Etapa 3), compilar `memcache` pelo PECL e reiniciar o Apache |
| 5 | Login só recarrega a tela | `session.cookie_secure=1` em HTTP, ou `https => true` na configuração do SEI | `session.cookie_secure = 0` e `'https' => false` |
| 6 | `Access denied` ao conectar no banco | pacote vem com usuário `teste`; senha do `sei_user` divergente | Corrigir `Usuario`/`Senha` em `ConfiguracaoSEI.php` e `ConfiguracaoSip.php`; `SET PASSWORD` no MariaDB |
| 7 | Erros de gravação | Apache sem escrita em `temp` | `sudo chmod -R 2770 /opt/sei/temp /opt/sip/temp` |

Em um dos testes do laboratório, o `sei_user` recebeu `GRANT ALL PRIVILEGES` nas bases `sei` e `sip` para eliminar erros de permissão no banco. Se for o caso, avalie voltar para `SELECT, INSERT, UPDATE, DELETE` ao final, conforme exige o manual.

Reiniciar os serviços após as correções:

```bash
sudo systemctl restart httpd php-fpm
```

Teste no navegador: `http://SEU_ENDERECO/sip` → login com `teste` / `teste` (usuário padrão do manual; **troque a senha**).

---

## Etapa 10: Solr

Referência no manual: seção 21 (Solr, passos 1 a 10).

### 10.1 Pré-requisitos

O nome correto do pacote leva hífen: `java-1.8.0-...` (com ponto, o `yum` responde *No package available*).

```bash
sudo yum install -y java-1.8.0-openjdk-headless
java -version

sudo useradd solr      # no lab o usuário já existia ("already exists" não é erro)
```

### 10.2 Preparar os arquivos em `/tmp`

```bash
sudo ls /opt/sei/config/solr
# Esperado: log4j.properties  sei-cores-8.2.0  sei-solr-8.2.0.sh  solr.service

sudo cp -r /opt/sei/config/solr/. /tmp/
```

> **Problema:** `cp: cannot stat '/opt/sei/config/solr/*'`.
> **Causa:** o `*` é expandido pelo shell do `ec2-user`, que não tem permissão de leitura na pasta.
> **Solução:** usar `/.` no lugar de `/*` (ou `sudo sh -c 'cp -r ... /*'`).

O `solr-8.2.0.tgz` **não veio no pacote** (confirmado com `sudo find / -name 'solr-8.2.0*'`). Download direto em `/tmp`:

```bash
sudo wget -P /tmp https://archive.apache.org/dist/lucene/solr/8.2.0/solr-8.2.0.tgz
ls /tmp
```

> O download (173 MB) levou cerca de 10 minutos nesta máquina.

### 10.3 Corrigir CRLF e executar o script de instalação

> **Problema:** `$'\r': command not found` e `syntax error near unexpected token $'do\r'`.
> **Causa:** arquivos com quebra de linha do Windows (CRLF).
> **Solução:** converter antes de executar.

```bash
sudo sed -i 's/\r$//' /tmp/sei-solr-8.2.0.sh /tmp/solr.service /tmp/log4j.properties
sudo find /tmp/sei-cores-8.2.0 -type f \( -name '*.xml' -o -name '*.properties' -o -name '*.txt' \) -exec sed -i 's/\r$//' {} +

sudo less /tmp/sei-solr-8.2.0.sh      # ler o que o script faz antes de executar
sudo bash /tmp/sei-solr-8.2.0.sh
sudo tree -d /dados
```

O script descompacta o Solr em `/opt/solr`, cria `/dados` com os cores e instala o serviço. As mensagens `mkdir: cannot create directory '/dados': File exists` e os `->` de links simbólicos **não são erro**.

**Resultado esperado de `sudo tree -d /dados`:**

```
/dados
├── sei-bases-conhecimento
│   ├── conf
│   └── conteudo
├── sei-protocolos
│   ├── conf
│   └── conteudo
└── sei-publicacoes
    ├── conf
    └── conteudo
```

(Sem `sudo`, o `tree` mostra `[error opening dir]`, pois `/dados` pertence ao usuário `solr`.)

### 10.4 Corrigir o erro de memória (`-Xmxlg`)

> **Problema:** o serviço aparecia como `active (running)`, mas o Solr não subia (porta 8983 fechada, sem processo `java`).
> **Diagnóstico:** `sudo tail -30 /opt/solr/server/logs/solr-8983-console.log` mostrou `Invalid maximum heap size: -Xmxlg`.
> **Causa:** em `/opt/solr/bin/solr.in.sh` a opção veio com a letra **L** minúscula no lugar do número **1**.

```bash
sudo grep -n "^SOLR_JAVA_MEM" /opt/solr/bin/solr.in.sh
sudo sed -i 's/-Xmxlg/-Xmx1g/' /opt/solr/bin/solr.in.sh
sudo grep -n "^SOLR_JAVA_MEM" /opt/solr/bin/solr.in.sh
# Esperado: SOLR_JAVA_MEM="-Xms512m -Xmx1g"
free -m      # a máquina do lab tem ~3,8 GB; com menos de 2 GB use -Xmx512m
```

Para ver o erro real de um Solr que não sobe, rode em primeiro plano:

```bash
sudo systemctl stop solr
sudo pkill -f /opt/solr/bin/solr
sudo -u solr /opt/solr/bin/solr start -f -p 8983
```

> O aviso *Available entropy is low* não é a causa (o valor era 256).

### 10.5 Subir o serviço e habilitar no boot

```bash
sudo systemctl daemon-reload
sudo systemctl restart solr
sleep 30
sudo systemctl enable solr
```

### 10.6 Verificar os cores

```bash
curl -sS "http://localhost:8983/solr/admin/cores?action=STATUS&wt=json" | grep -o '"name":"[^"]*"'
```

**Esperado:**

```
"name":"sei-bases-conhecimento"
"name":"sei-protocolos"
"name":"sei-publicacoes"
```

Saída vazia com o serviço ativo significa Solr fora do ar ou cores não criados. Use `curl -sS -i` (sem `-s` isolado) para ver `Connection refused`.

### 10.7 Restringir o acesso por IP (passo 9 do manual)

Lista branca com `IPAccessHandler`. No lab, SEI e Solr ficam na mesma máquina; apenas `127.0.0.1` é liberado.

```bash
# Backup
sudo cp -p /opt/solr/server/etc/jetty.xml /opt/solr/server/etc/jetty.xml.original

# Criar o bloco (executar NO TERMINAL, não dentro do vim)
cat > /tmp/ipaccess.xml <<'EOF'

    <!-- Restringe o acesso ao Solr por IP (lista branca) -->
    <Get id="oldhandler" name="handler"/>
    <Set name="handler">
      <New id="IPAccessHandler" class="org.eclipse.jetty.server.handler.IPAccessHandler">
        <Set name="handler"><Ref refid="oldhandler"/></Set>
        <Call name="addWhite"><Arg>127.0.0.1</Arg></Call>
        <Set name="whiteListByPath">false</Set>
      </New>
    </Set>

EOF

# Inserir antes de </Configure>
sudo sed -i '/<\/Configure>/e cat /tmp/ipaccess.xml' /opt/solr/server/etc/jetty.xml

# Validar, reiniciar e testar
sudo tail -30 /opt/solr/server/etc/jetty.xml
sudo xmllint --noout /opt/solr/server/etc/jetty.xml && echo "XML ok"
sudo systemctl restart solr
sleep 30
curl -sS "http://localhost:8983/solr/admin/cores?action=STATUS&wt=json" | grep -o '"name":"[^"]*"'
```

> **Problema:** ao colar o comando `cat > ... <<'EOF'` dentro do `vim`, o texto entrou no meio do XML e deixou o arquivo quebrado.
> **Solução:** sair com `:q!` (sem salvar), conferir com `sudo diff` contra o `.original` e refazer pelo terminal.

**Rollback**, se o Solr não subir:

```bash
sudo cp -p /opt/solr/server/etc/jetty.xml.original /opt/solr/server/etc/jetty.xml
sudo systemctl restart solr
```

Se o `ConfiguracaoSEI.php` apontar para o Solr por um endereço diferente de `localhost`/`127.0.0.1`, o IP de origem precisa estar na lista branca (`<Call name="addWhite"><Arg>IP</Arg></Call>`).

### 10.8 Indexação inicial

```bash
sudo /usr/bin/php -c /etc/php.ini /opt/sei/scripts/indexacao_protocolos_completa.php
sudo /usr/bin/php -c /etc/php.ini /opt/sei/scripts/indexacao_publicacoes.php
sudo /usr/bin/php -c /etc/php.ini /opt/sei/scripts/indexacao_bases_conhecimento.php
```

**Esperado:** cada script termina com `Operacao Finalizada.` (em base nova, `0` itens indexados).

---

## Etapa 11: wkhtmltopdf

Referência no manual: seção 2 (wkhtmltopdf 0.12.6).

```bash
# Dependências
sudo yum install -y fontconfig libXrender libXext xorg-x11-fonts-75dpi xorg-x11-fonts-Type1 libjpeg

# Download (os pacotes ficam no repositório "packaging")
cd /tmp
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6-1/wkhtmltox-0.12.6-1.centos7.x86_64.rpm
ls -lh /tmp/wkhtmltox*.rpm        # ~14 MB

# Instalação e conferência
sudo yum localinstall -y /tmp/wkhtmltox-0.12.6-1.centos7.x86_64.rpm
which wkhtmltopdf
wkhtmltopdf --version
```

**Esperado:** `/usr/local/bin/wkhtmltopdf` e `wkhtmltopdf 0.12.6 (with patched qt)`.

> **Problema:** `404 Not Found` ao baixar de `github.com/wkhtmltopdf/wkhtmltopdf/releases/...` e do arquivo `...amazonlinux2.x86_64.rpm`.
> **Solução:** usar o pacote `centos7` do repositório `wkhtmltopdf/packaging`, que funciona no Amazon Linux 2.

Se for preciso o caminho `/usr/bin/wkhtmltopdf`:

```bash
sudo ln -s /usr/local/bin/wkhtmltopdf /usr/bin/wkhtmltopdf
```

---

## Etapa 12: Agendamentos (cron)

Referência no manual: seção 11 (Agendamentos). Objetivo: o cron executa as tarefas automáticas do SEI e do SIP a cada minuto.

### Criar o arquivo

> **Atenção:** em `/etc/cron.d`, cada job precisa estar em **uma única linha** (5 campos de horário, usuário `root` e comando). Ao copiar do PDF do manual a linha costuma quebrar em duas.

```bash
sudo tee /etc/cron.d/sei-agendamentos > /dev/null << 'EOF'
* * * * * root /usr/bin/php -c /etc/php.ini /opt/sei/scripts/AgendamentoTarefaSEI.php >> /root/infra_agendamento_sei.log 2>&1
* * * * * root /usr/bin/php -c /etc/php.ini /opt/sip/scripts/AgendamentoTarefaSip.php >> /root/infra_agendamento_sip.log 2>&1
EOF

sudo chmod 644 /etc/cron.d/sei-agendamentos
sudo chown root:root /etc/cron.d/sei-agendamentos
```

O redirecionamento usa `>> arquivo 2>&1`. A ordem do manual (`2>&1 >> arquivo`) deixa os erros fora do arquivo de log.

### Conferir o arquivo

```bash
sudo cat -A /etc/cron.d/sei-agendamentos
```

Devem aparecer **2 linhas**, cada uma terminando em `$`, sem `^M` e sem linha começando por espaços. Se houver `^M`:

```bash
sudo sed -i 's/\r$//' /etc/cron.d/sei-agendamentos
```

### Verificar a execução

```bash
sleep 120
sudo tail -20 /var/log/cron
sudo ls -l /root/infra_agendamento_sei.log /root/infra_agendamento_sip.log
sudo tail /root/infra_agendamento_sei.log /root/infra_agendamento_sip.log
```

**Esperado:**
- `/var/log/cron` **sem** `bad minute` e com linhas `CMD (... AgendamentoTarefaSEI.php >> /root/infra_agendamento_sei.log 2>&1)`.
- Os dois logs existem (podem estar vazios), sem `Fatal error`, `Warning` ou erro de conexão com o banco.

### Problema encontrado no laboratório

O arquivo foi criado com cada job quebrado em duas linhas. O `/var/log/cron` mostrou:

```
(CRON) bad minute (/etc/cron.d/sei-agendamentos)
(root) CMD (/usr/bin/php -c /etc/php.ini /opt/sei/scripts/AgendamentoTarefaSEI.php )
```

O cron tratou a segunda linha (`2>&1 >> ...`) como outro job e a rejeitou. O comando rodou todo minuto sem o redirecionamento, por isso os logs em `/root` nunca foram criados. A solução é recriar o arquivo com uma linha por job, como acima.

### Confirmar no SEI/SIP

No navegador: **Infra > Agendamentos** (no SEI e no SIP) e **Infra > Log**. Os agendamentos de teste rodam a cada 5 minutos e deixam registro no log.

Pelo terminal:

```bash
mysql -u sei_user -p -e "SELECT dth_log, texto FROM sei.infra_log ORDER BY dth_log DESC LIMIT 5\G"
```

O SIP tem a mesma estrutura em `sip.infra_log`.

---

## Conferência final

```bash
sudo ss -tlnp | grep -E ':(80|443|3306|8983|11211) '
systemctl is-enabled httpd php-fpm mariadb memcached solr
```

| Porta | Serviço | Esperado |
|---|---|---|
| 80 | httpd | `LISTEN` |
| 3306 | mariadb | `LISTEN` |
| 8983 | solr (java) | `LISTEN` |
| 11211 | memcached | `LISTEN` em `127.0.0.1` |

No laboratório, o `php-fpm` apareceu como `disabled`. Se o manual exigir, habilite com `sudo systemctl enable php-fpm`.

---

## Problemas conhecidos (resumo)

| Área | Sintoma | Solução |
|---|---|---|
| yum | `No package java.1.8.0-openjdk-headless` | nome correto: `java-1.8.0-openjdk-headless` |
| Apache | `403 Forbidden` com pasta vazia | esperado; só `Connection refused` é erro |
| PHP | `Requires: epel-release = 7` (Remi) | `sudo amazon-linux-extras enable php7.3` |
| PHP | extensão `memcache` indisponível via yum | `pecl install memcache-4.0.5.2` + `/etc/php.d/50-memcache.ini` |
| SIP/SEI | `Class "" not found` | instalar extensões faltantes e `memcache` |
| Banco | `Access denied` para `sei_user` | `SET PASSWORD` e revisar `ConfiguracaoSEI.php`/`ConfiguracaoSip.php` |
| Permissões | `cp ... /*` falha com `cannot stat` | usar `/.` ou `sudo sh -c` |
| Permissões | `php -l` / `tree` com erro | rodar com `sudo` (pastas restritas) |
| Solr | `$'\r': command not found` | `sed -i 's/\r$//'` nos arquivos |
| Solr | `Invalid maximum heap size: -Xmxlg` | trocar por `-Xmx1g` em `solr.in.sh` |
| Solr | arquivo `jetty.xml` quebrado | `:q!`, restaurar `.original` e inserir o bloco via `sed` |
| wkhtmltopdf | `404` no download | usar o repositório `wkhtmltopdf/packaging` (pacote `centos7`) |
| cron | `bad minute` e logs não criados | uma linha por job em `/etc/cron.d` |

---

## Pendências

- [ ] Configurar o bloco **Solr** no `ConfiguracaoSEI.php` (endereço `localhost:8983` e nomes dos cores).
- [ ] Conferir o conteúdo de `/etc/cron.d/sei-limpeza` com o manual (seção 3).
- [ ] Confirmar o estado final do SELinux (`getenforce`) e documentar.
- [ ] Voltar `display_errors` para `Off` após o diagnóstico.
- [ ] Instalar/validar extensões ainda sinalizadas na verificação: `igbinary`, `ldap`, `uploadprogress`, `sodium`.
- [ ] Instalar o `ffmpeg` e conferir as fontes TrueType (requisitos do manual).
- [ ] Revisar o `GRANT` do `sei_user` (manual exige sem DDL).
- [ ] Etapa de HTTPS (`https => true` e `session.cookie_secure = 1`).
- [ ] Trocar a senha padrão `teste` do usuário do SIP.
- [ ] Validar login completo e registros em Infra/Log.

---

