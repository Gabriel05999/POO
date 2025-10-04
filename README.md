import sqlite3

# Conecta ou cria o banco de dados
conexao = sqlite3.connect("estoque.db")
cursor = conexao.cursor()

# Cria a tabela se não existir
cursor.execute("""
CREATE TABLE IF NOT EXISTS produtos (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nome TEXT UNIQUE NOT NULL,
    quantidade INTEGER NOT NULL,
    preco REAL NOT NULL
)
""")
conexao.commit()

# Funções CRUD
def adicionar(nome, quantidade, preco):
    try:
        cursor.execute("INSERT INTO produtos (nome, quantidade, preco) VALUES (?, ?, ?)",
                       (nome, quantidade, preco))
        conexao.commit()
        print("1
              Produto adicionado com sucesso!")
    except sqlite3.IntegrityError:
        print("Erro: Já existe um produto com esse nome.")

def listar():
    cursor.execute("SELECT * FROM produtos")
    produtos = cursor.fetchall()
    if produtos:
        for p in produtos:
            print(f"ID: {p[0]} | Nome: {p[1]} | Quantidade: {p[2]} | Preço: R$ {p[3]:.2f}")
    else:
        print("Nenhum produto cadastrado.")

def atualizar(id_produto, nova_qtd, novo_preco):
    cursor.execute("UPDATE produtos SET quantidade=?, preco=? WHERE id=?",
                   (nova_qtd, novo_preco, id_produto))
    if cursor.rowcount > 0:
        conexao.commit()
        print("Produto atualizado com sucesso!")
    else:
        print("ID não encontrado.")

def excluir(id_produto):
    cursor.execute("DELETE FROM produtos WHERE id=?", (id_produto,))
    if cursor.rowcount > 0:
        conexao.commit()
        print("Produto excluído com sucesso!")
    else:
        print("ID não encontrado.")

# Menu simples
def menu():
    while True:
        print("""
==== GERENCIAMENTO DE ESTOQUE ====
1 - Adicionar produto
2 - Listar produtos
3 - Atualizar produto
4 - Excluir produto
5 - Sair
""")
        opcao = input("Escolha: ")

        if opcao == "1":
            nome = input("Nome: ")
            qtd = int(input("Quantidade: "))
            preco = float(input("Preço: "))
            adicionar(nome, qtd, preco)

        elif opcao == "2":
            listar()

        elif opcao == "3":
            idp = int(input("ID do produto: "))
            qtd = int(input("Nova quantidade: "))
            preco = float(input("Novo preço: "))
            atualizar(idp, qtd, preco)

        elif opcao == "4":
            idp = int(input("ID do produto: "))
            excluir(idp)

        elif opcao == "5":
            print("Encerrando o sistema...")
            break

        else:
            print("Opção inválida, tente novamente.")

menu()
conexao.close()
