# Integração de URA com Asterisk usando a API da Google para AGI

Este repositório contém um guia passo a passo e um script de exemplo para integrar um sistema de Resposta Audível Interativa (URA) com o Asterisk, aproveitando a poderosa API da Google para AGI (Gateway de Interface de Aplicações).

## Pré-requisitos

- Um servidor Asterisk com Issabel funcional.
- Configuração do Banco de dados (nesse tutorial foi usado o MariaDB).
- Script em .php para a integração da URA com a API googletts.

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
$db = mysqli_connect("127.0.0.1", "root", "toor", "tts");


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

### 3. Configurando Permissões
```bash
chmod 700 /var/lib/asterisk/agi-bin/checkmatricula.agi
chown asterisk:asterisk /var/lib/asterisk/agi-bin/checkmatricula.agi
```

### 6. Exten para testar incluída no from-internal

```bash
nano /etc/asterisk/extensions_custom.conf
```

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

