### **EXERCÍCIO - PUBLICAR UMA APLICAÇÃO PHP EM UMA VM**

O sistema operacional utilizado para a resolução da atividade foi o Oracle Linux 

Passo 1 -  Atualizar o sistema usando `sudo apt update -y`

Passo 2 - Instalar o git - `sudo sudo apt install git -y`

Passo 3 - Verificar a instalação do git - `git --version`

Passo 4 - Clonar o repositório na VM - `git clone https://github.com/thiagoinacioalves/treinamento.git`

Passo 5 - Entrar na pasta - `cd treinamento/Linux/app-php`

Passo 6 - Conferir os arquivos - `ls`

Passo 7 - Conferir o conteudo PHP - `cat index.php`

Passo 8  - Instalar o apache - `sudo dnf install httpd -y`

Passo  9 - Habilitar o Apache para iniciar com o sistema- `sudo systemctl enable httpd`

Passo 10 - Iniciar/reiniciar o Apache - `sudo systemctl restart httpd`

Passo 11- Verificar se o Apache está funcionando - `sudo systemctl status httpd`

Passo 12 - Testar o Apache localmente - `curl http://localhost`

Passo 13 - Liberar acesso HTTP - `sudo firewall-cmd --permanent --add-service=http`

Passo 14 - Recarregar o Firewall - `sudo firewall-cmd --reload`

Passo 15 - Conferir se HTTP está liberado - `sudo firewall-cmd --list-services`

Passo 16 - Testar o servidor pelo navegador - `hostname -I`

Passo 17 - No computador, abra: `http://IP_DA_VM`

Passo 18 - Instalação do PHP - `sudo dnf module list php`

Passo 19 - Verificar a instalação do PHP - `php -v`

Passo 20 - Reiniciar o Apache - `sudo systemctl restart httpd`

Passo 21 - Testar o PHP criando um arquivo de teste- `sudo nano /var/www/html/info.php`

Coloque:

<?php
phpinfo();
?>

Salve:

**Ctrl + O → Enter → Ctrl + X**

Passo 22 - Testar o PHP pelo navegador - `hostname -I`

E acesse 

http://IP_DA_VM/info.php

Passo 23 - Remover o arquivo de teste, depois de confirmar que o PHP funciona - `sudo rm /var/www/html/info.php`

Passo 24 - Publicação da aplicação - `cd ~/treinamento/Linux/app-php`

Passo 25 - Conferir os arquivos - `ls -la`

Passo 26 - Verificar o diretório público do Apache - `ls -la /var/www/html/`

Passo 27 - Copiar a aplicação para o Apache - `sudo cp index.php /var/www/html/`

Passo 28 - Conferir se o arquivo foi copiado - `ls -la /var/www/html/`

Passo 29 - Ajustar as permissões - `sudo chmod 644 /var/www/html/index.php`

Depois

`ls -la /var/www/html/index.php`

Passo 30 - **Testar pelo terminal -** `curl http://localhost`

Passo 31 - Testar pelo navegador - `http://IP_DA_VM`

Página sem a alteração

!image.png

Passo 32 - Testar a página - `sudo nano /var/www/html/index.php`

Passo 33 - Alterar a mensagem - `Página publicada com sucesso no Oracle Linux!`

Passo 34 - Salvar - 

Pelo editor de código nano:

**Ctrl + O**

Depois:

**Enter**

Depois:

**Ctrl + X**

35 - Atualizar o navegador e verificar se a atualização foi aplicada
