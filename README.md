
import sqlite3

# -----------------------------------------------------
# Função para conectar (ou criar) o banco de dados
# -----------------------------------------------------
def conectar():
    conexao = sqlite3.connect("estoque.db")
    cursor = conexao.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS produtos (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            nome TEXT UNIQUE NOT NULL,
            quantidade INTEGER NOT NULL,
            preco REAL NOT NULL
        )
    """)
    conexao.commit()
    return conexao

# -----------------------------------------------------
# Função: Criar novo produto
# -----------------------------------------------------
def criar_produto(conexao, nome, quantidade, preco):
    try:
        cursor = conexao.cursor()
        cursor.execute("INSERT INTO produtos (nome, quantidade, preco) VALUES (?, ?, ?)",
                       (nome, quantidade, preco))
        conexao.commit()
        print(f"Produto '{nome}' adicionado com sucesso!\n")
    except sqlite3.IntegrityError:
        print("Erro: Já existe um produto com esse nome.\n")

# -----------------------------------------------------
# Função: Listar produtos
# -----------------------------------------------------
def listar_produtos(conexao):
    cursor = conexao.cursor()
    cursor.execute("SELECT * FROM produtos")
    produtos = cursor.fetchall()
    if produtos:
        print("\nProdutos no estoque:")
        print("-" * 40)
        for p in produtos:
            print(f"ID: {p[0]} | Nome: {p[1]} | Quantidade: {p[2]} | Preço: R$ {p[3]:.2f}")
        print("-" * 40 + "\n")
    else:
        print("Nenhum produto cadastrado.\n")

# -----------------------------------------------------
# Função: Atualizar produto
# -----------------------------------------------------
def atualizar_produto(conexao, id_produto, nova_quantidade, novo_preco):
    cursor = conexao.cursor()
    cursor.execute("SELECT * FROM produtos WHERE id = ?", (id_produto,))
    if cursor.fetchone() is None:
        print("Erro: Produto com esse ID não encontrado.\n")
        return
    cursor.execute("UPDATE produtos SET quantidade = ?, preco = ? WHERE id = ?",
                   (nova_quantidade, novo_preco, id_produto))
    conexao.commit()
    print("Produto atualizado com sucesso!\n")

# -----------------------------------------------------
# Função: Deletar produto
# -----------------------------------------------------
def deletar_produto(conexao, id_produto):
    cursor = conexao.cursor()
    cursor.execute("SELECT * FROM produtos WHERE id = ?", (id_produto,))
    if cursor.fetchone() is None:
        print("Erro: Produto com esse ID não existe.\n")
        return
    cursor.execute("DELETE FROM produtos WHERE id = ?", (id_produto,))
    conexao.commit()
    print("Produto removido com sucesso!\n")

# -----------------------------------------------------
# Função principal - Menu
# -----------------------------------------------------
def menu():
    conexao = conectar()

    while True:
        print("""
-------- MENU DE GERENCIAMENTO DE ESTOQUE ---------
1 - Adicionar novo produto
2 - Listar produtos
3 - Atualizar produto
4 - Deletar produto
5 - Sair
----------------------------------------------------
""")
        opcao = input("Escolha uma opção: ")

        if opcao == "1":
            nome = input("Nome do produto: ")
            try:
                quantidade = int(input("Quantidade: "))
                preco = float(input("Preço: "))
                criar_produto(conexao, nome, quantidade, preco)
            except ValueError:
                print("Erro: Quantidade e preço devem ser numéricos.\n")

        elif opcao == "2":
            listar_produtos(conexao)

        elif opcao == "3":
            try:
                id_produto = int(input("ID do produto a atualizar: "))
                nova_quantidade = int(input("Nova quantidade: "))
                novo_preco = float(input("Novo preço: "))
                atualizar_produto(conexao, id_produto, nova_quantidade, novo_preco)
            except ValueError:
                print("Erro: ID, quantidade e preço devem ser numéricos.\n")

        elif opcao == "4":
            try:
                id_produto = int(input("ID do produto a deletar: "))
                deletar_produto(conexao, id_produto)
            except ValueError:
                print("Erro: ID deve ser um número.\n")

        elif opcao == "5":
            conexao.close()
            print("Saindo do sistema. Até logo!")
            break

        else:
            print("Opção inválida. Tente novamente.\n")


# -----------------------------------------------------
# Execução do programa
# -----------------------------------------------------
if __name__ == "__main__":
    menu()
