from socket import *
import sys  # Necessário para encerrar o programa

# Cria o socket TCP (orientado à conexão)
serverSocket = socket(AF_INET, SOCK_STREAM)

# Prepara o socket do servidor
serverPort = 6789
serverSocket.bind(('', serverPort))   # liga em todas as interfaces
serverSocket.listen(1)                # aceita 1 conexão por vez (fila 1)

print(f"Servidor HTTP simples rodando na porta {serverPort}...")

while True:
    # Estabelece a conexão
    print('Ready to serve...')
    connectionSocket, addr = serverSocket.accept()

    try:
        # Recebe a mensagem do cliente (requisição HTTP)
        message = connectionSocket.recv(1024).decode()
        if not message:
            connectionSocket.close()
            continue

        filename = message.split()[1]
        f = open(filename[1:])   # remove o '/' inicial
        outputdata = f.read()
        f.close()

        # Envia a linha de status do cabeçalho HTTP
        header = "HTTP/1.1 200 OK\r\n"
        header += "Content-Type: text/html; charset=UTF-8\r\n"
        header += f"Content-Length: {len(outputdata.encode())}\r\n"
        header += "\r\n"
        connectionSocket.send(header.encode())

        # Envia o conteúdo do arquivo ao cliente
        connectionSocket.send(outputdata.encode())

        # Fecha a conexão com o cliente
        connectionSocket.close()

    except IOError:
        # Envia mensagem de erro 404 se o arquivo não for encontrado
        response = "HTTP/1.1 404 Not Found\r\n"
        response += "Content-Type: text/html; charset=UTF-8\r\n"
        response += "\r\n"
        response += "<html><body><h1>404 Not Found</h1></body></html>"
        connectionSocket.send(response.encode())

        # Fecha o socket do cliente
        connectionSocket.close()

serverSocket.close()
sys.exit()
