## Objetivo

Você recebeu acesso a um ambiente Linux que executa uma aplicação web simples. A aplicação possui três camadas:

- uma interface web;
- uma API responsável pela lógica da aplicação;
- um banco de dados que armazena os dados exibidos.

O ambiente foi preparado com uma falha controlada. Sua missão é investigar o estado da aplicação, identificar a causa raiz e reativar o fluxo completo para que a página volte a funcionar.

## Resultado esperado

Ao concluir o laboratório:

- a aplicação estará acessível pelo endereço recebido;
- a interface conseguirá consultar e exibir os dados do banco;
- os serviços necessários estarão funcionando;
- você conseguirá explicar em qual camada estava o problema e por que a correção resolveu o incidente.

## O que será avaliado

A avaliação considera a capacidade de raciocinar sobre uma aplicação distribuída em camadas, distinguir sintoma de causa, validar a recuperação e comunicar claramente o diagnóstico realizado.

O ambiente é exclusivamente para treinamento. Não altere configurações de outros trainees e não compartilhe a chave de acesso recebida.

# Resolução
Neste desafio em específico foi utilizado o git bash

### Passo 1 — Localizar o arquivo lab.pem
```bash
cd ~/Downloads
```
![Passo 1](images/ts2-1.0.png)
```bash
ls
```
![Passo 1](images/ts2-1.1.png)

### Passo 2 — Proteger a sua chave privada (o arquivo lab.pem), garantindo que apenas você possa ler o arquivo e ninguém mais no sistema tenha acesso a ele.
```bash
chmod 400 lab.pem
```
![Passo 2](images/ts2-2.0.png)
### Passo 3 — Conectar na máquina Linux
```bash
ssh -i lab.pem ubuntu@32.193.247.226
```
![Passo 3](images/ts2-3.0.png)
### Passo 4 — Testar no navegador
```bash
http://ec2-32-193-247-226.compute-1.amazonaws.com/
```
![Passo 4](images/ts2-4.0.png)

Aqui começa o troubleshooting de forma organizada, realização de testes
### Passo 5 — Verificar se o docker está rodando
```bash
sudo systemctl status docker
```
![Passo 5](images/ts2-5.0.png)
### Passo 6 — Verificar conteiners docker
```bash
cd /home/ubuntu/aws-docker
```

![Passo 6](images/ts2-6.0.png)


Problema encontrado: 
o Docker nem está instalado na instância. Por isso não há nada escutando na porta 80 e o navegador recebe ERR_CONNECTION_RESET — o SO aceita a conexão TCP, mas não existe processo nenhum para responder.
### Passo 7 — Confirmar que o pacote docker não existe
```bash
sudo systemctl status docker
```
![Passo 7](images/ts2-7.0.png)
```bash
 cd /home/ubuntu/aws-docker
```
![Passo 7](images/ts2-7.1.png)
```bash
which docker
docker --version
```
![Passo 7](images/ts2-7.2.png)
### Passo 8 — Instalar o Docker Engine (Ubuntu 24.04)
```bash
# Remove pacotes conflitantes antigos, se existirem
sudo apt-get remove -y docker docker-engine docker.io containerd runc 2>/dev/null

# Atualiza índices
sudo apt-get update

# Dependências
sudo apt-get install -y ca-certificates curl gnupg

# Chave GPG oficial do Docker
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Repositório
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

# Instala Docker + Compose plugin
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
### Passo 9 — Habilitar e iniciar o docker
```bash
sudo systemctl enable --now docker
sudo systemctl status docker
```
![Passo 9](images/ts2-9.0.png)
### Passo 10 — Verificar instalação
```bash
docker --version
```
![Passo 10](images/ts2-10.0.png)
### Passo 11 — Rodar docker sem sudo
```bash
sudo usermod -aG docker $USER
```
![Passo 1 ](images/ts2-11.0.png)
### Passo 12 — 
```bash

```
![Passo 10](images/ts2-12.0.png)
