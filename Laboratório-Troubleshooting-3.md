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


## Problema do laboratório - Crie um script para automatizar a resolução caso alguma aplicação php caia utilizando a mesma vm do desafio 2

### Resolução
Neste desafio em específico foi utilizado o git bash

### Passo a passo do script

### Passo 1 — Verificar se o container do PHP está rodando e "saudável" (healthy)
```bash

```

### Passo 2 — Se estiver parado ou com erro, tentar reiniciar automaticamente
```bash

```

### Passo 3 — Registrar tudo em um log com data/hora
```bash

```

### Passo 4 — Rodar de tempos em tempos via cron, sem precisar de intervenção manual
```bash

```
