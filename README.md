# Atividade de Programação AWS - Semana 6 | 2025

Estudante: Nataly de Souza Cunha | T13 | G01

Professor: <a href="https://www.linkedin.com/in/jefferson-o-silva/">Jefferson de Oliveira Silva</a> 

## 🎯 Introdução

&ensp;Esta atividade pretende concretizar o aprendizado de Cloud Computing através da criação uma aplicação em nuvem através do provedor  Amazon Web Services (AWS), utilizando instâncias EC2 para hospedar a aplicação e RDS como o banco de dados. Abaixo, segue o código desenvolvido.

&ensp;[Link de demonstração]()

&ensp;[Link da aplicação em deploy](http://ec2-54-209-7-130.compute-1.amazonaws.com/SamplePage.php)

## ⭐ Estrutura do projeto em nuvem

```
instância EC2/
│
├── var/www/        
│
├────── inc/         
│
├────────── dbinfo.inc/       # Arquivo de conexão com a instância RDS
│
├────── html/            
│
├────────── SamplePage.php       # Página com back-end e front-end

```

### Página SamplePage.php

&ensp;A seguir, tem-se a aplicação integrada a um banco de dados. Ela realiza operações com o banco RDS através de comandos SQL, possui um formulário de cadastro de clientes, um cadastro de tatuagens e, em seguida, uma listagem de todos os registros. Foi-se implementado um layout e design (front-end) básico para uma visualização mais limpa e agradável.

```php
<?php include "../inc/dbinfo.inc"; ?>

<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Estúdio de Tatuagem</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f4f4f4;
      margin: 0;
      padding: 0;
    }
    h1 {
      text-align: center;
      color: #333;
      margin-top: 20px;
    }
    .container {
      width: 80%;
      margin: 0 auto;
      background-color: #fff;
      padding: 20px;
      box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
      border-radius: 8px;
    }
    form {
      margin-bottom: 20px;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-bottom: 20px;
    }
    table, th, td {
      border: 1px solid #ddd;
    }
    th, td {
      padding: 10px;
      text-align: left;
    }
    th {
      background-color: #f8f8f8;
    }
    input[type="text"], input[type="date"], input[type="number"] {
      width: 100%;
      padding: 8px;
      margin: 5px 0;
      box-sizing: border-box;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
    input[type="submit"] {
      background-color: #4CAF50;
      color: white;
      padding: 10px 20px;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    input[type="submit"]:hover {
      background-color: #45a049;
    }
    .form-section {
      margin-bottom: 30px;
    }
    .form-section h2 {
      color: #555;
      margin-bottom: 10px;
    }
  </style>
</head>
<body>
  <h1>Estúdio de Tatuagem</h1>
  <div class="container">
    <?php
    /* Conectar ao PostgreSQL e selecionar o banco de dados. */
    $constring = "host=" . DB_SERVER . " dbname=" . DB_DATABASE . " user=" . DB_USERNAME . " password=" . DB_PASSWORD ;
    $connection = pg_connect($constring);

    if (!$connection){
      echo "Falha ao conectar ao PostgreSQL";
      exit;
    }

    /* Verificar se as tabelas CLIENTES e TATUAGENS existem. */
    VerificarTabelaClientes($connection, DB_DATABASE);
    VerificarTabelaTatuagens($connection, DB_DATABASE);

    /* Se os campos do formulário estiverem preenchidos, adiciona uma linha à tabela CLIENTES. */
    $nome_cliente = htmlentities($_POST['NOME_CLIENTE']);
    $telefone_cliente = htmlentities($_POST['TELEFONE']);

    if (strlen($nome_cliente) || strlen($telefone_cliente)) {
      AdicionarCliente($connection, $nome_cliente, $telefone_cliente);
    }

    /* Se os campos do formulário estiverem preenchidos, adiciona uma linha à tabela TATUAGENS. */
    $descricao_tatuagem = htmlentities($_POST['DESCRICAO']);
    $data_tatuagem = htmlentities($_POST['DATA_TATUAGEM']);
    $preco_tatuagem = htmlentities($_POST['PRECO']);
    $finalizada = isset($_POST['FINALIZADA']) ? 'TRUE' : 'FALSE';

    if (strlen($descricao_tatuagem) || strlen($data_tatuagem) || strlen($preco_tatuagem)) {
      AdicionarTatuagem($connection, $descricao_tatuagem, $data_tatuagem, $preco_tatuagem, $finalizada);
    }
    ?>

    <!-- Formulário para CLIENTES -->
    <div class="form-section">
      <h2>Adicionar Cliente</h2>
      <form action="<?PHP echo $_SERVER['SCRIPT_NAME'] ?>" method="POST">
        <table>
          <tr>
            <td>Nome do Cliente:</td>
            <td><input type="text" name="NOME_CLIENTE" maxlength="45" required /></td>
          </tr>
          <tr>
            <td>Telefone:</td>
            <td><input type="text" name="TELEFONE" maxlength="15" required /></td>
          </tr>
          <tr>
            <td colspan="2"><input type="submit" value="Adicionar Cliente" /></td>
          </tr>
        </table>
      </form>
    </div>

    <!-- Formulário para TATUAGENS -->
    <div class="form-section">
      <h2>Adicionar Tatuagem</h2>
      <form action="<?PHP echo $_SERVER['SCRIPT_NAME'] ?>" method="POST">
        <table>
          <tr>
            <td>Descrição:</td>
            <td><input type="text" name="DESCRICAO" maxlength="100" required /></td>
          </tr>
          <tr>
            <td>Data da Tatuagem:</td>
            <td><input type="date" name="DATA_TATUAGEM" required /></td>
          </tr>
          <tr>
            <td>Preço:</td>
            <td><input type="number" name="PRECO" step="0.01" required /></td>
          </tr>
          <tr>
            <td>Finalizada:</td>
            <td><input type="checkbox" name="FINALIZADA" /></td>
          </tr>
          <tr>
            <td colspan="2"><input type="submit" value="Adicionar Tatuagem" /></td>
          </tr>
        </table>
      </form>
    </div>

    <!-- Exibir dados da tabela CLIENTES -->
    <h2>Clientes Cadastrados</h2>
    <table>
      <tr>
        <th>ID</th>
        <th>Nome</th>
        <th>Telefone</th>
      </tr>
      <?php
      $resultado = pg_query($connection, "SELECT * FROM CLIENTES");
      while($dados = pg_fetch_row($resultado)) {
        echo "<tr>";
        echo "<td>",$dados[0], "</td>",
             "<td>",$dados[1], "</td>",
             "<td>",$dados[2], "</td>";
        echo "</tr>";
      }
      ?>
    </table>

    <!-- Exibir dados da tabela TATUAGENS -->
    <h2>Tatuagens Cadastradas</h2>
    <table>
      <tr>
        <th>ID</th>
        <th>Descrição</th>
        <th>Data</th>
        <th>Preço</th>
        <th>Finalizada</th>
      </tr>
      <?php
      $resultado = pg_query($connection, "SELECT * FROM TATUAGENS");
      while($dados = pg_fetch_row($resultado)) {
        echo "<tr>";
        echo "<td>",$dados[0], "</td>",
             "<td>",$dados[1], "</td>",
             "<td>",$dados[2], "</td>",
             "<td>",$dados[3], "</td>",
             "<td>",($dados[4] === 't' ? 'Sim' : 'Não'), "</td>";
        echo "</tr>";
      }
      ?>
    </table>

    <?php
    /* Limpar recursos. */
    pg_free_result($resultado);
    pg_close($connection);
    ?>
  </div>
</body>
</html>

<?php
/* Funções PHP (back-end) */
function AdicionarCliente($connection, $nome, $telefone) {
  $n = pg_escape_string($nome);
  $t = pg_escape_string($telefone);
  $query = "INSERT INTO CLIENTES (NOME, TELEFONE) VALUES ('$n', '$t');";
  if(!pg_query($connection, $query)) echo("<p>Erro ao adicionar dados do cliente.</p>");
}

function AdicionarTatuagem($connection, $descricao, $data, $preco, $finalizada) {
  $d = pg_escape_string($descricao);
  $dt = pg_escape_string($data);
  $p = pg_escape_string($preco);
  $f = pg_escape_string($finalizada);
  $query = "INSERT INTO TATUAGENS (DESCRICAO, DATA_TATUAGEM, PRECO, FINALIZADA) VALUES ('$d', '$dt', $p, $f);";
  if(!pg_query($connection, $query)) echo("<p>Erro ao adicionar dados da tatuagem.</p>");
}

function VerificarTabelaClientes($connection, $dbName) {
  if(!TabelaExiste("CLIENTES", $connection, $dbName)) {
    $query = "CREATE TABLE CLIENTES (
        ID serial PRIMARY KEY,
        NOME VARCHAR(45),
        TELEFONE VARCHAR(15)
      )";
    if(!pg_query($connection, $query)) echo("<p>Erro ao criar a tabela CLIENTES.</p>");
  }
}

function VerificarTabelaTatuagens($connection, $dbName) {
  if(!TabelaExiste("TATUAGENS", $connection, $dbName)) {
    $query = "CREATE TABLE TATUAGENS (
        ID serial PRIMARY KEY,
        DESCRICAO VARCHAR(100),
        DATA_TATUAGEM DATE,
        PRECO NUMERIC(10, 2),
        FINALIZADA BOOLEAN
      )";
    if(!pg_query($connection, $query)) echo("<p>Erro ao criar a tabela TATUAGENS.</p>");
  }
}

function TabelaExiste($nomeTabela, $connection, $dbName) {
  $t = strtolower(pg_escape_string($nomeTabela));
  $d = pg_escape_string($dbName);
  $query = "SELECT TABLE_NAME FROM information_schema.TABLES WHERE TABLE_NAME = '$t';";
  $verificarTabela = pg_query($connection, $query);
  if (pg_num_rows($verificarTabela) > 0) return true;
  return false;
}
?>
```

&ensp;Através da realização dessa atividade, foram reforçados os conhecimentos sobre a configuração de um ambiente em nuvem através da AWS, o entendimento e implementação de uma arquitetura em nuvem, os papéis de diferentes instâncias integradas em um sistema, bem como foram desenvolvidas habilidades relacionadas a programação back-end, front-end, integrações, deploy e banco de dados.
