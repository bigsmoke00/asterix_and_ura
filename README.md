# Integração de URA com Asterisk usando a API da Google para AGI

Este repositório contém um guia passo a passo e um script de exemplo para integrar um sistema de Resposta Audível Interativa (URA) com o Asterisk, aproveitando a poderosa API da Google para AGI (Gateway de Interface de Aplicações).

## Pré-requisitos

- Um servidor Asterisk funcional.
- Credenciais válidas para a API da Google Cloud.
- Conhecimento básico de linguagens de script (recomenda-se Python).

## Configuração do Asterisk

1. **Instalação do Asterisk AGI Scripting Gateway:**
   Certifique-se de que o módulo AGI está instalado no seu servidor Asterisk.

   ```bash
   sudo apt-get install asterisk
