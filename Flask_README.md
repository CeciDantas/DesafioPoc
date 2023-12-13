from flask import Flask, jsonify, request
import sqlite3

app = Flask(__name__)

#conectar ao banco de dados SQLite
def conectar_bd():
    return sqlite3.connect('dados.db')
def sortFn(dict):
    return dict['N']

# Rota para listar todas as informações
@app.route('/', methods=['GET'])
def listar_informacoes():
    try:
        # Conectar ao banco de dados
        conexao = conectar_bd()
        cursor = conexao.cursor()

        # Consultar o SQL para ver todas as informações
        cursor.execute('SELECT * FROM tabela')
        dados = cursor.fetchall()

        # Transformar para JSON
        resultado = [{'N': linha[0], 'Campo': linha[1], 'Descricao': linha[2], 'Tipo': linha[3], 'Tamanho': linha[4], 'Dec':linha[5], 'Entr': linha[6],'Saida':linha[7]} for linha in dados]

        conexao.close()
        resultado.sort(key=sortFn)

        # Retornar os dados como JSON
        return jsonify({'informacoes': resultado})

    except Exception as e:
        return jsonify({'error': str(e)})
    except sqlite3.Error as e:
        return jsonify({'error': f'Erro no banco de dados: {str(e)}'})


# Rota para deletar registros com base em um critério
@app.route('/deletar', methods=['DELETE'])
def deletar_registros():
    try:
        # Conectar ao banco de dados
        conexao = conectar_bd()
        cursor = conexao.cursor()

        # Verificar se a tabela existe ou não
        cursor.execute("SELECT name FROM sqlite_master WHERE type='table' AND name='tabela'")
        tabela_existe = cursor.fetchone()

        if not tabela_existe:
            return jsonify({'error': 'A tabela não existe'})

        # Obter os critérios
        campo = request.args.get('campo')
        valor = request.args.get('valor')

        # Validar as informações
        if campo is None or valor is None:
            return jsonify({'error': 'Campos obrigatórios ausentes'})

        # Construir o comando SQL para deletar registros
        comando_sql = f"DELETE FROM tabela WHERE {campo} = '{valor}'"

        # Executar o comando SQL
        cursor.execute(comando_sql)

        # Verificar se algum registro foi deletado
        if cursor.rowcount == 0:
            return jsonify({'message': f'Nenhum registro encontrado para deletar com {campo}={valor}'})

        conexao.commit()
        conexao.close()

        # Retornar uma mensagem indicando o sucesso da operação
        return jsonify({'message': f'Informação deletada com sucesso: {campo}={valor}'})

    except sqlite3.Error as e:
        return jsonify({'error': f'Erro no banco de dados: {str(e)}'})
    except Exception as e:
        return jsonify({'error': f'Erro durante a exclusão: {str(e)}'})

# Rota para inserir um novo registro
@app.route('/inserir', methods=['POST'])
def inserir_registro():
    try:
        # Conectar ao banco de dados
        conexao = conectar_bd()
        cursor = conexao.cursor()
        NArg = request.args.get('N')

        #Campos obrigatorios:
        Campo = request.json.get('Campo')
        Dec = request.json.get('Dec')
        Descricao = request.json.get('Descricao')
        Entr = request.json.get('Entr')
        N = request.json.get('N')
        Saida = request.json.get('Saida')
        Tam = request.json.get('Tamanho')
        Tipo = request.json.get('Tipo')

        # Validar os campos obrigatórios
        if N is None:
            return jsonify({'error': 'Campo obrigatório N  não está presente'})

        if NArg is None:

            # Comando SQL para inserir um novo registro
            comando_sql = f"INSERT INTO tabela (Campo, Dec, Descricao, Entr, N , Saida, Tam , Tipo) VALUES ( '{Campo}', '{Dec}', '{Descricao}', '{Entr}', '{N}' , '{Saida}', '{Tam}' , '{Tipo}')"

            # Executar o comando SQL
            cursor.execute(comando_sql)
        else:
            comando_sql = f"UPDATE tabela SET Campo = '{Campo}', Dec = '{Dec}', Descricao = '{Descricao}', Entr = '{Entr}', Saida = '{Saida}', Tam = '{Tam}', Tipo = '{Tipo}' WHERE N = '{N}'"

            cursor.execute(comando_sql)

        conexao.commit()

        conexao.close()

        # Retornar uma mensagem indicando o sucesso da operação
        return jsonify({'message': 'Novo registro inserido com sucesso'})

    except sqlite3.Error as e:
        return jsonify({'error': f'Erro no banco de dados: {str(e)}'})
    except Exception as e:
        return jsonify({'error': f'Erro durante a inserção: {str(e)}'})

if __name__ == '__main__':
    app.run(debug=True)


