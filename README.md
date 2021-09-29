# ora2pg-CentOS-7
Procedimento de instalação da ferramenta ora2pg no sistema operacional centos 7.

* * * *
Este documento tem como objetivo auxiliar na instalação da ferramenta ora2pg de forma simplificada.
Ora2pg possui distribuição gratuita e serve para migrar dados dos bancos Oracle ou MySQL para o Postgres. 

O ora2pg uma vez instalado permite a migração de objetos como: tabelas, packages, views, grants, sequence, 
triggers, functions, procedures, tablespace, type, partition, fdw, mview, query, dblink, synonym, directory.
Que pode ocorrer em conjunto (schema completo) ou separadamente. 
Esse e outros parâmetros são configurados no arquivo ora2pg.conf.
  
Requisitos:
* Oracle Instant Client - para a migração é necessária a instalação do oracle instant client ou oracle versão completa.
* Instalação do ora2pg;
* Configuração do ora2pg (ora2pg.conf)

## Instalando o Oracle Instant Client

PARTE 1) Depois de baixar os pacotes RPM no site abaixo, instalar o instant client
	https://www.oracle.com/br/database/technologies/instant-client/linux-x86-64-downloads.html

PARTE 1.1) Instalação de arquivos RPM:
Faça o download dos pacotes RPM do Instant Client desejados (trouxe p o linux-centOS pelo programa WinScp)
# 
	yum install oracle-instantclient12.2-basic-12.2.0.1.0-1.x86_64.rpm
	yum install oracle-instantclient12.2-devel-12.2.0.1.0-1.x86_64.rpm
	yum install oracle-instantclient12.2-jdbc-12.2.0.1.0-1.x86_64.rpm
	yum install oracle-instantclient12.2-sqlplus-12.2.0.1.0-1.x86_64.rpm
	
Para confirmar a instalação, ir até o diretório /usr/lib/oracle/12.2/lib/ e confirmar a existência das bibliotecas compartilhadas (shared libraries) - arquivos "libclntsh.so", "libclntsg.so.12.1", "libocci.so" e "libocci.so.12.1":
#
	ls /usr/lib/oracle/12.2/client64/lib/
o diretório de binários do sqlplus foi criado:
# 	
	ls /usr/lib/oracle/12.2/client64/bin/
os cabeçalhos ficarão (por padrão) neste diretório:
# 
	ls /usr/include/oracle/12.2/client64/
	
  
PARTE 1.2) Configurar a variável de ambiente (LD_LIBRARY_PATH, ORACLE_HOME e PATH)
* LD_LIBRARY_PATH - inclui as bibliotecas compartilhadas do oracle;
* ORACLE_HOME - especifica o local da instalação ou a estrutura de diretório de nível superior para a instalação do banco de dados.
* PATH - inclui o sqlplus e demais binários que estam nesse diretório no PATH
* TNS_ADMIN - especifica o local do arquivo tnsnames.ora - com as strings de conexão
# 
	vi /etc/profile
	export LD_LIBRARY_PATH=/usr/lib/oracle/12.2/client64/lib:$LD_LIBRARY_PATH
	export ORACLE_HOME=/usr/lib/oracle/12.2/client64
	export PATH=/usr/lib/oracle/12.2/client64/bin:$PATH
	export TNS_ADMIN=$ORACLE_HOME/lib/network/admin

depois de editar o aquivo, recarregue-os com o camando abaixo:
# 	
	source /etc/profile
	
PARTE 1.3) co-localizar arquivos de configuração Oracle opcionais (criar os diretórios e depois criar os arquivos)
# 
	sudo mkdir -p /usr/lib/oracle/12.2/client64/lib/network/admin
	
No arquivo "tnsnames.ora", incluir as String de conexão conforme exemplo abaixo:
# 
	vi /usr/lib/oracle/12.2/client64/lib/network/admin/tnsnames.ora
	meuBancoLocal =
	  (DESCRIPTION =
	    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.X.X)(PORT = 1521))
	    (CONNECT_DATA =
	      (SERVER = DEDICATED)
	      (SERVICE_NAME = orcl)
	    )
	  )

PARTE 1.5) Conectando ao Banco de dados pelo SQLPLUS
Exemplo 1) 
* como exemplo de montagem: sqlplus user/pass@local_SID 
# 
	sqlplus maria/1234@meuBancoLocal
	
Exemplo 2) 
* com a string de conexão - exemplo de montagem: 
# 
	sqlplus "maria@(DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.X.X)(PORT = 1521))(CONNECT_DATA =(SERVER = DEDICATED)(SERVICE_NAME = orcl)))"
	
Exemplo 3)
* Sem a senha
# 	
	sqlplus maria@meuBancoLocal
	
Exemplo 4) 
* exportando a variável de ambiente para uso temporário
#  
	export ORACLE_SID=meuBancoLocal 

PARTE 1.7) Consultar se o Banco de Dados Oracle está rodando (conferir no s.o. que ele está instalado) Consultar se o banco (no exemplo abaixo o banco possui nome "orcl", nome = orcl) está "READY"
# 
	lsnrctl status 
Se ele não estiver "READY", inicializar o banco com o comando
	# lsnrctl start
	
Se ele estiver "READY", rodar o comando no terminal (onde está o sqlplus da máquina cliente)
# 	
	telnet ipDaMaquinaQueRodaOBanco portaPadraoDoOracle
	telnet 192.168.x.x 1521 
	Trying 192.168.x.x...
	telnet: connect to address 192.168.1.2: Connection timed out
	
Se o erro acima aparecer, conferir o FireWall da máquina onde encontra o Banco de Dados. Caso a porta não esteja liberada, acrescente-a.
	
Se aparecer conectado é porque está se conectando com o banco (conforme o exemplo abaixo)
#
	telnet 192.168.x.x 1521 
	Trying 192.168.x.x...
	Connected to 192.168.x.x.
	Escape character is '^]'.

Se der certo a conexão, tentar acesso via sqlplus na máquina cliente.	
	
#################################################

Instalando o ORA2PG

PARTE 2) Baixar e instalar o cliente Git
# 	
	yum install git
		
PARTE 2.2) Instalar o Gerenciador de Pacote do Perl
# 
	yum install perl-App-cpanminus.noarch (confirmar se precisa na próxima instalação)
	yum install perl-DBD-Pg perl perl-devel perl-DBI perl-CPAN -y
	yum install gcc
	
PARTE 2.3) Instalando os módulos necessários para o ora2pg
#
	1. cpan
	2. cpan> install CPAN
	3. cpan> reload cpan
	4. cpan> install DBI
	5. cpan> install DBD::Oracle
	
PARTE 2.4) Instalando o ORA2PG
Ir até o diretório onde ele foi clonado do git
# 
	git clone https://github.com/darold/ora2pg.git
	cd ora2pg
	perl Makefile.PL
	make && make install	
	
#################################################

	
Configurando o ORA2PG

PARTE 3) Configurando o arquivo ora2pg.conf
	Ir até a localização do arquivo
# 
	cd /etc/ora2pg
Copiar o modelo existente no arquivo ".dist" em um novo arquivo ".conf"
# 
	cp ora2pg.conf.dist ora2pg.conf
	
Configurações: 
Editar o arquivo com os dados dos objetos que serão migrados assim como da string de conexão do banco oracle
#
	vi /etc/ora2pg/ora2pg.conf
	
Enjoy!

=)	
	
	
Parâmetros e maiores informações: 
* https://github.com/darold/ora2pg/blob/master/README
* https://ora2pg.darold.net/
	
	
	
