# Otimizando-o-Sistema-Banc-rio-com-Fun-es-Python
Este repositório contém o resumo das lições aprendidas durante o desenvolvimento do lab na DIO - Python
import sqlite3
from tabulate import tabulate
import textwrap
from datetime import datetime
# Banco como exemplo....
# Cria (ou abre) o banco de dados no arquivo 'Banco2.db'
def cria_bd():
    conn = sqlite3.connect('Banco2.db')
    cursor = conn.cursor()
# Criação da tabela Login
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS Agencia (
        agencia INTEGER NOT NULL,
        PRIMARY KEY (agencia)    
    )
    ''')
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS Conta (
        conta INTEGER PRIMARY KEY AUTOINCREMENT UNIQUE,
        agencia INTEGER NOT NULL,
        FOREIGN KEY (agencia) REFERENCES Agencia(agencia)
    )
    ''')
# Criação da tabela Cliente
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS Cliente (
        CPF TEXT NOT NULL UNIQUE,
        nome TEXT NOT NULL,
        sobre_nome TEXT NOT NULL,
        agencia INTEGER NOT NULL,
        conta INTEGER NOT NULL,
        PRIMARY KEY (CPF),
        FOREIGN KEY (agencia) REFERENCES Agencia(agencia),
        FOREIGN KEY (conta) REFERENCES Conta(conta)
    )
    ''')
# Criação da tabela Financiamento
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS Financiamento (
        Financiamento INTEGER NOT NULL UNIQUE,
        CPF TEXT NOT NULL,
        agencia INTEGER NOT NULL,
        FOREIGN KEY (CPF) REFERENCES Cliente(CPF),
        FOREIGN KEY (agencia) REFERENCES Agencia(agencia)
    )
    ''')
# Dados de 1 cliente comum
#beta_cliente = ('759754679', 'Junior', 'Globo', 3, 46)
# Comando INSERT INTO
#cursor.execute('''
#    INSERT INTO Cliente (CPF, nome, sobre_nome, agencia, conta)
#    VALUES (?, ?, ?, ?, ?)
#    ''', beta_cliente)
# Salvando e fechando
    conn.commit()
    conn.close()
#cursor.execute("UPDATE Cliente set CPF='303456333' nome='Julio', sobre_nome='Cezar', agencia='2', conta='55' WHERE CPF='303456333'")
#clientes = cursor.fetchall()
# Exibir resultados
#for cliente in clientes:
#    print(cliente)
global saldo
saldo = float()
def conecta_bd():
    # Conecta ao banco
    conn = sqlite3.connect('Banco2.db')
    cursor = conn.cursor()
    return conn, cursor
def conecta_bd_cadastro(cliente):
    # Conecta ao banco
    conn = sqlite3.connect('Banco2.db')
    cursor = conn.cursor()
    # Executa o INSERT
    cursor.execute('''
    INSERT INTO Cliente (CPF, nome, sobre_nome, agencia, conta)
    VALUES (?, ?, ?, ?, ?)
    ''', cliente)

    conn.commit()
    conn.close()
def informa_dados(): 
    global agencia   
    agencia = int(input("Informe sua agencia: "))
    global conta
    conta = int(input("Informe sua conta: "))
    print("Agência:", agencia, "| Conta:", conta)
def minha_conta(agencia,conta):
    print("🔍 Verificando sua conta...")

    conn, cursor = conecta_bd()
    cursor = conn.cursor()
    if conn is None:
        print("Falha de conexao com o banco.")
        return

    cursor.execute("SELECT * FROM Cliente WHERE agencia = ? AND conta = ?", (agencia,conta))
    resultado = cursor.fetchone()
    agencia = int(agencia)
    conta = int(conta)
    try:
        if resultado is not None:
            variavel_espaco = " "
            resultado_mescla = resultado[1] + variavel_espaco + resultado[2]
            linha_unica_lista = [
                [resultado_mescla]
            ]
            global cliente_extrato
            cliente_extrato = resultado_mescla
            dados_tabela = [
                ["Agência", resultado[3]],
                ["Conta", resultado[4]],
            ]
            print(f"Seja bem vindo!!")
            linha_unica = tabulate(linha_unica_lista, tablefmt="grid")  
            linhas = linha_unica.splitlines()
            total_width = len(linhas[0])            
            titulo = linha_unica .center(total_width + 2, " ")
        
            tabela = tabulate(dados_tabela, tablefmt="grid")
            print(titulo)
            print(tabela)
            data = datetime.now()
            global ano
            ano = data.year
            global mes
            mes = data.month
            global dia 
            dia = data.day
            global hora
            hora = data.hour
            global minutos
            minutos = data.minute 
            global data_formatada
            data_formatada = f"Dia: \t{dia}/{mes}/{ano} {hora}:{minutos}"
            print(data_formatada)  
        else:
            conn.close()
            print(f"🚫 Agencia: {agencia} e conta: {conta} não encontrados!") 
            exit()
    except:
        conn.close()
        exit()    
    conn.close()
def menu_admin():
    menu_admin = """\t
    [N]\tNovo Cliente
    [A]\tAlterar Cliente
    [E]\tExibir Clientes
    [Nv.Ag]\tNova Agencia
    [M]\tMetas
    [q]\tSair 
    => """
    return input(textwrap.dedent(menu_admin).strip().upper())
def consulta_clientes():
    print(f"\tCarregando exibicao:")
    conn, cursor = conecta_bd()
    # Executa o select
    cursor.execute("SELECT * FROM Cliente")
    # Faz a consulta
    consulta_clientes = cursor.fetchall()
    # Exibe os dados
    for cliente in consulta_clientes:
        dados_tabela = [
            ["CPF:", cliente[0]],
            ["Nome:", cliente[1]],
            ["Sobrenome:", cliente[2]],
            ["Agência:", cliente[3]],
            ["Conta:", cliente[4]]
            ]
        tabela = tabulate(dados_tabela, headers="firstrow", tablefmt="grid")
        print(tabela)
    conn.close()
def cadastra_novos_clientes():
    print("Cadastra Novo cliente")
    cpf_novo_cadastro = int(9)
    agencia_novo_cadastro = int(input("\n Escolha a Agencia: "))
    cpf_novo_cadastro = int(input("\n Informe o CPF para ser cadastrado: ")) 
    nome_novo_cad = str(input("\n Informe nome para ser cadastrado: ")) 
    sobre_nome_cad = str(input("\n Informe o sobrenome para ser cadastrado: "))
    nova_conta = gerar_nova_conta()
    cliente = (cpf_novo_cadastro, nome_novo_cad, sobre_nome_cad, agencia_novo_cadastro, nova_conta)
    conecta_bd_cadastro(cliente)
    print(f"O cliente foi cadastrado: {cliente}")
def gerar_nova_conta():
    # 1. Conecta ao banco
    conn, cursor = conecta_bd()
    cursor = conn.cursor()
    # 2. Pega o maior valor existente na coluna 'conta'
    cursor.execute("SELECT MAX(conta) FROM Conta")
    resultado = cursor.fetchone()[0]
    # 3. Se não houver nenhuma conta, inicia em zero
    maior = resultado if resultado is not None else 0
    # 4. Fecha a conexão
    conn.close()
    # 5. Retorna o próximo número de conta
    return maior + 1
def exibir_menu():
    menu = """\t
    [d]\tDepositar
    [s]\tSacar
    [c]\tCredito 
    [e]\tExtrato
    [q]\tSair
    => """
    return input(textwrap.dedent(menu))
def deposito():
    valor = float(input("Informe o valor do depósito: \n"))
    if valor > 0:
        global saldo
        saldo += valor
        global extrato
        extrato += f"Depósito: R$ {valor:.2f}\n"
        print(f"\t Depósito realizado no valor de: R$ {valor:.2f}\n")
    else:
        print("\t Operação falhou! O valor informado é inválido.\n") 
def saque(): 
    valor = float(input("Informe o valor do saque: \n"))
    global saldo
    global limite
    global LIMITE_SAQUES
    LIMITE_SAQUES = 3
    global extrato
    numero_saques = 0
    excedeu_saldo = valor > saldo
    excedeu_limite = valor > limite
    excedeu_saques = numero_saques >= LIMITE_SAQUES
    if excedeu_saldo:
        print("Operação falhou! Você não tem saldo suficiente.")
    elif excedeu_limite:
        print("Operação falhou! O valor do saque excede o limite.")
    elif excedeu_saques:
        print("Operação falhou! Número máximo de saques excedido.")
    elif valor > 0:
        saldo -= valor
        extrato += f"Saque: R$ {valor:.2f}\n"
        print(f"\t Saque realizado no valor de: R$ {valor:.2f}\n")
        numero_saques += 1
    else:
        print("Operação falhou! O valor informado é inválido.")   
def credito_financiamento():
    global data
    global extrato
    global saldo 
    global limite
    taxa_juros = 15
    score = 900
    numero_saques = 0
    global LIMITE_SAQUES 
    LIMITE_EM_MESES = 72
    salario = float(input(f"Informe o seu salario: "))
    MARGEM_FINANCIAMENTO = (salario * 60) * 0.30 
    if salario >= 1545:
        print("Analisando perfil, por favor aguarde.....")
        score = 900
        if saldo >= 500:
            if numero_saques == 0:
                calculadora_perfil = score * LIMITE_SAQUES
                if calculadora_perfil >= 2700:
                    MARGEM_FINANCIAMENTO_CLIENTE = MARGEM_FINANCIAMENTO
                    print(f"Sua Margem é:{MARGEM_FINANCIAMENTO_CLIENTE}")
                    menu_simular= """
                        [S] Simular
                        [P] Parar
                        => """
                    opcao = input(menu_simular)
                    if opcao == "S":
                        print(f"A taxa de juros selic anual é:\n{taxa_juros}")
                        valor_em = float(input("Informe o valor do empréstimo: "))
                        prazo_pagamento = float(input("Informe em quantos meses deseja pagar (limite 72): "))
                        if valor_em > 0 and valor_em <= MARGEM_FINANCIAMENTO_CLIENTE and prazo_pagamento <= LIMITE_EM_MESES:
                            valor_parcelas = (valor_em * ((((prazo_pagamento / 12) * taxa_juros) / 100) + 1)) / prazo_pagamento
                            VALOR_COMPROMETIDO_SALARIO = salario / 2
                            if valor_parcelas <= VALOR_COMPROMETIDO_SALARIO:
                                valor_parcelas_cliente = valor_parcelas
                                print(f"O emprestimo de: {valor_em} foi aprovado, com {prazo_pagamento} parcelas de R$ {valor_parcelas_cliente}")
                                if valor_em > 0 and prazo_pagamento > 0 and valor_parcelas_cliente > 0:
                                    menu_em = """
                                        [C] Confirmar emprestimo!
                                        [D] Deixar para depois!
                                        => """ 
                                    opcao = input(menu_em)
                                    if opcao == "C":
                                        global dia
                                        global mes
                                        global ano
                                        global hora
                                        global minutos
                                        saldo += valor_em
                                        data_emprestimo = f"{dia}/{mes}/{ano}"
                                        horario_emprestimo = f" horario: {hora} : {minutos}"
                                        print(f"Voce fez o emprestimo de R$ {valor_em} as parcelas de R$ {valor_parcelas_cliente} seram automaticamente recolhidas mes a mes o deposito do valor {valor_em} ja foi efetuado!")
                                        extrato += f"Voce fez um emprestimo dia: {data_emprestimo} \n" f"no{horario_emprestimo} no valor de R$ {valor_em}\n" f"com parcelas mensais de R$ {valor_parcelas_cliente}\n"
                                    elif opcao == "D":
                                        print(f"Certo deixar para depois!")
                                        return
                                else: 
                                    print("Ocorreu um erro inesperado tente novamente!")                                               
                            elif valor_parcelas > VALOR_COMPROMETIDO_SALARIO:
                                adequando_parcelas = (valor_parcelas / VALOR_COMPROMETIDO_SALARIO)
                                prazo_pagamento_adequacao = (prazo_pagamento * adequando_parcelas)
                                if prazo_pagamento_adequacao <= LIMITE_EM_MESES:
                                    valor_parcelas = (valor_em * ((((prazo_pagamento_adequacao / 12) * taxa_juros) / 100) + 1)) / prazo_pagamento_adequacao
                                    print(f"O emprestimo de: {valor_em} foi aprovado, com {prazo_pagamento_adequacao} com parcelas de R$ {valor_parcelas}")
                                    if valor_em > 0 and prazo_pagamento > 0 and valor_parcelas_cliente > 0:
                                        menu_em = """
                                        [C] Confirmar emprestimo!
                                        [D] Deixar para depois!
                                        => """ 
                                    opcao = input(menu_em)
                                    if opcao == "C":
                                        saldo += valor_em
                                        print(f"Voce fez o emprestimo de R$ {valor_em} as parcelas de R$ {valor_parcelas_cliente} seram automaticamente recolhidas mes a mes somente aguardar o deposito do valor {valor_em}")
                                    elif opcao == "D":
                                        print(f"Certo deixar para depois!")
                                        return
                                    else: 
                                        print("Ocorreu um erro inesperado tente novamente!") 
                                        return  
                                elif prazo_pagamento_adequacao > LIMITE_EM_MESES:
                                    print("Infelizmente não foi possível emprestimo nestas condições por favor verificar o valor de cada parcela, não pode ultrapassar metade do seu salário;")
                            else:
                                print("Não sabemos o que ocorreu, por favor tente novamente!")      
                        else:
                            print("Você deve informar um valor para o empréstimo")
                    elif opcao == "P":
                        print("Voce escolheu sair, saindo do aplicativo!")
                        return
                    else:
                        print("Voce deve escolher um opcao")
                        return
                            
                elif calculadora_perfil >= 1350:
                    MARGEM_FINANCIAMENTO_CLIENTE = MARGEM_FINANCIAMENTO * 0.75
                    print(f"Sua Margem é:{MARGEM_FINANCIAMENTO_CLIENTE}")
                    menu_simular= """
                        [S] Simular
                        [P] Parar
                    => """
                    opcao = input(menu_simular)
                    if opcao == "S":
                        print(f"A taxa de juros selic anual é:\n{taxa_juros}")
                        valor_em = float(input("Informe o valor do empréstimo: "))
                        prazo_pagamento = float(input("Informe em quantos meses deseja pagar: "))
                        if valor_em > 0 and valor_em <= MARGEM_FINANCIAMENTO_CLIENTE and prazo_pagamento <= LIMITE_EM_MESES:
                            valor_parcelas = (valor_em * ((((prazo_pagamento / 12) * taxa_juros) / 100) + 1)) / prazo_pagamento
                            VALOR_COMPROMETIDO_SALARIO = salario / 2
                            if valor_parcelas <= VALOR_COMPROMETIDO_SALARIO:
                                valor_parcelas_cliente = valor_parcelas
                                print(f"O emprestimo de: {valor_em} foi aprovado, com {prazo_pagamento} parcelas de R$ {valor_parcelas_cliente}")
                                if valor_em > 0 and prazo_pagamento > 0 and valor_parcelas_cliente > 0:
                                    menu_em = """
                                        [C] Confirmar emprestimo!
                                        [D] Deixar para depois!
                                        => """ 
                                    opcao = input(menu_em)
                                    if opcao == "C":
                                        saldo += valor_em
                                        print(f"Voce fez o emprestimo de R$ {valor_em} as parcelas de R$ {valor_parcelas_cliente} seram automaticamente recolhidas mes a mes somente aguardar o deposito do valor {valor_em}")
                                    elif opcao == "D":
                                        print(f"Certo deixar para depois!")
                                        return
                                else: 
                                    print("Ocorreu um erro inesperado tente novamente!")   
                            elif valor_parcelas > VALOR_COMPROMETIDO_SALARIO:
                                adequando_parcelas = (valor_parcelas / VALOR_COMPROMETIDO_SALARIO)
                                prazo_pagamento_adequacao = (prazo_pagamento * adequando_parcelas)
                                if prazo_pagamento_adequacao <= LIMITE_EM_MESES:
                                    valor_parcelas = (valor_em * ((((prazo_pagamento_adequacao / 12) * taxa_juros) / 100) + 1)) / prazo_pagamento_adequacao
                                    print(f"O emprestimo de: {valor_em} foi aprovado, com {prazo_pagamento_adequacao} com parcelas de R$ {valor_parcelas}")
                                    if valor_em > 0 and prazo_pagamento > 0 and valor_parcelas_cliente > 0:
                                        menu_em = """
                                        [C] Confirmar emprestimo!
                                        [D] Deixar para depois!
                                        => """ 
                                        opcao = input(menu_em)
                                        if opcao == "C":
                                            saldo += valor_em
                                            print(f"Voce fez o emprestimo de R$ {valor_em} as parcelas de R$ {valor_parcelas_cliente} seram automaticamente recolhidas mes a mes somente aguardar o deposito do valor {valor_em}")
                                        elif opcao == "D":
                                            print(f"Certo deixar para depois!")
                                            return
                                        else:
                                            print("Voce deve escolher uma opcao")
                                            return
                                    else: 
                                        print("Ocorreu um erro inesperado tente novamente!") 
                                elif prazo_pagamento_adequacao > LIMITE_EM_MESES:
                                    print("Infelizmente não foi possível emprestimo nestas condições por favor verificar o valor de cada parcela, não pode ultrapassar metade do seu salário;")
                                else:
                                    print("Não sabemos o que ocorreu, por favor tente novamente!")      
                        else:
                            print("Você deve informar um valor para o empréstimo")
                    elif opcao == "P":
                        print("Voce escolheu sair, saindo do aplicativo!")
                        return
                elif calculadora_perfil < 1350:
                    print(f"Infelizmente você não tem margem")
                    if numero_saques >1:    
                        calculadora_perfil = (score / numero_saques) * LIMITE_SAQUES
                        if calculadora_perfil >= 2700:
                            MARGEM_FINANCIAMENTO_CLIENTE = (1.25 * MARGEM_FINANCIAMENTO)
                            print(f"Sua Margem é:{MARGEM_FINANCIAMENTO_CLIENTE}")
                            menu_simular= """
                                [S] Simular
                                [P] Parar
                            => """
                            opcao = input(menu_simular)
                            if opcao == "S":
                                print(f"A taxa de juros selic anual é:\n{taxa_juros}")
                                        
                                valor_em = float(input("Informe o valor do empréstimo: "))
                                prazo_pagamento = float(input("Informe em quantos meses deseja pagar: "))
                                if valor_em > 0 and valor_em <= MARGEM_FINANCIAMENTO_CLIENTE and prazo_pagamento <= LIMITE_EM_MESES:
                                    valor_parcelas = (valor_em * ((((prazo_pagamento / 12) * taxa_juros) / 100) + 1)) / prazo_pagamento
                                    VALOR_COMPROMETIDO_SALARIO = salario / 2
                                    if valor_parcelas <= VALOR_COMPROMETIDO_SALARIO:
                                        valor_parcelas_cliente = valor_parcelas
                                        print(f"O emprestimo de: {valor_em} foi aprovado, com {prazo_pagamento} parcelas de R$ {valor_parcelas_cliente}")
                                        if valor_em > 0 and prazo_pagamento > 0 and valor_parcelas_cliente > 0:
                                            menu_em = """
                                            [C] Confirmar emprestimo!
                                            [D] Deixar para depois!
                                            => """ 
                                            opcao = input(menu_em)
                                            if opcao == "C":
                                                saldo += valor_em
                                                print(f"Voce fez o emprestimo de R$ {valor_em} as parcelas de R$ {valor_parcelas_cliente} seram automaticamente recolhidas mes a mes somente aguardar o deposito do valor {valor_em}")
                                            elif opcao == "D":
                                                print(f"Certo deixar para depois!")
                                                return
                                            else:
                                                print("Voce deve escolher uma opcao")
                                                return 
                                        else: 
                                            print("Ocorreu um erro inesperado tente novamente!")                                              
                                    elif valor_parcelas > VALOR_COMPROMETIDO_SALARIO:
                                        adequando_parcelas = (valor_parcelas / VALOR_COMPROMETIDO_SALARIO)
                                        prazo_pagamento_adequacao = (prazo_pagamento * adequando_parcelas)
                                        if prazo_pagamento_adequacao <= LIMITE_EM_MESES:
                                            valor_parcelas = (valor_em * ((((prazo_pagamento_adequacao / 12) * taxa_juros) / 100) + 1)) / prazo_pagamento_adequacao
                                            print(f"O emprestimo de: {valor_em} foi aprovado, com {prazo_pagamento_adequacao} com parcelas de R$ {valor_parcelas}")
                                            if valor_em > 0 and prazo_pagamento > 0 and valor_parcelas_cliente > 0:
                                                menu_em = """
                                                [C] Confirmar emprestimo!
                                                [D] Deixar para depois!
                                                => """ 
                                                opcao = input(menu_em)
                                                if opcao == "C":
                                                    saldo += valor_em
                                                    print(f"Voce fez o emprestimo de R$ {valor_em} as parcelas de R$ {valor_parcelas_cliente} seram automaticamente recolhidas mes a mes somente aguardar o deposito do valor {valor_em}")
                                                elif opcao == "D":
                                                    print(f"Certo deixar para depois!")
                                                    return
                                                else:
                                                    print("Voce deve escolher uma opcao")
                                                    return 
                                            else: 
                                                print("Ocorreu um erro inesperado tente novamente!") 
                                        elif prazo_pagamento_adequacao > LIMITE_EM_MESES:
                                            print("Infelizmente não foi possível emprestimo nestas condições por favor verificar o valor de cada parcela, não pode ultrapassar metade do seu salário;")
                                    else:
                                        print("Não sabemos o que ocorreu, por favor tente novamente!")      
                                else:
                                    print("Você deve informar um valor para o empréstimo")
                            elif opcao == "P":
                                print("Voce escolheu sair, saindo do aplicativo!")
                                return
                            else:
                                print("Voce deve escolher um opcao")
                                return            
                elif calculadora_perfil >= 1350:
                    MARGEM_FINANCIAMENTO_CLIENTE = MARGEM_FINANCIAMENTO * 0.75
                    print(f"Sua Margem é:{MARGEM_FINANCIAMENTO_CLIENTE}")
                    menu_simular= """
                        [S] Simular
                        [P] Parar
                    => """
                    opcao = input(menu_simular)
                    if opcao == "S":
                        print(f"A taxa de juros selic anual é:\n{taxa_juros}")
                        valor_em = float(input("Informe o valor do empréstimo: "))                            
                        prazo_pagamento = float(input("Informe em quantos meses deseja pagar: "))
                        if valor_em > 0 and valor_em <= MARGEM_FINANCIAMENTO_CLIENTE and prazo_pagamento <= LIMITE_EM_MESES:
                            valor_parcelas = (valor_em * ((((prazo_pagamento / 12) * taxa_juros) / 100) + 1)) / prazo_pagamento
                            VALOR_COMPROMETIDO_SALARIO = salario / 2
                            if valor_parcelas <= VALOR_COMPROMETIDO_SALARIO:
                                valor_parcelas_cliente = valor_parcelas
                                print(f"O emprestimo de: {valor_em} foi aprovado, com {prazo_pagamento} parcelas de R$ {valor_parcelas_cliente}")
                                if valor_em > 0 and prazo_pagamento > 0 and valor_parcelas_cliente > 0:
                                    menu_em = """
                                    [C] Confirmar emprestimo!
                                    [D] Deixar para depois!
                                    => """ 
                                    opcao = input(menu_em)
                                    if opcao == "C":
                                        saldo += valor_em
                                        print(f"Voce fez o emprestimo de R$ {valor_em} as parcelas de R$ {valor_parcelas_cliente} seram automaticamente recolhidas mes a mes somente aguardar o deposito do valor {valor_em}")
                                    elif opcao == "D":
                                        print(f"Certo deixar para depois!")
                                        return
                                    else:
                                        print("Voce deve escolher uma opcao")
                                        return 
                                else: 
                                    print("Ocorreu um erro inesperado tente novamente!")
                            elif valor_parcelas > VALOR_COMPROMETIDO_SALARIO:
                                adequando_parcelas = (valor_parcelas / VALOR_COMPROMETIDO_SALARIO)
                                prazo_pagamento_adequacao = (prazo_pagamento * adequando_parcelas)
                                if prazo_pagamento_adequacao <= LIMITE_EM_MESES:
                                    valor_parcelas = (valor_em * ((((prazo_pagamento_adequacao / 12) * taxa_juros) / 100) + 1)) / prazo_pagamento_adequacao
                                    print(f"O emprestimo de: {valor_em} foi aprovado, com {prazo_pagamento_adequacao} com parcelas de R$ {valor_parcelas}")
                                    if valor_em > 0 and prazo_pagamento > 0 and valor_parcelas_cliente > 0:
                                        menu_em = """
                                        [C] Confirmar emprestimo!
                                        [D] Deixar para depois!
                                        => """ 
                                        opcao = input(menu_em)
                                        if opcao == "C":
                                            saldo += valor_em
                                            print(f"Voce fez o emprestimo de R$ {valor_em} as parcelas de R$ {valor_parcelas_cliente} seram automaticamente recolhidas mes a mes somente aguardar o deposito do valor {valor_em}")
                                        elif opcao == "D":
                                            print(f"Certo deixar para depois!")
                                            return
                                        else:
                                            print("Voce deve escolher uma opcao")
                                            return 
                                    else: 
                                        print("Ocorreu um erro inesperado tente novamente!")
                                elif prazo_pagamento_adequacao > LIMITE_EM_MESES:
                                    print("Infelizmente não foi possível emprestimo nestas condições por favor verificar o valor de cada parcela, não pode ultrapassar metade do seu salário;")
                            else:
                                print("Não sabemos o que ocorreu, por favor tente novamente!")      
                        else:
                            print("Você deve informar um valor para o empréstimo")
                    if opcao == "P":
                        print("Voce escolheu sair, saindo do aplicativo!")
                        return
                elif calculadora_perfil < 1350:
                    print(f"Infelizmente você não tem margem")
            else:
                print(f"Algo inesperado aconteceu! Tente Novamente!")
        else:
            print("Voce não tem score!")
    else:
        print("Você não tem margem alguma, precisa comprovar renda!")
def exibe_extrato():
    global data_formatada
    global agencia
    global conta
    global hora
    global minutos
    global dia
    global mes
    global ano
    print("\n================ EXTRATO =================")
    print(f"Data do extrato: {dia}/{mes}/{ano}  \t  {hora} : {minutos}")
    print(f"Agencia: {agencia}\t                Conta: {conta}")
    print(f"Cliente: {cliente_extrato}")
    print("Não foram realizadas movimentações." if not extrato else extrato)
    print(f"\nSaldo: R$ {saldo:.2f}")
    print("==========================================")
def main():
    informa_dados()
    global extrato 
    extrato = ""
    global saldo
    global limite
    limite = 500
    global LIMITE_SAQUES
    LIMITE_SAQUES = 3
    
    if agencia == 1 and conta == 1:
        print("Bem vindo Administrador:")
        while True:
            opcao = menu_admin()
            print(f"({opcao})")
            if opcao == "N" or opcao == "n":
                return cadastra_novos_clientes()
            elif opcao == "A" or opcao == "A":
                print("Altera Cadastro de Cliente")
            elif opcao == "E" or opcao == "e":
                return consulta_clientes()
            elif opcao == "NV.AG" or opcao == "nv.ag" or opcao == "Nv.ag":
                print("Cadastra Nova Agencia")
            elif opcao == "M" or opcao == "m":
                print("Exibe as metas")
            elif opcao == "q" or opcao == "Q":
                print("Saindo")
                break
            else:
                print("Operação inválida, por favor selecione novamente a operação desejada.")
    elif agencia > 0 and conta > 1:
        while True:    
            conn = conecta_bd()
            minha_conta(agencia, conta)
            #if encontrado is not None:            
            print("Bem vindo ao Banco Santander, Internet Banking versão 2005")
        
            if conn is not None:
                valido = True
                if valido is True:
                    opcao = exibir_menu() 
                    if opcao == "d":
                        deposito()
                    elif opcao == "s":
                        saque()
                    elif opcao == "c":
                        credito_financiamento()
                    elif opcao == "e":
                        exibe_extrato()
                    elif opcao == "q":
                        break
                else:
                    print("Operação inválida, por favor selecione novamente a operação desejada.")
            else:
                print(f"Ocorreu uma falha, por favor tente novamente.")
                break     
    else:
       print("Agencia não cadastrada!")
main() 
