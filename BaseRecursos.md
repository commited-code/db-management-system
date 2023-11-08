# Modelo de Banco de Dados: Boas Práticas e Segurança

Este artigo descreve um modelo de banco de dados que segue as boas práticas de administração de banco de dados (DBA) e enfatiza a segurança dos dados. O modelo inclui uma estrutura de dados para um sistema que gerencia informações de usuários, clientes e auditoria, com ênfase em relacionamentos eficazes e segurança de senhas.

## Modelo Conceitual

O modelo conceitual define as entidades e seus relacionamentos:

- **Entidade "Usuários":**
  - Atributos: 
    - ID (Chave Primária)
    - Nome
    - Email (único)
    - Senha (armazenada com hash e salt)
    - Data de Criação
    - Data de Atualização
    - Data de Remoção (para exclusões lógicas)
  - Relacionamentos: Funções (muitos para muitos), Consumo de Serviço (um para muitos).

- **Entidade "Clientes":**
  - Atributos: 
    - ID (Chave Primária)
    - Nome
    - Email
    - Telefone
    - Endereço

- **Entidade "Auditoria":**
  - Atributos: 
    - ID (Chave Primária)
    - ID do Usuário (Chave Estrangeira)
    - Data de Acesso
    - Serviço Acessado
    - Total de Acessos
    - Total de Serviços Consumidos
    - Tempo de Conexão (em segundos)

- **Entidade "Funções":**
  - Atributos: 
    - ID (Chave Primária)
    - Nome
    - Descrição

- **Entidade "Permissões":**
  - Atributos: 
    - ID (Chave Primária)
    - Nome
    - Descrição

- **Tabela de Associação entre Funções e Permissões (muitos para muitos):**
  - Tabela de associação entre Funções e Permissões, para definir quais permissões cada função possui.

## Modelo Físico

O modelo físico representa a implementação real do banco de dados com ênfase em segurança e relacionamentos:

```sql
-- Tabela de Usuários
CREATE TABLE Usuarios (
    ID INT PRIMARY KEY,
    Nome NVARCHAR(100),
    Email NVARCHAR(255) UNIQUE,
    SenhaHash VARBINARY(MAX),
    Salt VARBINARY(MAX),
    Data_Criacao DATETIME,
    Data_Atualizacao DATETIME,
    Data_Remocao DATETIME
);

-- Tabela de Clientes
CREATE TABLE Clientes (
    ID INT PRIMARY KEY,
    Nome NVARCHAR(100),
    Email NVARCHAR(255),
    Telefone NVARCHAR(20),
    Endereco NVARCHAR(255)
);

-- Tabela de Auditoria
CREATE TABLE Auditoria (
    ID INT PRIMARY KEY,
    UsuarioID INT,
    Data_Acesso DATETIME,
    Servico_Acessado NVARCHAR(100),
    Total_Acessos INT,
    Servicos_Consumidos INT,
    Tempo_Conexao INT -- Tempo em segundos
);

-- Tabela de Funções
CREATE TABLE Funcoes (
    ID INT PRIMARY KEY,
    Nome NVARCHAR(50),
    Descricao NVARCHAR(255)
);

-- Tabela de Permissões
CREATE TABLE Permissoes (
    ID INT PRIMARY KEY,
    Nome NVARCHAR(50),
    Descricao NVARCHAR(255)
);

-- Tabela de Associação entre Funções e Permissões (muitos para muitos)
CREATE TABLE Funcoes_Permissoes (
    FuncaoID INT,
    PermissaoID INT,
    PRIMARY KEY (FuncaoID, PermissaoID),
    FOREIGN KEY (FuncaoID) REFERENCES Funcoes(ID),
    FOREIGN KEY (PermissaoID) REFERENCES Permissoes(ID)
);

-- Chave Estrangeira para relacionar a tabela de Auditoria com a tabela de Usuários
ALTER TABLE Auditoria
ADD FOREIGN KEY (UsuarioID) REFERENCES Usuarios(ID);
```

## Subquery para Total de Tempo Logado e Consumo de Serviço

Uma subconsulta pode ser usada para calcular o tempo total logado e o consumo de serviço para cada usuário. Aqui está uma representação simplificada:

```sql
SELECT
    U.ID,
    U.Nome,
    (SELECT SUM(Tempo_Conexao) 
     FROM Auditoria A 
     WHERE A.UsuarioID = U.ID) AS Tempo_Logado,
    (SELECT SUM(Servicos_Consumidos) 
     FROM Auditoria A 
     WHERE A.UsuarioID = U.ID) AS Consumo_de_Servico
FROM Usuarios U;
```
