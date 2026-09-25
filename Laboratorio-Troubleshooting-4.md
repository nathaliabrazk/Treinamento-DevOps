
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


## Problema do laboratório - Crie um agendamento pra restartar o serviço do docker e gravar um log do horário que foi restartado o container, utilize a mesma vm do lab 2

### Resolução
Neste desafio em específico foi utilizado o git bash

### Destrinchando o script


### Utilizando o script na prática 

### Passo 1 — Teanferir o script para a VM
```bash
cd ~/Downloads
scp -i lab.pem restart-scheduled.sh ubuntu@32.193.247.226:/home/ubuntu/
```
![Passo 1](images/ts4-1.0.png)

### Passo 2 — Conectar na VM e dar permissão de execução
```bash
ssh -i lab.pem ubuntu@32.193.247.226
chmod +x /home/ubuntu/restart-scheduled.sh
```
![Passo 2](images/ts4-2.0.png)

### Passo 3 — Testar manualmente
```bash
/home/ubuntu/restart-scheduled.sh
cat /home/ubuntu/restart-scheduled.log
```
![Passo 3](images/ts4-3.0.png)


### Passo 4 — Definindo parâmetros via cron
Agendar via cron

Escolha a frequência. Alguns exemplos:

A cada 1 hora:
```bash
(crontab -l 2>/dev/null; echo "0 * * * * /home/ubuntu/restart-scheduled.sh") | crontab -
```
![Passo 4](images/ts4-4.0.png)

