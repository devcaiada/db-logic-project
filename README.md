# Projeto Lógico para Banco de Dados

Neste projeto, iremos criar um banco de dados para um e-commerce, desde a modelagem inicial, a inserção de dados exemplo, algumas consultas até temas mais avançados como procedures e eventos.

## 1. Diagrama Entidade-Relacionamento (ER)

Com o diagrama podemos identificar as Entidade e Atributos e fazer os relacionamentos entre as tabelas.

![ecommerce](https://github.com/devcaiada/db-logic-project/blob/main/assets/ecommerce_ER.png?raw=true)

## 2. Criação do Banco de Dados com MySQL

A partir do MySQL Workbench vamos criar o banco de dados. Criaremos todas as tabelas, os relacionamentos, adicionar as constraints, as chaves primárias e estrangeiras de acordo com o nosso diagrama acima.

```SQL
-- criação do banco de dados para o cenário de E-commerce
create database ecommerce;
use ecommerce;

-- criar tabela cliente
CREATE TABLE clients (
    idClient INT AUTO_INCREMENT PRIMARY KEY,
    Fname VARCHAR(10),
    Minit CHAR(3),
    Lname VARCHAR(20),
    CPF CHAR(11) NOT NULL,
    Address VARCHAR(255),
    ClientType ENUM('PJ', 'PF') NOT NULL,
    CONSTRAINT unique_cpf_client UNIQUE (CPF)
);


alter table clients auto_increment=1;


create table product(
		idProduct int auto_increment primary key,
        Pname varchar(255) not null,
        classification_kids bool default false,
        category enum('Eletrônico','Vestimenta','Brinquedos','Alimentos','Móveis') not null,
        avaliação float default 0,
        size varchar(10)
);

alter table product auto_increment=1;


create table payments(
	idclient int,
    idPayment int,
    typePayment enum('Boleto','Cartão','Dois cartões'),
    limitAvailable float,
    primary key(idClient, idPayment)
);


create table orders(
	idOrder int auto_increment primary key,
    idOrderClient int,
    orderStatus enum('Cancelado','Confirmado','Em processamento') default 'Em processamento',
    orderDescription varchar(255),
    sendValue float default 10,
    paymentCash boolean default false,
    constraint fk_ordes_client foreign key (idOrderClient) references clients(idClient)
			on update cascade
);
alter table orders auto_increment=1;

desc orders;


create table productStorage(
	idProdStorage int auto_increment primary key,
    storageLocation varchar(255),
    quantity int default 0
);
alter table productStorage auto_increment=1;


create table supplier(
	idSupplier int auto_increment primary key,
    SocialName varchar(255) not null,
    CNPJ char(15) not null,
    contact char(11) not null,
    constraint unique_supplier unique (CNPJ)
);
alter table supplier auto_increment=1;

desc supplier;


create table seller(
	idSeller int auto_increment primary key,
    SocialName varchar(255) not null,
    AbstName varchar(255),
    CNPJ char(15),
    CPF char(9),
    location varchar(255),
    contact char(11) not null,
    constraint unique_cnpj_seller unique (CNPJ),
    constraint unique_cpf_seller unique (CPF)
);

alter table seller auto_increment=1;


create table productSeller(
	idPseller int,
    idPproduct int,
    prodQuantity int default 1,
    primary key (idPseller, idPproduct),
    constraint fk_product_seller foreign key (idPseller) references seller(idSeller),
    constraint fk_product_product foreign key (idPproduct) references product(idProduct)
);

desc productSeller;

create table productOrder(
	idPOproduct int,
    idPOorder int,
    poQuantity int default 1,
    poStatus enum('Disponível', 'Sem estoque') default 'Disponível',
    primary key (idPOproduct, idPOorder),
    constraint fk_productorder_product foreign key (idPOproduct) references product(idProduct),
    constraint fk_productorder_order foreign key (idPOorder) references orders(idOrder)

);

create table storageLocation(
	idLproduct int,
    idLstorage int,
    location varchar(255) not null,
    primary key (idLproduct, idLstorage),
    constraint fk_storage_location_product foreign key (idLproduct) references product(idProduct),
    constraint fk_storage_location_storage foreign key (idLstorage) references productStorage(idProdStorage)
);

create table productSupplier(
	idPsSupplier int,
    idPsProduct int,
    quantity int not null,
    primary key (idPsSupplier, idPsProduct),
    constraint fk_product_supplier_supplier foreign key (idPsSupplier) references supplier(idSupplier),
    constraint fk_product_supplier_prodcut foreign key (idPsProduct) references product(idProduct)
);
```

## 3. Inserção de Dados Exemplo

Com o banco de dados pronto vamos persistir alguns dados para podermos fazer consultas e responder algumas perguntas.

```SQL

-- idClient, Fname, Minit, Lname, CPF, Address
insert into Clients (Fname, Minit, Lname, CPF, address)
	   values('Maria','M','Silva', 12346789, 'rua silva de prata 29, Carangola - Cidade das flores'),
		     ('Matheus','O','Pimentel', 987654321,'rua alemeda 289, Centro - Cidade das flores'),
			 ('Ricardo','F','Silva', 45678913,'avenida alemeda vinha 1009, Centro - Cidade das flores'),
			 ('Julia','S','França', 789123456,'rua lareijras 861, Centro - Cidade das flores'),
			 ('Roberta','G','Assis', 98745631,'avenidade koller 19, Centro - Cidade das flores'),
			 ('Isabela','M','Cruz', 654789123,'rua alemeda das flores 28, Centro - Cidade das flores');


-- idProduct, Pname, classification_kids boolean, category('Eletrônico','Vestimenta','Brinquedos','Alimentos','Móveis'), avaliação, size
insert into product (Pname, classification_kids, category, avaliação, size) values
							  ('Fone de ouvido',false,'Eletrônico','4',null),
                              ('Barbie Elsa',true,'Brinquedos','3',null),
                              ('Body Carters',true,'Vestimenta','5',null),
                              ('Microfone Vedo - Youtuber',False,'Eletrônico','4',null),
                              ('Sofá retrátil',False,'Móveis','3','3x57x80'),
                              ('Farinha de arroz',False,'Alimentos','2',null),
                              ('Fire Stick Amazon',False,'Eletrônico','3',null);


-- idOrder, idOrderClient, orderStatus, orderDescription, sendValue, paymentCash

delete from orders where idOrderClient in  (1,2,3,4);
insert into orders (idOrderClient, orderStatus, orderDescription, sendValue, paymentCash) values
							 (1, default,'compra via aplicativo',null,1),
                             (2,default,'compra via aplicativo',50,0),
                             (3,'Confirmado',null,null,1),
                             (4,default,'compra via web site',150,0);

-- idPOproduct, idPOorder, poQuantity, poStatus

insert into productOrder (idPOproduct, idPOorder, poQuantity, poStatus) values
						 (1,5,2,null),
                         (2,5,1,null),
                         (3,6,1,null);

-- storageLocation,quantity
insert into productStorage (storageLocation,quantity) values
							('Rio de Janeiro',1000),
                            ('Rio de Janeiro',500),
                            ('São Paulo',10),
                            ('São Paulo',100),
                            ('São Paulo',10),
                            ('Brasília',60);

-- idLproduct, idLstorage, location
insert into storageLocation (idLproduct, idLstorage, location) values
						 (1,2,'RJ'),
                         (2,6,'GO');

-- idSupplier, SocialName, CNPJ, contact
insert into supplier (SocialName, CNPJ, contact) values
							('Almeida e filhos', 123456789123456,'21985474'),
                            ('Eletrônicos Silva',854519649143457,'21985484'),
                            ('Eletrônicos Valma', 934567893934695,'21975474');


-- idPsSupplier, idPsProduct, quantity
insert into productSupplier (idPsSupplier, idPsProduct, quantity) values
						 (1,1,500),
                         (1,2,400),
                         (2,4,633),
                         (3,3,5),
                         (2,5,10);

-- idSeller, SocialName, AbstName, CNPJ, CPF, location, contact
insert into seller (SocialName, AbstName, CNPJ, CPF, location, contact) values
						('Tech eletronics', null, 123456789456321, null, 'Rio de Janeiro', 219946287),
					    ('Botique Durgas',null,null,123456783,'Rio de Janeiro', 219567895),
						('Kids World',null,456789123654485,null,'São Paulo', 1198657484);


-- idPseller, idPproduct, prodQuantity
insert into productSeller (idPseller, idPproduct, prodQuantity) values
						 (1,6,80),
                         (2,7,10);


insert into orders (idOrderClient, orderStatus, orderDescription, sendValue, paymentCash) values
							 (2, default,'compra via aplicativo',null,1);
```

## 4. Realizando Consultas ao Banco de Dados

Agora vamos realizar algumas consultas para responder algumas perguntas.

### Quais clientes possuem pedidos?

```SQL
select Fname,Lname, idOrder, orderStatus from clients c, orders o where c.idClient = idOrderClient;
```

![Clientes x pedidos](https://github.com/devcaiada/db-logic-project/blob/main/assets/Clientes%20x%20pedidos.png?raw=true)

Utilizando o Alias para personalizar a consulta:

```SQL
select concat(Fname,' ',Lname) as Client, idOrder as Request, orderStatus as Status from clients c, orders o where c.idClient = idOrderClient;
```

![Cliente x pedido alias](https://github.com/devcaiada/db-logic-project/blob/main/assets/Cliente%20x%20pedido%20alias.png?raw=true)

### Quantos clientes efetuaram pedidos?

```SQL
select count(*) from clients c, orders o
			where c.idClient = idOrderClient;
```

**4**

### Quantos pedidos cada cliente efetuou?

```SQL
select c.idClient, Fname, count(*) as Number_of_orders from clients c
				inner join orders o ON c.idClient = o.idOrderClient
		group by idClient;
```

## ![Number of orders](https://github.com/devcaiada/db-logic-project/blob/main/assets/Number%20of%20orders.png?raw=true)

## 5. Criando índices para otimizar as consultas

### Índice Clusterizado (na tabela orders)

Criação do índice clusterizado na coluna idOrderClient.

```SQL
ALTER TABLE orders
ADD INDEX idx_order_client (idOrderClient);
```

### Índice Não-clusterizado (na tabela productStorage)

Criação do índice não-clusterizado na coluna storageLocation.

```SQL
CREATE INDEX idx_storage_location ON productStorage (storageLocation);
```

### Índice em Coluna de Chave Estrangeira (na tabela productOrder)

Criação do índice na coluna idPOproduct (chave estrangeira).

```SQL
ALTER TABLE productOrder
ADD INDEX idx_product_order (idPOproduct);
```

### Índice em Coluna de Busca e Filtro (na tabela product)

Criação do índice na coluna Pname.

```SQL
CREATE INDEX idx_product_name ON product (Pname);
```

## 6. Criando procedure para inserção de dados

Vamos criar uma procedure para inserir dados na tabela **clients** do nosso banco de dados com uma verificação no tamanho do **CPF**.

```SQL
delimiter \\
create procedure procedure_client_insert(
	in Fname_p VARCHAR(10),
	in Minit_p VARCHAR(3),
	in Lname_p VARCHAR(20),
	in CPF_p CHAR(11),
	in Address_p VARCHAR(255),
	in ClientType_p ENUM('PF', 'PJ')
)
begin
	if (length(CPF_p) = 11) then
		insert into clients (Fname, Minit, Lname, CPF, Address, ClientType) values(Fname_p, Minit_p, Lname_p, CPF_p, Address_p, ClientType_p);
	else
		select 'Forneça um CPF válido.' as Message_error;
	end if;

end \\
delimiter ;
```

Com a procedure criada podemos inserir os dados sem utilizar a função **INSERT** do SQL. Nesse caso chamaríamos a procedure e passaríamos os dados da seguinte forma:

```SQL
call procedure_client_insert('Caio', 'P', 'Arruda', '12345678910', 'Rua das dores, 42', 'PF');
```

Podemos criar algo mais avançado com as procedures. No caso vamos criar uma função que recebe um valor inteiro, e para cada valor de 1 a 3, ele retorna uma ação.

```SQL
DELIMITER //
CREATE PROCEDURE ManageClients(IN action INT)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE client_id INT;
    DECLARE first_name VARCHAR(255);
    DECLARE last_name VARCHAR(255);
    DECLARE cpf CHAR(11);
    DECLARE client_type ENUM('PJ', 'PF');

    DECLARE cur CURSOR FOR
        SELECT idClient, Fname, Lname, CPF, ClientType
        FROM clients;

    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    IF action = 1 THEN
        -- Selecionar registros
        OPEN cur;
        read_loop: LOOP
            FETCH cur INTO client_id, first_name, last_name, cpf, client_type;
            IF done THEN
                LEAVE read_loop;
            END IF;
                SELECT idClient, Fname, Lname FROM clients WHERE idClient = client_id;
        END LOOP;
        CLOSE cur;
    ELSEIF action = 2 THEN
        -- Atualizar registros
            UPDATE clients SET Address = 'NovoEndereco' WHERE idClient = 1;
    ELSEIF action = 3 THEN
        -- Excluir registros
            DELETE FROM clients WHERE idClient = 1;
    ELSE
        -- Ação inválida
        SELECT 'Ação inválida. Escolha 1 para SELECT, 2 para UPDATE ou 3 para DELETE.';
    END IF;
END //
DELIMITER ;
```

Na opção 1 (Selecionar clientes), criamos um looping onde ele trará uma consulta para cada cliente cadastrado na tabela **clients**.

---

## 7. Criando views no banco de dados

Vamos criar algumas views para responder perguntas padrões.

### View de Clientes com Nome Completo

```SQL
CREATE VIEW view_clients AS
    SELECT idClient, CONCAT(Fname, ' ', Minit, '.', ' ', Lname) AS Nome, CPF, Address as Endereco, ClientType as Tipo
    FROM clients;
```

### View de Produtos Disponíveis

```SQL
CREATE VIEW view_available_products AS
    SELECT idProduct, Pname, category, avaliação, size
    FROM product
    WHERE classification_kids = false;
```

### View de Pedidos Confirmados

```SQL
CREATE VIEW view_confirmed_orders AS
    SELECT idOrder, orderDescription, sendValue
    FROM orders
    WHERE orderStatus = 'Confirmado';
```

### View de Fornecedores com Nome e CNPJ

```SQL
CREATE VIEW view_suppliers AS
    SELECT idSupplier, SocialName, CNPJ
    FROM supplier;
```

### View de Vendedores com Nome e CPF

```SQL
CREATE VIEW vw_sellers AS
    SELECT idSeller, SocialName, CPF
    FROM seller;
```

## Criando um usuário para acessar nossas views

Vamos criar um novo usuário para o nosso banco de dados e conceder acesso total para a nossa view **view_clients** conforme exemplo abaixo:

### Criando o usuário

```SQL
CREATE USER 'caio'@'localhost' IDENTIFIED BY '123789';
```

### Concedendo os privilégios

```SQL
GRANT ALL PRIVILEGES ON ecommerce.view_clients TO 'caio'@'localhost';
```

## Dica

Sempre que você fizer alterações nas permissões, recarregue os privilégios para que as alterações entrem em vigor:

```SQL
FLUSH PRIVILEGES;
```

## 8. Criação de triggers

Imagine que por algum motivo podemos excluir cadastros da tabela cliente. Como esse é um dado muito importante, é necessário criarmos um trigger para não perdermos essas informações conforme exemplo abaixo:

```SQL
DELIMITER //
CREATE TRIGGER trigger_before_delete_client
BEFORE DELETE ON clients
FOR EACH ROW
BEGIN
    INSERT INTO client_history (idClient, Fname, Minit, Lname, CPF, Address, ClientType)
    VALUES (OLD.idClient, OLD.Fname, OLD.Minit, OLD.Lname, OLD.CPF, OLD.Address, OLD.ClientType);
END;
//
DELIMITER ;
```

Neste exemplo, assumimos que temos uma tabela **client_history** com os campos (idClient, Fname, Minit, Lname, CPF, Address, ClientType) para armazenarmos os dados de clientes excluidos.

Ainda assumindo que temos a tabela **client_history**, podemos criar um trigger para o update de clientes. Uma vez que haja alteração do CPF, armazenamos as informações do antigo cliente.

```SQL
DELIMITER //
CREATE TRIGGER trg_before_update_client
BEFORE UPDATE ON clients
FOR EACH ROW
BEGIN
    IF NEW.CPF <> OLD.CPF THEN
        INSERT INTO client_history (idClient, Fname, Minit, Lname, CPF, Address, ClientType)
        VALUES (OLD.idClient, OLD.Fname, OLD.Minit, OLD.Lname, OLD.CPF, OLD.Address, OLD.ClientType);
    END IF;
END;
//
DELIMITER ;
```

Caso o novo CPF seja diferente do anterior, então é um novo cliente, podendo corromper as informações do banco de dados no update.

---

<br>
</br>

## 9. Criação de Transações com Rollback e Procedure

Vamos criar um exemplo de Stored Procedure que realiza uma inserção de um novo cliente, um novo pedido e a atualização do estoque de produtos com um comando Rollback caso a transação não seja concluída ou haja erros.

Primeiro de tudo devemos desabilitar o autocommit no MySQL:

```SQL
-- Desabilitar o autocommit para a sessão atual
SET autocommit = 0;
```

Na sequência vamos começar nossa procedure e logo em seguida nossa transação:

```SQL
DELIMITER //

CREATE PROCEDURE CreateOrder(
    IN p_Fname VARCHAR(10),
    IN p_Minit CHAR(3),
    IN p_Lname VARCHAR(20),
    IN p_CPF CHAR(11),
    IN p_Address VARCHAR(255),
    IN p_ClientType ENUM('PJ', 'PF'),
    IN p_orderDescription VARCHAR(255),
    IN p_sendValue FLOAT,
    IN p_paymentCash BOOLEAN,
    IN p_productId INT,
    IN p_quantity INT
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        -- Rollback da trnsação caso haja erro
        ROLLBACK;
    END;

    -- Inicio da transação
    START TRANSACTION;

    -- Inserção de um novo cliente
    INSERT INTO clients (Fname, Minit, Lname, CPF, Address, ClientType)
    VALUES (p_Fname, p_Minit, p_Lname, p_CPF, p_Address, p_ClientType);

    -- Pegar o último ID inserido
    SET @lastClientId = LAST_INSERT_ID();

    -- Inserir uma nova ordem
    INSERT INTO orders (idOrderClient, orderDescription, sendValue, paymentCash)
    VALUES (@lastClientId, p_orderDescription, p_sendValue, p_paymentCash);

    -- Pegar o último ID inserido em orders
    SET @lastOrderId = LAST_INSERT_ID();

    -- Inserir o produto em order
    INSERT INTO productOrder (idPOproduct, idPOorder, poQuantity)
    VALUES (p_productId, @lastOrderId, p_quantity);

    -- Atualizar a quantidade do produto
    UPDATE productStorage
    SET quantity = quantity - p_quantity
    WHERE idProdStorage = p_productId;

    -- Commitar a transação
    COMMIT;
END //

DELIMITER ;
```

Este procedimento armazenado realiza as seguintes operações dentro de uma transação:

1. Insere um novo cliente na tabela clients;
2. Insere um novo pedido na tabela orders associado ao cliente recém-criado;
3. Insere o produto no pedido na tabela productOrder;
4. Atualiza a quantidade do produto no estoque na tabela productStorage.

Se ocorrer algum erro durante a execução, a transação será revertida (rollback) para garantir a integridade dos dados.

### Utilizando Rollback parcial (SAVEPOINT)

```SQL
DELIMITER //

CREATE PROCEDURE CreateOrder(
    IN p_Fname VARCHAR(10),
    IN p_Minit CHAR(3),
    IN p_Lname VARCHAR(20),
    IN p_CPF CHAR(11),
    IN p_Address VARCHAR(255),
    IN p_ClientType ENUM('PJ', 'PF'),
    IN p_orderDescription VARCHAR(255),
    IN p_sendValue FLOAT,
    IN p_paymentCash BOOLEAN,
    IN p_productId INT,
    IN p_quantity INT
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        -- Rollback da transação em caso de erros
        ROLLBACK;
    END;

    -- Start da transação
    START TRANSACTION;

    -- Savepoint depois do start da transação
    SAVEPOINT sp_start;

    -- Inserir um novo cliente
    INSERT INTO clients (Fname, Minit, Lname, CPF, Address, ClientType)
    VALUES (p_Fname, p_Minit, p_Lname, p_CPF, p_Address, p_ClientType);

    -- Savepoint depois de inserir um cliente
    SAVEPOINT sp_after_client;

    -- Pegar o ID do último cliente inserido
    SET @lastClientId = LAST_INSERT_ID();

    -- Inserir um novo pedido
    INSERT INTO orders (idOrderClient, orderDescription, sendValue, paymentCash)
    VALUES (@lastClientId, p_orderDescription, p_sendValue, p_paymentCash);

    -- Savepoint depois do pedido
    SAVEPOINT sp_after_order;

    -- Pegar o ID do último pedido
    SET @lastOrderId = LAST_INSERT_ID();

    -- Inserção do produto no pedido
    INSERT INTO productOrder (idPOproduct, idPOorder, poQuantity)
    VALUES (p_productId, @lastOrderId, p_quantity);

    -- Savepoint depois de inserir o produto
    SAVEPOINT sp_after_product_order;

    -- Atualização da quantidade do produto no estoque
    UPDATE productStorage
    SET quantity = quantity - p_quantity
    WHERE idProdStorage = p_productId;

    -- Savepoint depois de atualizar o estoque
    SAVEPOINT sp_after_storage_update;

    -- Commit da transação
    COMMIT;
END //

DELIMITER ;
```

Neste exemplo, adicionei SAVEPOINTs após cada operação crítica. Se ocorrer um erro, você pode fazer rollback para qualquer um desses pontos de salvamento específicos, se necessário. Isso proporciona um controle mais granular sobre a transação.

<br>
</br>

## 10. Executando Backup e Recovery de um Banco de Dados com MySQLDump

Vamos executar um comando de backup de banco de dados utilizando o MySQLDump. Essa ferramenta é acessível através do CMD do Windows.

O CMD DEVE SER EXECUTADO COMO ADMINISTRADOR PARA FUNCIONAR.

Primeiro de tudo devemos acessar a pasta do MySQLDump:

```
cd c:\program files\mysql\mysql server 8.0\bin\
```

Após acessar a pasta da nossa ferramenta, vamos executar o seguinte comando para gerar um backup do nosso banco **ecommerce**.

```
mysqldump --user root --password --databases ecommerce > ecommerce_backup.sql
```

A partir desse comando ela irá gerar um arquivo SQL com todas as sintaxes do banco escolhido. Deixei o arquivo neste repositório, na pasta **backup** como exemplo.

- Trecho do arquivo salvo:

```SQL
-- MySQL dump 10.13  Distrib 8.0.35, for Win64 (x86_64)
--
-- Host: localhost    Database: ecommerce
-- ------------------------------------------------------
-- Server version	8.0.35

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!50503 SET NAMES utf8mb4 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Current Database: `ecommerce`
--

CREATE DATABASE /*!32312 IF NOT EXISTS*/ `ecommerce` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci */ /*!80016 DEFAULT ENCRYPTION='N' */;

USE `ecommerce`;

--
-- Table structure for table `clients`
--

DROP TABLE IF EXISTS `clients`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `clients` (
  `idClient` int NOT NULL AUTO_INCREMENT,
  `Fname` varchar(10) DEFAULT NULL,
  `Minit` char(3) DEFAULT NULL,
  `Lname` varchar(20) DEFAULT NULL,
  `CPF` char(11) NOT NULL,
  `Address` varchar(255) DEFAULT NULL,
  `ClientType` enum('PJ','PF') NOT NULL,
  PRIMARY KEY (`idClient`),
  UNIQUE KEY `unique_cpf_client` (`CPF`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;
...
```

<br></br>

### Restauração

Para restaurarmos um banco de dados a partir do backup que geramos, basta utilizar o seguinte comando no CMD:

```
mysql -u root -p mydatabase < mydatabase_backup.sql
```

<br></br>

### Criando um backup com routines, eventos e triggers

Agora vamos criar um backup de nosso banco de dados de uma forma mais completa, adicionando routines (procedures), eventos e triggers. Abaixo o código utilizado no CMD do Windows:

```
mysqldump --routines --triggers --events -u root -p ecommerce > ecommerce_routines_bkp.sql
```

Lembrando que para executar esse código é necessário estar dentro da seguinte pasta no CMD:

```
c:\program files\mysql\mysql server 8.0\bin\
```

Vou deixar o arquivo gerado **_ecommerce_routines_bkp_** na pasta backup desse repositório.

<br></br>

### EXTRA

- Compressão do backup:

```
mysqldump -u root -p mydatabase | gzip > mydatabase_backup.sql.gz
```

- Restauração de um backup comprimido:

```
gunzip < mydatabase_backup.sql.gz | mysql -u root -p mydatabase
```
