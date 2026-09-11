
## Resolução
### **EXERCÍCIO - PUBLICAR UMA APLICAÇÃO PHP EM UMA VM**
# 🖥️ Resolução — Publicação de uma aplicação PHP em uma VM

## 📌 Sobre a atividade

Este exercício tem como objetivo preparar uma máquina virtual com **Oracle Linux**, instalar e configurar um servidor web com suporte ao **PHP**, publicar uma aplicação PHP e validar seu funcionamento por meio do navegador.

Durante a atividade foram praticados conceitos de:

- 🐧 Administração do Linux
- 🔧 Instalação e configuração de serviços
- 📦 Git e clonagem de repositórios
- 🌐 Servidor web Apache (`httpd`)
- 🐘 PHP
- 🔥 Configuração de firewall
- 📁 Diretórios e permissões
- 🚀 Publicação de uma aplicação web

---

# 🚀 Resolução

### Passo 1 — Atualizar o sistema
```bash
sudo dnf update -y
```
![Página Atualizada](images/1.0.png)
![Página Atualizada](images/1.1.png)
### Passo 2 — Instalar o git
```bash
sudo sudo dnf install git -y
```
![Página Atualizada](images/2.0.png)
![Página Atualizada](images/2.1.png)
### Passo 3 - Verificar a instalação do git 
```bash
git --version
```
![Página Atualizada](images/3.png)
### Passo 4 - Clonar o repositório na VM 
```bash
git clone https://github.com/thiagoinacioalves/treinamento.git`
```
![Página Atualizada](images/4.png)
### Passo 5 - Entrar na pasta 
```bash
cd treinamento/Linux/app-php
```
![Página Atualizada](images/5.png)
### Passo 6 - Conferir os arquivos 
```bash
ls
```
![Página Atualizada](images/6.png)
### Passo 7 - Conferir o conteudo PHP 
```bash
cat index.php
```
![Página Atualizada](images/7.0.png)
![Página Atualizada](images/7.1.png)
### Passo 8  - Instalar o apache 
```bash
sudo dnf install httpd -y
```
![Página Atualizada](images/8.0.png)
![Página Atualizada](images/8.1.png)
### Passo  9 - Habilitar o Apache para iniciar com o sistema
```bash
sudo systemctl enable httpd
```
![Página Atualizada](images/9.png)
### Passo 10 - Iniciar/reiniciar o Apache 
```bash
sudo systemctl restart httpd
```
![Página Atualizada](images/10.png)
### Passo 11- Verificar se o Apache está funcionando
```bash
sudo systemctl status httpd
```
![Página Atualizada](images/11.png)
### Passo 12 - Testar o Apache localmente
```bash
curl http://localhost
```
![Página Atualizada](images/12.0.png)
![Página Atualizada](images/12.1.png)
![Página Atualizada](images/12.2.png)
### Passo 13 - Liberar acesso HTTP 
```bash
sudo firewall-cmd --permanent --add-service=http
```
![Página Atualizada](images/13.png)
### Passo 14 - Recarregar o Firewall
```bash
sudo firewall-cmd --reload
```
![Página Atualizada](images/14.png)
### Passo 15 - Conferir se HTTP está liberado
```bash
sudo firewall-cmd --list-services
```
![Página Atualizada](images/15.png)
### Passo 16 - Testar o servidor pelo navegador
```bash 
hostname -I
```
![Página Atualizada](images/16.png)
### Passo 17 - Acessar a VM via navegador
```bash
No computador, abra: http://IP_DA_VM
```
![Página Atualizada](images/17.png)
### Passo 18 - Instalação do PHP
```bash
sudo dnf install httpd php -y
```
![Página Atualizada](images/18.0.png)
![Página Atualizada](images/18.1.png)
### Passo 19 - Verificar a instalação do PHP
```bash
php -v
```
![Página Atualizada](images/19.png)
### Passo 20 - Reiniciar o Apache
```bash
sudo systemctl restart httpd
```
![Página Atualizada](images/20.png)
### Passo 21 - Testar o PHP criando um arquivo de teste
```bash
sudo nano /var/www/html/info.php
Coloque:

<?php
phpinfo();
?>

Salve:

Ctrl + O → Enter → Ctrl + X
```
![Página Atualizada](images/21.png)

### Passo 22 - Testar o PHP pelo navegador
```bash
hostname -I

E acesse 

http://IP_DA_VM/info.php
```
![Página Atualizada](images/22.png)
### Passo 23 - Remover o arquivo de teste, depois de confirmar que o PHP funciona
```bash
sudo rm /var/www/html/info.php
```
![Página Atualizada](images/23.png)
### Passo 24 - Publicação da aplicação
```bash
cd ~/treinamento/Linux/app-php
```
![Página Atualizada](images/24.png)
### Passo 25 - Conferir os arquivos
```bash
ls -la
```
![Página Atualizada](images/25.png)
### Passo 26 - Verificar o diretório público do Apache
```bash
ls -la /var/www/html/
```
![Página Atualizada](images/26.png)
### Passo 27 - Copiar a aplicação para o Apache 
```bash 
sudo cp index.php /var/www/html/
```
![Página Atualizada](images/27.png)
### Passo 28 - Conferir se o arquivo foi copiado
```bash
ls -la /var/www/html/
```
![Página Atualizada](images/28.png)
### Passo 29 - Ajustar as permissões index.php definindo quem pode ler, escrever ou executar esse arquivo
```bash
sudo chmod 644 /var/www/html/index.php

Depois

ls -la /var/www/html/index.php
```
![Página Atualizada](images/29.png)
### Passo 30 - Testar pelo terminal
```bash
curl http://localhost
```
![Página Atualizada](images/30.0.png)
![Página Atualizada](images/30.1.png)
### Passo 31 - Testar pelo navegador
```bash
http://IP_DA_VM
```
![Página Atualizada](images/31.png)
![Página sem a alteração](images/pagina-sem-alteracao.png)

### Passo 32 - Testar a página
```bash
sudo nano /var/www/html/index.php
```
![Página Atualizada](images/32.0.png)
![Página Atualizada](images/32.1.png)
**A partir deste ponto, a atividade foge do escopo da questão. A etapa foi realizada apenas para testar o funcionamento e a exibição da página**
### Passo 33 - Alterar a mensagem e salvar
```bash 
Página publicada com sucesso no Oracle Linux!

Pelo editor de código nano:

Ctrl + O

Depois:

**Enter**

Depois:

Ctrl + X
```
![Página Atualizada](images/33.png)
### 34 - Atualizar o navegador e verificar se a atualização foi aplicada
![Página Atualizada](images/34)
