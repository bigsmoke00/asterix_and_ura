# Integração de URA com Asterisk usando a API da Google para AGI

Este repositório contém um guia passo a passo e um script de exemplo para integrar um sistema de Resposta Audível Interativa (URA) com o Asterisk, aproveitando a poderosa API da googleTTS.

## Asterisk

O Asterisk é uma plataforma de código aberto para comunicações, especialmente conhecida por suas capacidades de PBX (Private Branch Exchange). Desenvolvido pela Digium (agora parte da Sangoma Technologies), o Asterisk oferece uma infraestrutura flexível para construir sistemas de telefonia e comunicação.

## Principais características do Asterisk:

- Código Aberto: Como software de código aberto, o Asterisk permite que desenvolvedores e empresas personalizem e adaptem suas funcionalidades conforme necessário.

- Suporte a Protocolos: Integra-se a uma variedade de protocolos de comunicação, incluindo SIP (Session Initiation Protocol), IAX (Inter-Asterisk eXchange), H.323, entre outros.

- Recursos Avançados: Oferece uma gama de recursos avançados, como voicemail, IVR (Interactive Voice Response), conferência, chamadas em espera e muito mais.

- Flexibilidade e Escalabilidade: Pode ser implementado em ambientes diversos, desde pequenas empresas até grandes corporações, e sua flexibilidade permite a construção de sistemas de telefonia altamente personalizados.


## Issabel

Issabel é uma distribuição de software PBX baseada no Asterisk, projetada para simplificar a implementação e gerenciamento de sistemas de telefonia. Ela integra diversas ferramentas e módulos em uma interface única, proporcionando uma solução abrangente para as necessidades de comunicação das organizações.

## Principais características do Issabel:

- Interface Gráfica Intuitiva: Oferece uma interface de usuário amigável, permitindo a administração e configuração do PBX por meio de um painel gráfico.

- Módulos Integrados: Vem com uma variedade de módulos integrados, incluindo relatórios, monitoramento em tempo real, configuração de trunk SIP, entre outros.

- Recursos Avançados: Incorpora recursos avançados do Asterisk, permitindo a implementação de IVR, gravação de chamadas, conferências, e muito mais.

- Colaboração e Integração: Suporta integração com aplicativos de colaboração, como email e CRM (Customer Relationship Management), melhorando a eficiência nas comunicações empresariais.

- Segurança: Oferece recursos de segurança, como controle de acesso, autenticação e criptografia para proteger as comunicações.




## Pré-requisitos

- Um servidor Asterisk com Issabel funcional.
- Configuração do Banco de dados (nesse tutorial foi usado o MariaDB).
- Script em .php para a integração da URA com a API googletts.

Ao combinar a robustez do Asterisk com a facilidade de uso do Issabel, as organizações podem implantar sistemas de telefonia eficientes e escaláveis, adaptados às suas necessidades específicas. Essas soluções desempenham um papel crucial na modernização das comunicações empresariais, proporcionando flexibilidade, economia de custos e uma variedade de recursos avançados.


## Criando ramal 3000

![image](https://github.com/bigsmoke00/asterix_and_ura/assets/91279736/e06afd04-7e9e-43d3-b19f-700567390fe6)


## Definindo senha

![image](https://github.com/bigsmoke00/asterix_and_ura/assets/91279736/64a52a5c-8313-403a-b45f-0ae468150b8a)

## Aplicando configuração de ramal

![image](https://github.com/bigsmoke00/asterix_and_ura/assets/91279736/7a92f38a-e048-4072-84a4-97b96aba7a76)

## Pacotes Necessarios 

### 1. Instalação de pacotes basicos

```bash
yum install perl-libwww-perl perl-IO-Socket-SSL perl-LWP-Protocol-https mpg123 git nano -y
```
### 2. Clone do repositorio API GOOGLETTS

```bash
cd /root
git clone git://github.com/zaf/asterisk-googletts
cp asterisk-googletts/googletts.agi /var/lib/asterisk/agi-bin/
cd /var/lib/asterisk/agi-bin/
```

### 3. Configurando Permissões

```bash
chmod 700 googletts.agi
chown asterisk:asterisk googletts.agi
```

### 4. Configurando bases do MYSQL

```bash
mysql -u root -ptoor
CREATE DATABASE tts;
USE tts;
CREATE TABLE cliente( id INT(11) UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY, matricula VARCHAR(11) NOT NULL, nome VARCHAR(100) NOT NULL);
CREATE INDEX idx_matricula ON cliente(matricula);
INSERT INTO cliente (matricula,nome) VALUES ('20211380043','Geraldo Cruz'),('1','ifpb');
```

### 5. Criando AGI para consulta ao MYSQL

```bash
nano /var/lib/asterisk/agi-bin/checkmatricula.agi
```

```bash
#!/usr/bin/php -q

<?php

include('phpagi.php');

$matricula = $argv[1];
$agi = new AGI();
$db = mysqli_connect("127.0.0.1", "root", "root", "tts");


if( ! $db ){
    $agi->verbose("Erro ao tentar conectar no MYSQL " . mysqli_connect_error());
    exit(0);

}

$res = $db->query("SELECT * FROM cliente WHERE matricula = '$matricula';");

if( $res->num_rows == 0 ){
    $agi->set_variable("MATRICULASTATUS",0);
    $agi->set_variable("MATRICULAMESSAGE", "MATRICULA NAO ENCONTRADA. DIGITE NOVAMENTE");

}else{
    $row = $res->fetch_object();
    $agi->set_variable("MATRICULASTATUS",1);
    $agi->set_variable("MATRICULAMESSAGE", $row->nome);


}

$res->close();
$db->close();
exit(0);
?>
```

### 6. Configurando Permissões
```bash
chmod 700 /var/lib/asterisk/agi-bin/checkmatricula.agi
chown asterisk:asterisk /var/lib/asterisk/agi-bin/checkmatricula.agi
```

### 7. Exten para testar incluída no from-internal

```bash
nano /etc/asterisk/extensions_custom.conf
```

### 8. Configturação para a URA ficar no ramal 5000 e realizar consultas ao database.

```bash
exten => 5000,1,Verbose(Testando Google TTS);
same => n,Answer();
same => n,Agi(googletts.agi,"Seja bem vindo ao IFPB", pt-BR);
same => n(ini),Agi(googletts.agi,"Por favor, digite a sua matricula",pt-BR);
same => n,Read(matricula,"silence/1",11,,,5);
same => n,Agi(checkmatricula.agi,${matricula});
same => n,GotoIf($[ "${MATRICULASTATUS}" = "1" ]?ok:err);
same => n,Hangup();

same => n(ok),Agi(googletts.agi,"Olá, ${MATRICULAMESSAGE}, escolha uma das opções a seguir.",pt-BR);
same => n,MusicOnHold();
same => n,Hangup();

same => n(err),Agi(googletts.agi,"${MATRICULAMESSAGE}",pt-BR);
same => n,Goto(ini);
```

