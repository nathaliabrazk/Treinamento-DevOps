
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
### Passo 12 - Testar o Apache localmente
```bash
curl http://localhost
```
### Passo 13 - Liberar acesso HTTP 
```bash
sudo firewall-cmd --permanent --add-service=http
```
### Passo 14 - Recarregar o Firewall
```bash
sudo firewall-cmd --reload
```
### Passo 15 - Conferir se HTTP está liberado
```bash
sudo firewall-cmd --list-services
```
### Passo 16 - Testar o servidor pelo navegador
```bash 
hostname -I
```
### Passo 17 - Acessar a VM via navegador
```bash
No computador, abra: http://IP_DA_VM
```
### Passo 18 - Instalação do PHP
```bash
sudo dnf module list php
```
### Passo 19 - Verificar a instalação do PHP
```bash
php -v
```
### Passo 20 - Reiniciar o Apache
```bash
sudo systemctl restart httpd
```
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

### Passo 22 - Testar o PHP pelo navegador
```bash
hostname -I

E acesse 

http://IP_DA_VM/info.php
```
### Passo 23 - Remover o arquivo de teste, depois de confirmar que o PHP funciona
```bash
sudo rm /var/www/html/info.php
```
### Passo 24 - Publicação da aplicação
```bash
cd ~/treinamento/Linux/app-php
```
### Passo 25 - Conferir os arquivos
```bash
ls -la
```
### Passo 26 - Verificar o diretório público do Apache
```bash
ls -la /var/www/html/
```
### Passo 27 - Copiar a aplicação para o Apache 
```bash 
sudo cp index.php /var/www/html/
```
### Passo 28 - Conferir se o arquivo foi copiado
```bash
ls -la /var/www/html/
```
### Passo 29 - Ajustar as permissões index.php definindo quem pode ler, escrever ou executar esse arquivo
```bash
sudo chmod 644 /var/www/html/index.php

Depois

ls -la /var/www/html/index.php
```
### Passo 30 - Testar pelo terminal
```bash
curl http://localhost
```
### Passo 31 - Testar pelo navegador
```bash
http://IP_DA_VM
```
![Página sem a alteração](images/pagina-sem-alteracao.png)

### Passo 32 - Testar a página
```bash
sudo nano /var/www/html/index.php
```
**A partir deste ponto, a atividade foge do escopo da questão. A etapa foi realizada apenas para testar o funcionamento e a exibição da página**
### Passo 33 - Alterar a mensagem
```bash 
Página publicada com sucesso no Oracle Linux!
```
### Passo 34 Salvar  
```bash
Pelo editor de código nano:

Ctrl + O

Depois:

**Enter**

Depois:

Ctrl + X
```
### 35 - Atualizar o navegador e verificar se a atualização foi aplicada
![Página Atualizada](images/pagina-atualizada.png)
