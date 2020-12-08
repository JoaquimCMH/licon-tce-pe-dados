<h3 align="center">Resturando Base de Dados Aberta de Licitações e Contratos (LICON) do TCE-PE</h3>

<div align="center">

[![Status](https://img.shields.io/badge/status-active-success.svg)]()
[![License](https://img.shields.io/badge/licence-GNU%20Aferro%20V3-blue.svg)](/LICENSE)

</div>

---

<p align="center">
    <br> 
</p>

## 📝 Índice

- [Sobre](#about)
- [Pré-Requisitos](#req)
- [Uso](#usage)


## 🧐 Sobre <a name = "about"></a>

Este repositório fornece meios, através do docker, para facilitar o uso dos dados de contratos e licitações (banco LICON) oriundos do [Tribunal de Contas do estado de Pernambuco](https://www.tce.pe.gov.br/internet/index.php/dados-abertos-visivel/113-dados-abertos-downloads). Com ele é possível restaurar o banco SQL Server com todos os pacotes necessários para o funcionamento da aplicação, como dependências e bibliotecas.

### 🎈 Pré-requisitos <a name="req"></a>

- Garanta que você tenha, no mínimo, **13gb** de espaço livre em seu computador.

- Os serviços utilizam docker para configuração do ambiente e execução do script. Instale o  [docker](https://docs.docker.com/install/) e tenha certeza que você também tem o  [docker-compose](https://docs.docker.com/compose/install/) instalado. 


##  🏁 Uso <a name="usage"></a>

Essas instruções fornecerão acesso ao banco funcionando em sua máquina local para fins de desenvolvimento, execução de pesquisas ou outros meios para promover o controle social.

Os passos para a restauração e utilização do banco de dados são elecandos abaixo:

1. Após ter clonado o repositório, baixe o [arquivo](https://www.tce.pe.gov.br/internet/docs/licon/sagresliconcidadao.zip) com o banco no formato *.zip* e o descompacte na pasta *dados*.

```shell
# clona o repositório
git clone  https://github.com/JoaquimCMH/licon-tce-pe-dados.git
cd licon-tce-pe-dados

# baixa o arquivo compactado com o backup
wget -P dados "https://www.tce.pe.gov.br/internet/docs/licon/sagresliconcidadao.zip"

# descompacta o arquivo na pasta dados
unzip dados/sagresliconcidadao.zip -d dados
```
2. É necessário adicionar no arquivo *docker-compose.yml*, na pasta *config*, o diretório até a pasta *licon-tce-pe-dados*. Sendo assim, no terminal, utilize o comando `pwd` para recuperar o diretório completo e depois insira o resultado onde está escrito "\<insira-aqui-diretorio-completo\>". Essas configurações permitirão que o banco seja restaurado na pasta *mssql*.

3. Execute o docker e crie a pasta onde ficará o arquivo *.bak*.

```shell
cd config

# execute o container
sudo docker-compose up -d

# entre no container 
sudo docker exec -it mssql-licon-tce-pe bash

# crie a pasta backup dentro do container
mkdir /var/opt/mssql/backup

# SAIA do container utilizando ctrl+d
```
4. Copie o .bak para o container.

```shell
sudo docker cp ../dados/sagresLiconCidadao.bak mssql-licon-tce-pe:/var/opt/mssql/backup
```

5. Restaure o banco.
```shell
# entre no container
sudo docker exec -it mssql-licon-tce-pe bash

# execute o sqlcmd
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'S3cr3t-P4ssw0rd!'

# verifique os arquivos do arquivo de backup. 
# (observe na saída se existe SagresLiconCidadao.mdf e SagresLiconCidadao_1.ldf para prosseguir com a importação)
RESTORE FILELISTONLY
FROM DISK = '/var/opt/mssql/backup/sagresLiconCidadao.bak'
GO

# restaure o banco (demora alguns minutos)
RESTORE DATABASE LICON
FROM DISK = '/var/opt/mssql/backup/sagresLiconCidadao.bak' WITH
MOVE 'sagresLiconCidadao' TO '/var/opt/mssql/data/sagresLiconCidadao.mdf',
MOVE 'SagresLiconCidadao_log' TO '/var/opt/mssql/data/SagresLiconCidadao_1.ldf'
GO
```
6. Verifique se a restauração funcionou recuperando as tabelas presente no banco.

```shell
USE LICON
SELECT * FROM information_schema.tables;
GO
```

7. Por fim, para interromper a execução do container execute:
```shell
sudo docker-compose down
```

Informações sobre os dados estão disponíveis na pasta **doc**.

## ⛏️ Ferramentas <a name = "built_using"></a>

- [Docker](https://www.docker.com/) - Containers
- [SQL Server](https://www.microsoft.com/pt-br/sql-server/sql-server-downloads) - Sistema gerenciador de Banco de dados relacional


