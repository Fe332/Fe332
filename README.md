import tkinter as tk
from tkinter import messagebox, ttk
import mysql.connector
from mysql.connector import Error
 
# Configuração do banco de dados
def setup_database():
    connection = None
    try:
        connection = mysql.connector.connect(
            host='localhost',
            user='root',          
            password='acesso123', 
            database='livros'  
        )
       
        if connection.is_connected():
            cursor = connection.cursor()
            cursor.execute("DROP TABLE IF EXISTS historico")
            cursor.execute("DROP TABLE IF EXISTS emprestimos")
            cursor.execute("DROP TABLE IF EXISTS leitores")
            cursor.execute("DROP TABLE IF EXISTS livros")
 
            cursor.execute('''CREATE TABLE IF NOT EXISTS livros (
                id INT AUTO_INCREMENT PRIMARY KEY,
                titulo VARCHAR(255), autor VARCHAR(255), genero VARCHAR(255), idioma VARCHAR(255), localizacao VARCHAR(255))''')
            cursor.execute('''CREATE TABLE IF NOT EXISTS leitores (
                id INT AUTO_INCREMENT PRIMARY KEY,
                nome VARCHAR(255), email VARCHAR(255))''')
            cursor.execute('''CREATE TABLE IF NOT EXISTS emprestimos (
                id INT AUTO_INCREMENT PRIMARY KEY,
                livro_id INT, leitor_id INT,
                data_emprestimo DATE, data_devolucao DATE,
                FOREIGN KEY (livro_id) REFERENCES livros (id),
                FOREIGN KEY (leitor_id) REFERENCES leitores (id))''')
            cursor.execute('''CREATE TABLE IF NOT EXISTS historico (
                id INT AUTO_INCREMENT PRIMARY KEY,
                acao VARCHAR(255), livro_id INT, leitor_id INT,
                livro_titulo VARCHAR(255), livro_autor VARCHAR(255), leitor_nome VARCHAR(255), data DATE,
                FOREIGN KEY (livro_id) REFERENCES livros (id),
                FOREIGN KEY (leitor_id) REFERENCES leitores (id))''')
 
            connection.commit()
    except Error as e:
        print(f"Erro ao conectar ao MySQL: {e}")
    finally:
        if connection and connection.is_connected():
            cursor.close()
            connection.close()
 
# Funções de gerenciamento
def add_livro(titulo, autor, genero, idioma, localizacao):
    try:
        connection = mysql.connector.connect(
            host='localhost',
            user='root',
            password='acesso123',
            database='livros'
        )
 
        cursor = connection.cursor()
        cursor.execute("INSERT INTO livros (titulo, autor, genero, idioma, localizacao) VALUES (%s, %s, %s, %s, %s)",
                       (titulo, autor, genero, idioma, localizacao))
        livro_id = cursor.lastrowid
        cursor.execute("INSERT INTO historico (acao, livro_id, livro_titulo, livro_autor, data) VALUES (%s, %s, %s, %s, CURDATE())",
                       ("Cadastro", livro_id, titulo, autor))
        connection.commit()
        messagebox.showinfo("Sucesso", "Livro cadastrado com sucesso!")
    except Error as e:
        messagebox.showerror("Erro", f"Erro ao cadastrar livro: {e}")
    finally:
        if connection and connection.is_connected():
            cursor.close()
            connection.close()
 
def add_leitor(nome, email):
    try:
        connection = mysql.connector.connect(
            host='localhost',
            user='root',
            password='acesso123',
            database='livros'
        )
 
        cursor = connection.cursor()
        cursor.execute("INSERT INTO leitores (nome, email) VALUES (%s, %s)", (nome, email))
        connection.commit()
        messagebox.showinfo("Sucesso", "Leitor cadastrado com sucesso!")
    except Error as e:
        messagebox.showerror("Erro", f"Erro ao cadastrar leitor: {e}")
    finally:
        if connection and connection.is_connected():
            cursor.close()
            connection.close()
 
def emprestar_livro(livro_id, leitor_id):
    try:
        connection = mysql.connector.connect(
            host='localhost',
            user='root',
            password='acesso123',
            database='livros'
        )
 
        cursor = connection.cursor()
        cursor.execute("SELECT titulo, autor FROM livros WHERE id = %s", (livro_id,))
        livro_info = cursor.fetchone()
        cursor.execute("SELECT nome FROM leitores WHERE id = %s", (leitor_id,))
        leitor_info = cursor.fetchone()
        if livro_info and leitor_info:
            cursor.execute("INSERT INTO emprestimos (livro_id, leitor_id, data_emprestimo) VALUES (%s, %s, CURDATE())",
                           (livro_id, leitor_id))
            cursor.execute("INSERT INTO historico (acao, livro_id, leitor_id, livro_titulo, livro_autor, leitor_nome, data) VALUES (%s, %s, %s, %s, %s, %s, CURDATE())",
                           ("Empréstimo", livro_id, leitor_id, livro_info[0], livro_info[1], leitor_info[0]))
            connection.commit()
            messagebox.showinfo("Sucesso", "Livro emprestado com sucesso!")
        else:
            messagebox.showerror("Erro", "Livro ou leitor não encontrado.")
    except Error as e:
        messagebox.showerror("Erro", f"Erro ao emprestar livro: {e}")
    finally:
        if connection and connection.is_connected():
            cursor.close()
            connection.close()
 
def mostrar_historico():
    try:
        connection = mysql.connector.connect(
            host='localhost',
            user='root',
            password='acesso123',
            database='livros'
        )
 
        cursor = connection.cursor()
        cursor.execute("SELECT * FROM historico")
        rows = cursor.fetchall()
        historico_window = tk.Toplevel()
        historico_window.title("Histórico de Ações")
        tree = ttk.Treeview(historico_window, columns=("ID", "Ação", "ID Livro", "Título", "Autor", "ID Leitor", "Nome Leitor", "Data"), show="headings")
        tree.heading("ID", text="ID")
        tree.heading("Ação", text="Ação")
        tree.heading("ID Livro", text="ID Livro")
        tree.heading("Título", text="Título")
        tree.heading("Autor", text="Autor")
        tree.heading("ID Leitor", text="ID Leitor")
        tree.heading("Nome Leitor", text="Nome Leitor")
        tree.heading("Data", text="Data")
        for row in rows:
            tree.insert("", tk.END, values=row)
        tree.pack(expand=True, fill=tk.BOTH)
        ttk.Button(historico_window, text="Fechar", command=historico_window.destroy).pack()
    except Error as e:
        messagebox.showerror("Erro", f"Erro ao mostrar histórico: {e}")
    finally:
        if connection and connection.is_connected():
            cursor.close()
            connection.close()
 
def save_historico():
    try:
        connection = mysql.connector.connect(
            host='localhost',
            user='root',
            password='acesso123',
            database='livros'
        )
 
        cursor = connection.cursor()
        cursor.execute("SELECT * FROM historico")
        rows = cursor.fetchall()
   
        with open("historico.txt", "w") as file:
            for row in rows:
                file.write("\t".join(map(str, row)) + "\n")
   
        messagebox.showinfo("Sucesso", "Histórico salvo com sucesso!")
    except Error as e:
        messagebox.showerror("Erro", f"Erro ao salvar histórico: {e}")
    finally:
        if connection and connection.is_connected():
            cursor.close()
            connection.close()
 
# Interface Gráfica
class BibliotecaApp(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Sistema de Biblioteca")
        self.geometry("600x400")
       
        # Cadastro de Livros
        self.label_livros = tk.Label(self, text="Cadastro de Livros")
        self.label_livros.pack()
        self.titulo_entry = tk.Entry(self)
        self.titulo_entry.pack()
        self.titulo_entry.insert(0, "Título")
        self.autor_entry = tk.Entry(self)
        self.autor_entry.pack()
        self.autor_entry.insert(0, "Autor")
        self.genero_entry = tk.Entry(self)
        self.genero_entry.pack()
        self.genero_entry.insert(0, "Gênero")
        self.idioma_entry = tk.Entry(self)
        self.idioma_entry.pack()
        self.idioma_entry.insert(0, "Idioma")
        self.localizacao_entry = tk.Entry(self)
        self.localizacao_entry.pack()
        self.localizacao_entry.insert(0, "Localização")
        self.btn_add_livro = tk.Button(self, text="Adicionar Livro", command=self.cadastrar_livro)
        self.btn_add_livro.pack()
 
        # Cadastro de Leitores
        self.label_leitores = tk.Label(self, text="Cadastro de Leitores")
        self.label_leitores.pack()
        self.nome_entry = tk.Entry(self)
        self.nome_entry.pack()
        self.nome_entry.insert(0, "Nome")
        self.email_entry = tk.Entry(self)
        self.email_entry.pack()
        self.email_entry.insert(0, "Email")        
        self.btn_add_leitor = tk.Button(self, text="Adicionar Leitor", command=self.cadastrar_leitor)
        self.btn_add_leitor.pack()
 
        # Empréstimo
        self.label_emprestimos = tk.Label(self, text="Empréstimo de Livros")
        self.label_emprestimos.pack()
        self.livro_id_entry = tk.Entry(self)
        self.livro_id_entry.pack()
        self.livro_id_entry.insert(0, "ID do Livro")
        self.leitor_id_entry = tk.Entry(self)
        self.leitor_id_entry.pack()
        self.leitor_id_entry.insert(0, "ID do Leitor")
        self.btn_add_emprestar = tk.Button(self, text="Emprestar Livro", command=self.emprestar)
        self.btn_add_emprestar.pack()
 
        # Histórico
        self.btn_historico = tk.Button(self, text="Ver Histórico", command=mostrar_historico)
        self.btn_historico.pack()

        # Botão para gerar relatório de livros
        self.btn_relatorio = tk.Button(self, text="Gerar Relatório de Livros", command= gerar_relatorio_livros)
        self.btn_relatorio.pack()

 
    def cadastrar_livro(self):
        add_livro(self.titulo_entry.get(), self.autor_entry.get(), self.genero_entry.get(), self.idioma_entry.get(), self.localizacao_entry.get())
 
    def cadastrar_leitor(self):
        add_leitor(self.nome_entry.get(), self.email_entry.get())
 
    def emprestar(self):
        try:
            livro_id = int(self.livro_id_entry.get())
            leitor_id = int(self.leitor_id_entry.get())
            emprestar_livro(livro_id, leitor_id)
        except ValueError:
            messagebox.showerror("Erro", "Por favor, insira IDs válidos.")

def gerar_relatorio_livros():
    try:
        connection = mysql.connector.connect(
            host='localhost',
            user='root',
            password='acesso123',
            database='livros'
        )

        cursor = connection.cursor()
        cursor.execute("SELECT * FROM livros")
        rows = cursor.fetchall()
        
        relatorio_window = tk.Toplevel()
        relatorio_window.title("Relatório de Livros")
        tree = ttk.Treeview(relatorio_window, columns=("ID", "Título", "Autor", "Gênero", "Idioma", "Localização"), show="headings")
        
        tree.heading("ID", text="ID")
        tree.heading("Título", text="Título")
        tree.heading("Autor", text="Autor")
        tree.heading("Gênero", text="Gênero")
        tree.heading("Idioma", text="Idioma")
        tree.heading("Localização", text="Localização")
        
        for row in rows:
            tree.insert("", tk.END, values=row)
        
        tree.pack(expand=True, fill=tk.BOTH)
        ttk.Button(relatorio_window, text="Fechar", command=relatorio_window.destroy).pack()
    except Error as e:
        messagebox.showerror("Erro", f"Erro ao gerar relatório: {e}")
    finally:
        if connection and connection.is_connected():
            cursor.close()
            connection.close()
 
# Tela de login
def login_window():
    def login():
        username = entry_user.get()
        password = entry_pass.get()
        if username == "root" and password == "acesso123":
            messagebox.showinfo("Login", "Login bem-sucedido!")
            window.destroy()
            app = BibliotecaApp()
            app.mainloop()
        else:
            messagebox.showerror("Login", "Usuário ou senha inválidos.")
 
    window = tk.Tk()
    window.title("Tela de Login")
    tk.Label(window, text="Usuário:").pack()
    entry_user = tk.Entry(window)
    entry_user.pack()
    tk.Label(window, text="Senha:").pack()
    entry_pass = tk.Entry(window, show='*')
    entry_pass.pack()
    tk.Button(window, text="Login", command=login).pack()
    window.mainloop()
 
if __name__ == "__main__":
    setup_database()  
    login_window()    
 
 
