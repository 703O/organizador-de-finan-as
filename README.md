import json
import os
import tkinter as tk
from tkinter import ttk, messagebox
from datetime import datetime

class GerenciadorDespesas:
    def __init__(self, arquivo='despesas.json'):
        self.arquivo = arquivo
        self.despesas = self.carregar_despesas()

    def carregar_despesas(self):
        if os.path.exists(self.arquivo):
            with open(self.arquivo, 'r') as f:
                return json.load(f)
        return []

    def salvar_despesas(self):
        with open(self.arquivo, 'w') as f:
            json.dump(self.despesas, f, indent=4)

    def adicionar_despesa(self, descricao, valor, mes):
        despesa = {
            'descricao': descricao,
            'valor': valor,
            'mes': mes
        }
        self.despesas.append(despesa)
        self.salvar_despesas()

    def listar_despesas(self, mes):
        if mes == "Todos":
            return self.despesas
        return [despesa for despesa in self.despesas if despesa['mes'] == mes]

    def remover_despesa(self, indice):
        try:
            self.despesas.pop(indice)
            self.salvar_despesas()
        except IndexError:
            print("Índice inválido.")

    def calcular_total_por_mes(self, mes):
        total = sum(despesa['valor'] for despesa in self.despesas if despesa['mes'] == mes)
        return total

class Calculadora:
    @staticmethod
    def somar(a, b):
        return a + b

    @staticmethod
    def subtrair(a, b):
        return a - b

    @staticmethod
    def multiplicar(a, b):
        return a * b

    @staticmethod
    def dividir(a, b):
        if b == 0:
            return "Erro: Divisão por zero"
        return a / b

class Aplicacao:
    def __init__(self, root):
        self.gerenciador = GerenciadorDespesas()
        self.calculadora = Calculadora()
        self.root = root
        self.root.title("Gerenciador de Despesas")

        self.tab_control = ttk.Notebook(root)
        self.tab_despesas = ttk.Frame(self.tab_control)
        self.tab_calculadora = ttk.Frame(self.tab_control)
        self.tab_control.add(self.tab_despesas, text='Despesas')
        self.tab_control.add(self.tab_calculadora, text='Calculadora')
        self.tab_control.pack(expand=1, fill='both')

        self.criar_gui_despesas()
        self.criar_gui_calculadora()

        # Set initial focus to descricao entry
        self.entrada_descricao.focus_set()

    def criar_gui_despesas(self):
        self.label_mes = ttk.Label(self.tab_despesas, text="Mês:")
        self.label_mes.grid(column=0, row=0, padx=10, pady=10)
        self.meses = ["Janeiro", "Fevereiro", "Março", "Abril", "Maio", "Junho", "Julho", "Agosto", "Setembro", "Outubro", "Novembro", "Dezembro", "Todos"]
        self.selecao_mes = ttk.Combobox(self.tab_despesas, values=self.meses)
        self.selecao_mes.set(datetime.now().strftime("%B"))
        self.selecao_mes.grid(column=1, row=0, padx=10, pady=10)

        self.label_descricao = ttk.Label(self.tab_despesas, text="Descrição:")
        self.label_descricao.grid(column=0, row=1, padx=10, pady=10)
        self.entrada_descricao = ttk.Entry(self.tab_despesas, width=30)
        self.entrada_descricao.grid(column=1, row=1, padx=10, pady=10)

        self.label_valor = ttk.Label(self.tab_despesas, text="Valor:")
        self.label_valor.grid(column=0, row=2, padx=10, pady=10)
        self.entrada_valor = ttk.Entry(self.tab_despesas, width=30)
        self.entrada_valor.grid(column=1, row=2, padx=10, pady=10)

        self.botao_adicionar = ttk.Button(self.tab_despesas, text="Adicionar", command=self.adicionar_despesa)
        self.botao_adicionar.grid(column=0, row=3, padx=10, pady=10)

        self.botao_listar = ttk.Button(self.tab_despesas, text="Listar", command=self.listar_despesas)
        self.botao_listar.grid(column=1, row=3, padx=10, pady=10)

        self.lista_despesas = tk.Listbox(self.tab_despesas, width=50)
        self.lista_despesas.grid(column=0, row=4, columnspan=2, padx=10, pady=10)

        self.label_total = ttk.Label(self.tab_despesas, text="Total do Mês:")
        self.label_total.grid(column=0, row=5, padx=10, pady=10)
        self.entrada_total = ttk.Entry(self.tab_despesas, width=30, state='readonly')
        self.entrada_total.grid(column=1, row=5, padx=10, pady=10)

        self.botao_remover = ttk.Button(self.tab_despesas, text="Remover", command=self.remover_despesa)
        self.botao_remover.grid(column=0, row=6, columnspan=2, padx=10, pady=10)

        # Bind Enter key to move to the next widget
        self.entrada_descricao.bind("<Return>", lambda event: self.entrada_valor.focus_set())
        self.entrada_valor.bind("<Return>", lambda event: self.adicionar_despesa())

    def adicionar_despesa(self):
        descricao = self.entrada_descricao.get()
        try:
            valor = float(self.entrada_valor.get())
            mes = self.selecao_mes.get()
            self.gerenciador.adicionar_despesa(descricao, valor, mes)
            messagebox.showinfo("Sucesso", "Despesa adicionada com sucesso!")
            self.entrada_descricao.delete(0, tk.END)
            self.entrada_valor.delete(0, tk.END)
            self.listar_despesas()  # Atualiza a lista de despesas após adicionar
            self.entrada_descricao.focus_set()  # Retorna o foco para a entrada de descrição
        except ValueError:
            messagebox.showerror("Erro", "Valor inválido!")

    def listar_despesas(self):
        self.lista_despesas.delete(0, tk.END)
        mes = self.selecao_mes.get()
        despesas = self.gerenciador.listar_despesas(mes)
        for i, despesa in enumerate(despesas):
            self.lista_despesas.insert(tk.END, f"{i + 1}. {despesa['descricao']} - R$ {despesa['valor']:.2f} ({despesa['mes']})")
        
        total = self.gerenciador.calcular_total_por_mes(mes)
        self.entrada_total.config(state='normal')
        self.entrada_total.delete(0, tk.END)
        self.entrada_total.insert(0, f"R$ {total:.2f}")
        self.entrada_total.config(state='readonly')

    def remover_despesa(self):
        try:
            selecionado = self.lista_despesas.curselection()[0]
            self.gerenciador.remover_despesa(selecionado)
            self.listar_despesas()
            messagebox.showinfo("Sucesso", "Despesa removida com sucesso!")
        except IndexError:
            messagebox.showerror("Erro", "Nenhuma despesa selecionada!")

    def criar_gui_calculadora(self):
        self.label_num1 = ttk.Label(self.tab_calculadora, text="Número 1:")
        self.label_num1.grid(column=0, row=0, padx=10, pady=10)
        self.entrada_num1 = ttk.Entry(self.tab_calculadora, width=30)
        self.entrada_num1.grid(column=1, row=0, padx=10, pady=10)

        self.label_num2 = ttk.Label(self.tab_calculadora, text="Número 2:")
        self.label_num2.grid(column=0, row=1, padx=10, pady=10)
        self.entrada_num2 = ttk.Entry(self.tab_calculadora, width=30)
        self.entrada_num2.grid(column=1, row=1, padx=10, pady=10)

        self.botao_somar = ttk.Button(self.tab_calculadora, text="Somar", command=lambda: self.calcular('somar'))
        self.botao_somar.grid(column=0, row=2, padx=10, pady=10)

        self.botao_subtrair = ttk.Button(self.tab_calculadora, text="Subtrair", command=lambda: self.calcular('subtrair'))
        self.botao_subtrair.grid(column=1, row=2, padx=10, pady=10)

        self.botao_multiplicar = ttk.Button(self.tab_calculadora, text="Multiplicar", command=lambda: self.calcular('multiplicar'))
        self.botao_multiplicar.grid(column=0, row=3, padx=10, pady=10)

        self.botao_dividir = ttk.Button(self.tab_calculadora, text="Dividir", command=lambda: self.calcular('dividir'))
        self.botao_dividir.grid(column=1, row=3, padx=10, pady=10)

        self.label_resultado = ttk.Label(self.tab_calculadora, text="Resultado:")
        self.label_resultado.grid(column=0, row=4, padx=10, pady=10)
        self.entrada_resultado = ttk.Entry(self.tab_calculadora, width=30, state='readonly')
        self.entrada_resultado.grid(column=1, row=4, padx=10, pady=10)

        # Bind Enter key to move to the next widget
        self.entrada_num1.bind("<Return>", lambda event: self.entrada_num2.focus_set())
        self.entrada_num2.bind("<Return>", lambda event: self.calcular('somar'))

    def calcular(self, operacao):
        try:
            num1 = float(self.entrada_num1.get())
            num2 = float(self.entrada_num2.get())
            if operacao == 'somar':
                resultado = self.calculadora.somar(num1, num2)
            elif operacao == 'subtrair':
                resultado = self.calculadora.subtrair(num1, num2)
            elif operacao == 'multiplicar':
                resultado = self.calculadora.multiplicar(num1, num2)
            elif operacao == 'dividir':
                resultado = self.calculadora.dividir(num1, num2)
            self.entrada_resultado.config(state='normal')
            self.entrada_resultado.delete(0, tk.END)
            self.entrada_resultado.insert(0, str(resultado))
            self.entrada_resultado.config(state='readonly')
        except ValueError:
            messagebox.showerror("Erro", "Número inválido!")

if __name__ == '__main__':
    root = tk.Tk()
    app = Aplicacao(root)
    root.mainloop()
