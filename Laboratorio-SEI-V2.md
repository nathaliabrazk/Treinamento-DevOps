

# Documentação - SEI/SIP Oracle Linux 8.10

Guia de instalação do SEI e do SIP em uma instância Oracle Linux 8.10, baseado no manual **INSTALACAO 4.0.12.15** e nos problemas encontrados durante o laboratório.

Esta versáo do manual de Instalação é integralmente baseada na documentaçáo do projeto SEI 4.0, disponibilizado pelo Tribunal Regional da 4° Região (TRF4).

> **Convenções**
> - Neste laboratório, a rede em que a máquina cliente estava bloqueava o acesso direto à porta 80 da VM pelo IP público. Para contornar, foi criado um **túnel SSH no MobaXterm** (MobaSSHTunnel), que encaminha uma porta local para a porta 80 da VM.

Os testes foram feitos pelo `localhost`, e não pelo IP público.

| Parâmetro do túnel | Valor |
|---|---|
| Ferramenta | MobaXterm (MobaSSHTunnel) |
| Servidor SSH | `<IP_PUBLICO>`, usuário `opc`, com chave privada |
| Porta local (origem) | `<PORTA_LOCAL>` (ex.: 8080) |
| Destino remoto | `localhost:80` (Apache da VM) |

Equivalente em linha de comando:

```bash
ssh -i <CHAVE_PRIVADA> -L 8080:localhost:80 opc@<IP_PUBLICO>
```

URLs de teste, com o túnel ativo:

- SIP: `http://localhost:8080/sip/`
- SEI: `http://localhost:8080/sei/`

**Observação:** o SEI e o SIP guardam a URL com o IP público (campo `URL` dos arquivos de configuração e tabela `sistema` do banco). Por isso, o login pode redirecionar de volta ao IP público e falhar pelo túnel. Se isso acontecer, ajuste temporariamente a URL para `http://localhost:<PORTA_LOCAL>` e reverta ao final.

---
## 1 - Visão Geral 
Neste laboratório, todos os serviços rodam na mesma VM: uma instância Oracle Linux 8 na OCI (Oracle Cloud, região São Paulo). O manual do SEI foi pensado para produção, com cada serviço em um servidor separado. Aqui eles foram consolidados em uma única máquina para fins de aprendizado.

## 2 - Arquivos de instalação
OS arquivos para a instalação estão disponiveis para download na página de lançamentos (Releases) do projeto no GitHub. O acesso é restrito a usuários previamente autorizados. O pacote de versão contém os arquivos:

* SEI-Fontes-[VERSAO].zip
* SEI-BD-Mysql-[VERSAO].zip
* SEI-BD-SQLServer-[VERSAO].zip
* SEI-BD-Oracle-[VERSAO].zip
* SEI-Novidades-[VERSAO].md


## 3 - Serviços

| Serviço | Função | Porta | Configuração |
|---|---|---|---|
| Apache 2.4 | Servidor web: recebe o navegador e entrega as páginas| 80 (HTTPS na Etapa 12) |/etc/httpd/conf.d/sei.conf|
| PHP 7.3 (php-fpm) | Executa o código do SEI, do SIP e do InfraPHP | socket /run/php-fpm/www.sock| /etc/php.ini e /etc/php-fpm.d/www.conf |
| SIP | Login, usuários, perfis, unidades e permissões | |	/opt/sip/config/ConfiguracaoSip.php|
| SEI | Processos e documentos | | /opt/sei/config/ConfiguracaoSEI.php |
| MariaDB 10.3 | Bases sei e sip (charset latin1) | 3306 | /etc/my.cnf e /etc/my.cnf.d/ |
| Memcached | Cache de sessões e permissões | 11211 (somente local) | /etc/sysconfig/memcached |
| Solr 8.2.0 (Java 8) | Indexação e pesquisa | 8983 | /opt/solr, dados em /dados/data |
| Cron | Agendamentos e limpeza | | /etc/cron.d/ |
| Repositório de arquivos | Documentos anexados | | /sei-nfs/dados |

## 4 - SEI e SIP
O SIP controla o login, e o SEI delega a ele a autenticação. As siglas devem ser idênticas nos dois sistemas e no banco.

| Chave | SIP | SEI |
|---|---|---|
| SiglaSistema| SIP | SEI |
| SiglaOrgaoSistema | LAB | LAB |
| Banco | sip | sei |
| Usuário do banco | sei_user (sem DDL) | sei_user (sem DDL) |
| Cache| localhost:11211 | localhost:11211 |

## 5 - Acesso por túnel SSH
A rede do cliente bloqueava a porta 80 da VM pelo IP público. Foi criado um túnel SSH no MobaXterm (MobaSSHTunnel), e os testes foram feitos pelo localhost.

| Parâmetro | Valor |  
|---|---|
| Servidor SSH | <IP_PUBLICO>, usuário opc, chave privada |
| Porta local | <PORTA_LOCAL> (ex.: 8080) |
| Destino remoto | 	localhost:80 |
| Cache| localhost:11211 |

Equivalente em linha de comando:

`ssh -i <CHAVE_PRIVADA> -L 8080:localhost:80 opc@<IP_PUBLICO>`

URLs de teste: http://localhost:8080/sip/ e http://localhost:8080/sei/.

## 6 - Pendências

### 6.1. Serviços e módulos principais

| Requisito do manual | Lab (esperado)  | Como verificar |
|---|---|---|
| Apache 2.4.6 | 2.4.37 (Oracle Linux 8)  | `httpd -v` |
| PHP 7.3.12 | 7.3.33 no início; módulo Remi 7.4 habilitado depois | ⚠️ confirmar versão | `php -v` e `php-fpm -v` |
| Framework InfraPHP | `/opt/infra` | `php -i \| grep include_path` |
| Aplicação SIP | `/opt/sip` | `curl -I http://localhost/sip/` |
| Aplicação SEI | `/opt/sei` | `curl -I http://localhost/sei/` |
| Acesso ao banco | MariaDB 10.3.39 via `mysqli` | `php -m \| grep -i mysqli` |
| SQL Server, Oracle, PostgreSQL | não usados | N/A | — |
| Java Runtime 1.8 | `java-1.8.0` instalado | `java -version` |
| Memcached | serviço na porta 11211 | `ss -tlnp \| grep 11211` |
| Solr 8.2.0 | serviço systemd `solr` | ver seção 6 |
---

### 6.2. Extensões PHP

Extensões exigidas pelo manual. A extensão deve estar carregada no **PHP-FPM**, e não só na linha de comando.

| Extensão |
|---|
| bcmath, bz2, calendar, ctype |
| curl, date, dom, exif, fileinfo, filter |
| gd, gettext, hash, iconv, intl, json |
| libxml, mbstring, openssl, pcre, Phar |
| SimpleXML, soap, sodium, zip, zlib | 
| mysqli |
| memcache (4.0.4) |
| uploadprogress (1.1.3) |

Verificação em lote:

```bash
for e in bcmath bz2 calendar ctype curl date dom exif fileinfo filter gd gettext hash iconv intl json libxml mbstring openssl pcre Phar SimpleXML soap sodium zip zlib mysqli memcache uploadprogress; do
  php -m | grep -qix "$e" && echo "OK     $e" || echo "FALTA  $e"
done
```

Instalação das que faltarem (com Remi habilitado, os nomes podem variar):

```bash
dnf install -y php-bcmath php-gd php-intl php-mbstring php-soap php-xml php-json php-zip php-mysqlnd php-sodium
systemctl restart php-fpm httpd
```

### 6.3 uploadprogress

O manual aceita o pacote do repositório para a versão do PHP, ou a compilação do código em `/opt/sei/config/uploadprogress` (a pasta existe).

```bash
ls -la /opt/sei/config/uploadprogress
dnf install -y php-pecl-uploadprogress           # opção 1: repositório
```

Se não houver pacote, compilar:

```bash
dnf install -y php-devel gcc make
cd /opt/sei/config/uploadprogress
phpize && ./configure && make && make install
echo 'extension=uploadprogress.so' > /etc/php.d/40-uploadprogress.ini
systemctl restart php-fpm
php -m | grep -i uploadprogress
```

---

## 7. SELinux

### 7.1 Situação atual

O SELinux foi colocado em **permissive** durante o laboratório, para descartar bloqueios nos testes de upload e de conexão.

| Item |
|---|
| `setenforce 0` aplicado |
| `SELINUX=permissive` em `/etc/selinux/config` |
| Retorno a `enforcing` com os contextos corretos |

### 7.2 Mudar o modo agora (sem reiniciar)

```bash
sudo setenforce 0
getenforce
```

Deve responder `Permissive`. Vale só até o próximo reboot.

### 7.3 Tornar permanente

```bash
sudo cp -p /etc/selinux/config /etc/selinux/config.bkp
sudo sed -i 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config
grep -n '^SELINUX=' /etc/selinux/config
```

Evite `SELINUX=disabled`: no modo permissive o SELinux continua registrando o que bloquearia, o que ajuda no diagnóstico, e voltar a `enforcing` não exige reetiquetar o sistema.

### 7.4 Conferir após reiniciar

```bash
sudo reboot
# depois de voltar:
getenforce
sestatus
```

### 7.5 Parâmetros (booleans) exigidos pelo manual

O texto copiado do manual tem erros de digitação. Os comandos corretos são:

```bash
setsebool -P httpd_can_network_connect 1
setsebool -P httpd_can_network_connect_db 1
setsebool -P httpd_can_network_memcache 1
setsebool -P httpd_can_network_connect_ldap 1
setsebool -P httpd_execmem 1
setsebool -P httpd_use_nfs 1
```

| Correção | Texto do manual | Correto |
|---|---|---|
| Memcache | `http_can_network_memcache` | `httpd_can_network_memcache` |
| Execmem | `setbool ... httpd_exemcm` | `setsebool ... httpd_execmem` |
| Banco | `httpd_can_network_coneect_db` | `httpd_can_network_connect_db` |

Conferir o estado:

```bash
getsebool -a | grep -E 'httpd_(can_network_connect|can_network_connect_db|can_network_memcache|can_network_connect_ldap|execmem|use_nfs)'
```

`httpd_use_nfs` só é necessário se o repositório de arquivos ficar em NFS. No laboratório, `/sei-nfs/dados` é um diretório local.

### 7.6 Contexto do repositório de arquivos

```bash
semanage fcontext -a -t httpd_sys_rw_content_t "/sei-nfs/dados(/.*)?"
restorecon -Rv /sei-nfs/dados
```

Se faltar o `semanage`: `dnf install -y policycoreutils-python-utils`.

### 7.7 Voltar para enforcing

Depois de aplicar os booleans e os contextos, e de testar o upload:

```bash
setenforce 1
sed -i 's/^SELINUX=.*/SELINUX=enforcing/' /etc/selinux/config
ausearch -m avc -ts recent | tail -n 20      # se algo falhar, ver o que foi negado
```

Se aparecerem negações, voltar a `setenforce 0`, corrigir o contexto ou boolean indicado e repetir.

---

## 8. Repositório de arquivos e upload

| Item | Verificação |
|---|---|
| `/sei-nfs/dados` existe | `ls -ld /sei-nfs/dados` |
| Dono `apache:apache` e permissão `750` | `stat -c '%U:%G %a' /sei-nfs/dados` |
| `RepositorioArquivos` aponta para `/sei-nfs/dados` | `grep -n RepositorioArquivos /opt/sei/config/ConfiguracaoSEI.php` |
| `/dados` é usado somente pelo Solr | `ls -ld /dados` |
| Apache consegue gravar no repositório | `sudo -u apache touch /sei-nfs/dados/t && sudo -u apache rm /sei-nfs/dados/t` |
| `/opt/sei/temp` é gravável pelo Apache | `ls -ld /opt/sei/temp` |


Criação do repositório:

```bash
mkdir -p /sei-nfs/dados
chown -R apache:apache /sei-nfs/dados
chmod 750 /sei-nfs/dados
```

**Problema aberto:** "Erro desconhecido realizando upload de arquivo". A causa mais provável é o repositório inexistente ou sem permissão. Verificar os logs logo após uma tentativa de upload:

```bash
tail -n 30 /var/log/php-fpm/www-error.log
tail -n 30 /var/log/httpd/error_log
```

**Atenção ao `/dados`:** o histórico mostra `chmod 770 /dados` e `chown solr:apache /dados`. Isso funciona, mas dá ao Apache acesso ao diretório do Solr. O manual recomenda que o repositório do SEI **não** fique em `/dados`.

---

## 9. Solr

| Item |
|---|
| Solr 8.2.0 instalado em `/opt/solr` | 
| Usuário `solr` criado |
| Serviço systemd |
| Cores copiados para `/dados/data` |
| Três cores no `STATUS` |
| Escuta restrita a `127.0.0.1` |
| Porta 8983 fechada no firewall e na OCI |

```bash
/opt/solr/bin/solr stop -all
cp -r ~/sei-pacote/sei/config/solr/sei-cores-8.2.0/* /dados/data/
chown -R solr:solr /dados/data
systemctl reset-failed solr
systemctl restart solr
curl "http://localhost:8983/solr/admin/cores?action=STATUS"
```

Para restringir a `127.0.0.1`, definir `SOLR_JETTY_HOST="127.0.0.1"` no `solr.in.sh` usado pelo serviço e reiniciar.

Também validar que o `ConfiguracaoSEI.php` aponta para o Solr local e que a indexação roda (scripts de indexação e agendamento previstos no manual).

---

## 10. Demais pendências

### 10.1 Aplicação

| Pendência | Prioridade |
|---|---|
| Validar o SEI de ponta a ponta (login pelo SIP, criar processo, anexar documento, pesquisar) | Alta |
| Trocar `Producao` de `false` para `true` em `ConfiguracaoSip.php` e `ConfiguracaoSEI.php` | Alta, no fim |
| Conferir senha do `sei_user` igual nos dois arquivos e no banco | Alta |
| Confirmar que `UPDATE orgao` no banco `sei` ficou com `Laboratorio Trainee`, e não com `SUA_DESCRICAO` | Média |
| Manter `ChaveAcesso` original (backups `.original` existem) | Média |
| Conferir permissão `root:apache 640` nos arquivos de configuração | Média |

### 10.2 Fontes, conversão de PDF e vídeo

```bash
fc-list | wc -l
dnf install -y dejavu-sans-fonts liberation-fonts
which wkhtmltopdf ffmpeg
```

- O `dnf install wkhtmltopdf` foi executado, mas não há saída registrada. Confirmar com `which wkhtmltopdf`.
- O `ffmpeg` costuma não estar nos repositórios padrão do Oracle Linux 8. Verificar com `dnf search ffmpeg`. Se não houver, usar um repositório adicional (como o RPM Fusion) ou um binário estático, registrando a origem na documentação.
- Instalar um conjunto de fontes TrueType e atualizar o cache: `fc-cache -fv`.

### 10.3 Locale e charset

| Item | 
|---|
| `localedef -i pt_BR -f ISO-8859-1 pt_BR.ISO-8859-1` executado |
| `locale -a \| grep pt_BR` lista o locale |
| `setlocale(LC_ALL,'pt_BR')` funciona no PHP | 
| Charset do MariaDB em `latin1` |

### 10.4 Rede e acesso

| Item |
|---|
| `http` liberado no firewalld |
| Regra de entrada da porta 80 na Security List/NSG da OCI | 
| Acesso de teste por túnel SSH (MobaXterm) | 
| Nenhum acesso ao SEI/SIP exposto sem HTTPS em uso real | 
| HTTPS (Etapa 12), com `https = true` nos arquivos de configuração | 
| IP público pode mudar se a VM for recriada (refazer `UPDATE sistema` e `URL`) |

Ao implantar HTTPS: liberar `https` no firewalld, instalar o certificado, ajustar o Apache e mudar `URL`, `pagina_inicial` e `web_service` para `https://`.

### 10.5 Banco e memcached

| Item | Verificação |
|---|---|
| `sei_user` só com DML (sem DDL) em produção | `SHOW GRANTS FOR 'sei_user'@'localhost';` |
| MariaDB e Memcached escutando só localmente | `ss -tlnp \| grep -E '3306\|11211'` |
| Memcached habilitado no boot | `systemctl is-enabled memcached` |
| MariaDB habilitado no boot | `systemctl is-enabled mariadb` |
| Backup dos bancos | `mysqldump` agendado (não configurado) |

### 10.6 Boas práticas

- Nunca versionar `ConfiguracaoSEI.php`, `ConfiguracaoSip.php`, senhas ou chaves privadas.
- Registrar a versão exata do sistema (`cat /etc/os-release`).
- Evitar `dnf remove java*` em servidores com outros serviços Java; ele remove pacotes dependentes.
- Reiniciar `php-fpm` e `httpd` depois de qualquer mudança em `php.ini`.

---

## 11. Problemas já resolvidos (referência)

| Problema | Solução |
|---|---|
| `CREATE: command not found` | Rodar SQL dentro do `mysql>` |
| `Table 'sip.orgao' doesn't exist` | Importar o dump do SIP |
| `No route to host` na porta 80 | `firewall-cmd --permanent --add-service=http` |
| 404 em `/sei` e `/sip` | Criar `/etc/httpd/conf.d/sei.conf` com os `Alias` |
| Navegador mostrando código PHP | `short_open_tag = On` |
| 500 com `Failed opening required 'Infra.php'` | `include_path` com `/opt/infra/infra_php` |
| `solr.service` com porta 8983 ocupada | Parar o Solr manual e usar o systemd |

---

## 12. Script de verificação geral

```bash
echo "== Sistema ==";   cat /etc/os-release | head -n 3; uname -r
echo "== Versões ==";    httpd -v | head -n 1; php -v | head -n 1; mysql -V; java -version 2>&1 | head -n 1
echo "== Serviços ==";   systemctl is-active httpd php-fpm mariadb memcached solr
echo "== Portas ==";     ss -tlnp | grep -E ':(80|3306|11211|8983)'
echo "== SELinux ==";    getenforce; grep -n '^SELINUX=' /etc/selinux/config
getsebool -a | grep -E 'httpd_(can_network_connect|can_network_connect_db|can_network_memcache|can_network_connect_ldap|execmem|use_nfs)'
echo "== PHP ==";        grep -nE '^(short_open_tag|include_path)' /etc/php.ini
for e in bcmath bz2 calendar ctype curl date dom exif fileinfo filter gd gettext hash iconv intl json libxml mbstring openssl pcre Phar SimpleXML soap sodium zip zlib mysqli memcache uploadprogress; do php -m | grep -qix "$e" && echo "OK     $e" || echo "FALTA  $e"; done
echo "== Binários ==";   which ffmpeg wkhtmltopdf
echo "== Arquivos ==";   ls -ld /sei-nfs/dados /dados /opt/sei/temp
echo "== Aplicação ==";  curl -s -o /dev/null -w "SIP: %{http_code}\n" http://localhost/sip/; curl -s -o /dev/null -w "SEI: %{http_code}\n" http://localhost/sei/
echo "== Solr ==";       curl -s "http://localhost:8983/solr/admin/cores?action=STATUS" | head -c 600
echo "== Cron ==";       ls -l /etc/cron.d/
```

---

## 13. Resumo das prioridades

| Prioridade | Pendência |
|---|---|
| Alta | Criar `/sei-nfs/dados` e corrigir o erro de upload |
| Alta | Criar os cores do Solr e validar a pesquisa |
| Alta | Conferir as extensões PHP exigidas (principalmente `gd`, `soap`, `sodium`, `zip`, `memcache`, `uploadprogress`) |
| Alta | Confirmar a versão do PHP (7.3 ou 7.4) e os ajustes do `php.ini` |
| Média | Configurar o Cron do SEI e do SIP |
| Média | Aplicar os booleans do SELinux e voltar a `enforcing` |
| Média | Instalar fontes TrueType, `wkhtmltopdf` e `ffmpeg` |
| Média | Restringir o Solr a `127.0.0.1` |
| Baixa | Backup dos bancos, HTTPS e `Producao = true` ao final |


