# Cad_Saude[Uploading index.php…]()<?php
// Conexão com o banco de dados
$host = "localhost";
$user = "root";
$pass = "";
$db = "cad";
$conn = new mysqli($host, $user, $pass, $db);

if ($conn->connect_error) {
    die("Erro de conexão: " . $conn->connect_error);
}

// Processamento dos formulários
if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    if (isset($_POST['action'])) {
        $action = $_POST['action'];
        $id = $_POST['id'] ?? null;
        $nome = trim($_POST['nome']);
        $quantidade = (int)$_POST['quantidade'];
        $validade = $_POST['validade'];
        $lote = trim($_POST['lote']);

        if ($action == 'add') {
            $stmt = $conn->prepare("INSERT INTO medicamentos (nome, quantidade, validade, lote) VALUES (?, ?, ?, ?)");
            $stmt->bind_param("siss", $nome, $quantidade, $validade, $lote);
            $msg = $stmt->execute() ? ["success" => "Medicamento cadastrado!"] : ["error" => "Erro ao cadastrar"];
        } 
        elseif ($action == 'edit' && $id) {
            $stmt = $conn->prepare("UPDATE medicamentos SET nome=?, quantidade=?, validade=?, lote=? WHERE id=?");
            $stmt->bind_param("sissi", $nome, $quantidade, $validade, $lote, $id);
            $msg = $stmt->execute() ? ["success" => "Medicamento atualizado!"] : ["error" => "Erro ao atualizar"];
        }
        $stmt->close();
    }
}

// Processamento de exclusão
if (isset($_GET['delete'])) {
    $id = (int)$_GET['delete'];
    $stmt = $conn->prepare("DELETE FROM medicamentos WHERE id=?");
    $stmt->bind_param("i", $id);
    $msg = $stmt->execute() ? ["success" => "Medicamento excluído!"] : ["error" => "Erro ao excluir"];
    $stmt->close();
}

// Consulta dos medicamentos
$search = $_GET['search'] ?? '';
$sql = "SELECT * FROM medicamentos WHERE nome LIKE ? ORDER BY nome";
$stmt = $conn->prepare($sql);
$searchParam = "%$search%";
$stmt->bind_param("s", $searchParam);
$stmt->execute();
$result = $stmt->get_result();

// Verifica edição
$editing = false;
$editData = null;
if (isset($_GET['edit'])) {
    $id = (int)$_GET['edit'];
    $stmt = $conn->prepare("SELECT * FROM medicamentos WHERE id=?");
    $stmt->bind_param("i", $id);
    $stmt->execute();
    $editData = $stmt->get_result()->fetch_assoc();
    $stmt->close();
    $editing = (bool)$editData;
}
?>
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestão de Medicamentos</title>
    <style>
        :root {
            --primary: #4361ee;
            --secondary: #3a0ca3;
            --light: #f8f9fa;
            --dark: #212529;
            --success: #4cc9f0;
            --danger: #f72585;
            --warning: #f8961e;
            --info: #4895ef;
            --text: #2b2d42;
            --bg: #f8f9fa;
            --card-bg: #ffffff;
            --border: #dee2e6;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
        }

        body {
            background-color: var(--bg);
            color: var(--text);
            font-size: 0.875rem;
            line-height: 1.5;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 1rem;
        }

        header {
            background: linear-gradient(135deg, var(--primary), var(--secondary));
            color: white;
            padding: 1rem 0;
            margin-bottom: 1.5rem;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }

        header h1 {
            font-size: 1.5rem;
            font-weight: 600;
            text-align: center;
        }

        .card {
            background: var(--card-bg);
            border-radius: 0.5rem;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
            padding: 1.25rem;
            margin-bottom: 1.5rem;
        }

        .card h2 {
            color: var(--primary);
            font-size: 1.125rem;
            font-weight: 600;
            margin-bottom: 1rem;
            padding-bottom: 0.5rem;
            border-bottom: 1px solid var(--border);
        }

        .form-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 1rem;
            margin-bottom: 1rem;
        }

        .form-group {
            margin-bottom: 0.75rem;
        }

        label {
            display: block;
            margin-bottom: 0.25rem;
            font-weight: 500;
            color: var(--dark);
            font-size: 0.8125rem;
        }

        input, select {
            width: 100%;
            padding: 0.5rem 0.75rem;
            border: 1px solid var(--border);
            border-radius: 0.375rem;
            font-size: 0.8125rem;
            transition: all 0.2s;
        }

        input:focus, select:focus {
            border-color: var(--primary);
            outline: none;
            box-shadow: 0 0 0 2px rgba(67, 97, 238, 0.2);
        }

        .btn {
            display: inline-flex;
            align-items: center;
            justify-content: center;
            padding: 0.5rem 1rem;
            border-radius: 0.375rem;
            font-size: 0.8125rem;
            font-weight: 500;
            cursor: pointer;
            transition: all 0.2s;
            border: none;
            text-decoration: none;
        }

        .btn-primary {
            background-color: var(--primary);
            color: white;
        }

        .btn-primary:hover {
            background-color: var(--secondary);
        }

        .btn-success {
            background-color: var(--success);
            color: var(--dark);
        }

        .btn-danger {
            background-color: var(--danger);
            color: white;
        }

        .btn-warning {
            background-color: var(--warning);
            color: white;
        }

        .btn-sm {
            padding: 0.375rem 0.75rem;
            font-size: 0.75rem;
        }

        .alert {
            padding: 0.75rem;
            border-radius: 0.375rem;
            margin-bottom: 1rem;
            font-size: 0.8125rem;
        }

        .alert-success {
            background-color: rgba(76, 201, 240, 0.1);
            color: var(--success);
            border: 1px solid rgba(76, 201, 240, 0.3);
        }

        .alert-danger {
            background-color: rgba(247, 37, 133, 0.1);
            color: var(--danger);
            border: 1px solid rgba(247, 37, 133, 0.3);
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 1rem;
            font-size: 0.8125rem;
        }

        th, td {
            padding: 0.75rem;
            text-align: left;
            border-bottom: 1px solid var(--border);
        }

        th {
            background-color: var(--primary);
            color: white;
            font-weight: 500;
            font-size: 0.75rem;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }

        tr:hover {
            background-color: rgba(67, 97, 238, 0.03);
        }

        .actions {
            display: flex;
            gap: 0.5rem;
        }

        .search-container {
            display: flex;
            gap: 0.5rem;
            margin-bottom: 1rem;
        }

        .search-container input {
            flex: 1;
        }

        .badge {
            display: inline-block;
            padding: 0.25rem 0.5rem;
            border-radius: 1rem;
            font-size: 0.6875rem;
            font-weight: 600;
        }

        .badge-success {
            background-color: rgba(76, 201, 240, 0.2);
            color: var(--success);
        }

        .badge-warning {
            background-color: rgba(248, 150, 30, 0.2);
            color: var(--warning);
        }

        .badge-danger {
            background-color: rgba(247, 37, 133, 0.2);
            color: var(--danger);
        }

        .text-muted {
            color: #6c757d;
            font-size: 0.75rem;
        }

        @media (max-width: 768px) {
            .form-grid {
                grid-template-columns: 1fr;
            }
            
            .actions {
                flex-direction: column;
            }
        }
    </style>
</head>
<body>
    <header>
        <div class="container">
            <h1>Gestão de Medicamentos</h1>
        </div>
    </header>
    
    <div class="container">
        <?php if (isset($msg)): ?>
            <div class="alert <?php echo isset($msg['success']) ? 'alert-success' : 'alert-danger'; ?>">
                <?php echo $msg['success'] ?? $msg['error']; ?>
            </div>
        <?php endif; ?>
        
        <div class="card">
            <h2><?= $editing ? '✏️ Editar Medicamento' : '➕ Novo Medicamento'; ?></h2>
            <form method="post">
                <input type="hidden" name="action" value="<?= $editing ? 'edit' : 'add'; ?>">
                <?php if ($editing): ?>
                    <input type="hidden" name="id" value="<?= $editData['id']; ?>">
                <?php endif; ?>
                
                <div class="form-grid">
                    <div class="form-group">
                        <label for="nome">Nome do Medicamento</label>
                        <input type="text" id="nome" name="nome" required value="<?= $editing ? htmlspecialchars($editData['nome']) : ''; ?>">
                    </div>
                    
                    <div class="form-group">
                        <label for="quantidade">Quantidade</label>
                        <input type="number" id="quantidade" name="quantidade" min="1" required value="<?= $editing ? $editData['quantidade'] : ''; ?>">
                    </div>
                    
                    <div class="form-group">
                        <label for="validade">Validade</label>
                        <input type="date" id="validade" name="validade" required value="<?= $editing ? $editData['validade'] : ''; ?>">
                    </div>
                    
                    <div class="form-group">
                        <label for="lote">N° do Lote</label>
                        <input type="text" id="lote" name="lote" required value="<?= $editing ? htmlspecialchars($editData['lote']) : ''; ?>">
                    </div>
                </div>
                
                <button type="submit" class="btn <?= $editing ? 'btn-success' : 'btn-primary'; ?>">
                    <?= $editing ? 'Atualizar' : 'Cadastrar'; ?>
                </button>
                
                <?php if ($editing): ?>
                    <a href="index.php" class="btn btn-danger">Cancelar</a>
                <?php endif; ?>
            </form>
        </div>
        
        <div class="card">
            <h2>📋 Estoque de Medicamentos</h2>
            
            <div class="search-container">
                <form method="get" action="index.php" style="flex: 1; display: flex; gap: 0.5rem;">
                    <input type="search" name="search" placeholder="Pesquisar..." value="<?= htmlspecialchars($search); ?>">
                    <button type="submit" class="btn btn-primary">Buscar</button>
                    <?php if ($search): ?>
                        <a href="index.php" class="btn btn-danger">Limpar</a>
                    <?php endif; ?>
                </form>
            </div>
            
            <?php if ($result->num_rows > 0): ?>
                <table>
                    <thead>
                        <tr>
                            <th>Medicamento</th>
                            <th>Qtd.</th>
                            <th>Validade</th>
                            <th>Lote</th>
                            <th>Ações</th>
                        </tr>
                    </thead>
                    <tbody>
                        <?php while($row = $result->fetch_assoc()): 
                            $today = new DateTime();
                            $expiry = new DateTime($row['validade']);
                            $interval = $today->diff($expiry);
                            $days = $interval->days;
                            $isExpired = $interval->invert;
                        ?>
                        <tr>
                            <td>
                                <div style="font-weight: 500;"><?= htmlspecialchars($row['nome']); ?></div>
                                <div class="text-muted">ID: <?= $row['id']; ?></div>
                            </td>
                            <td><?= $row['quantidade']; ?></td>
                            <td>
                                <div><?= date('d/m/Y', strtotime($row['validade'])); ?></div>
                                <?php if ($isExpired): ?>
                                    <span class="badge badge-danger">Vencido</span>
                                <?php elseif ($days <= 30): ?>
                                    <span class="badge badge-warning">Vence em <?= $days; ?>d</span>
                                <?php else: ?>
                                    <span class="badge badge-success">Válido</span>
                                <?php endif; ?>
                            </td>
                            <td><?= htmlspecialchars($row['lote']); ?></td>
                            <td>
                                <div class="actions">
                                    <a href="index.php?edit=<?= $row['id']; ?>" class="btn btn-warning btn-sm">Editar</a>
                                    <a href="index.php?delete=<?= $row['id']; ?>" class="btn btn-danger btn-sm" onclick="return confirm('Excluir este medicamento?')">Excluir</a>
                                </div>
                            </td>
                        </tr>
                        <?php endwhile; ?>
                    </tbody>
                </table>
            <?php else: ?>
                <p class="text-muted">Nenhum medicamento encontrado.</p>
            <?php endif; ?>
        </div>
    </div>
    
    <script>
        // Validação de data
        document.getElementById('validade').addEventListener('change', function() {
            const hoje = new Date();
            const dataSelecionada = new Date(this.value);
            
            if (dataSelecionada < hoje) {
                alert('Atenção: Data de validade vencida!');
            }
        });
        
        // Focar no campo de pesquisa quando houver busca
        <?php if ($search): ?>
            document.querySelector('input[name="search"]').focus();
        <?php endif; ?>
    </script>
</body>
</html>
<?php $conn->close(); ?>

Cadastro Medicamentos 
