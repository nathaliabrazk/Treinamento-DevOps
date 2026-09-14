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
```bash
ls
```
![Passo 1](images/1.0.png)

### Passo 2 — Proteger a sua chave privada (o arquivo lab.pem), garantindo que apenas você possa ler o arquivo e ninguém mais no sistema tenha acesso a ele.
```bash
chmod 400 lab.pem
```

### Passo 3 - Conectar na máquina
```bash
ssh -i lab.pem ubuntu@54.167.237.81
```

### Passo 4 - Conectar na máquina
```bash
ssh -i lab.pem ubuntu@54.167.237.81
```

### Passo 5 - Testar no navegador
```bash
![Passo 1](images/1.0.png)
```
Obs: realizei o teste utilizando o ip 54.167.237.81 porém aparecia um bloqueio, sendo assim testei com o dns: ec2-54-167-237-81.compute-1.amazonaws.com e a aplicação subiu como o esperado
![Erro-IP](images/1.0.png)
![Acerto-DNS](images/1.0.png)

Erro observado na página: "Falha ao consultar a aplicação
Unexpected token '<'"

Aqui começa o troubleshooting de forma organizada, realização de testes 
### Passo 6 - Confirmar o usuário
```bash
whoami

```

### Passo 7 - Verificar a máquina
```bash
hostname
cat /etc/os-release
```
