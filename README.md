# Integração de URA com Asterisk usando a API da Google para AGI

Este repositório contém um guia passo a passo e um script de exemplo para integrar um sistema de Resposta Audível Interativa (URA) com o Asterisk, aproveitando a poderosa API da Google para AGI (Gateway de Interface de Aplicações).

## Pré-requisitos

- Um servidor Asterisk com Issabel funcional.
- Script em .php para a integração da URA com a API da Google.
- Configuração do Banco de dados (nesse tutorial foi usado o MariaDB).

## Configuração do Asterisk

### 1. Instalação do Asterisk AGI Scripting Gateway

Certifique-se de que o módulo AGI está instalado no seu servidor Asterisk.

```bash
yum install perl-libwww-perl perl-IO-Socket-SSL perl-LWP-Protocol-https mpg123 git nano -y
'''
# Integração de URA com Asterisk usando a API da Google para AGI

Este repositório contém um guia passo a passo e um script de exemplo para integrar um sistema de Resposta Audível Interativa (URA) com o Asterisk, aproveitando a poderosa API da Google para AGI (Gateway de Interface de Aplicações).

## Pré-requisitos
