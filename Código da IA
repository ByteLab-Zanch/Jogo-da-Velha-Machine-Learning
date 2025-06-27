import serial
import time

# Configura a conexão serial
porta_usb = "/dev/ttyACM0"  # Ou COMx no Windows
baud_rate = 9600
arduino = serial.Serial(porta_usb, baud_rate, timeout=0.1)
time.sleep(2)

tabuleiro = [' '] * 9  # 0 a 8

def imprimir_tabuleiro():
    simbolos = [c if c != ' ' else str(i+1) for i, c in enumerate(tabuleiro)]
    print("\n")
    for i in range(0, 9, 3):
        print(f"{simbolos[i]} | {simbolos[i+1]} | {simbolos[i+2]}")
        if i < 6:
            print("--+---+--")
    print("\n")

def receber_jogada_do_arduino():
    if arduino.in_waiting > 0:
        linha = arduino.readline().decode().strip()
        if linha.startswith("JOGADA_LOCAL:"):
            try:
                pos = int(linha.split(":")[1])
                if 0 <= pos < 9 and tabuleiro[pos] == ' ':
                    tabuleiro[pos] = 'J'
                    return True
            except:
                pass
    return False

def verificar_vitoria(simbolo):
    combinacoes = [
        (0,1,2), (3,4,5), (6,7,8),
        (0,3,6), (1,4,7), (2,5,8),
        (0,4,8), (2,4,6)
    ]
    for a, b, c in combinacoes:
        if tabuleiro[a] == tabuleiro[b] == tabuleiro[c] == simbolo:
            return True
    return False

def tabuleiro_cheio():
    return all(c != ' ' for c in tabuleiro)

def minimax(board, profundidade, eh_ia):
    if verificar_vitoria('I'):
        return 10 - profundidade
    if verificar_vitoria('J'):
        return profundidade - 10
    if tabuleiro_cheio():
        return 0

    if eh_ia:
        melhor_valor = -float('inf')
        for i in range(9):
            if board[i] == ' ':
                board[i] = 'I'
                valor = minimax(board, profundidade + 1, False)
                board[i] = ' '
                melhor_valor = max(melhor_valor, valor)
        return melhor_valor
    else:
        pior_valor = float('inf')
        for i in range(9):
            if board[i] == ' ':
                board[i] = 'J'
                valor = minimax(board, profundidade + 1, True)
                board[i] = ' '
                pior_valor = min(pior_valor, valor)
        return pior_valor

def jogada_ia():
    melhor_valor = -float('inf')
    melhor_jogada = None

    for i in range(9):
        if tabuleiro[i] == ' ':
            tabuleiro[i] = 'I'
            valor = minimax(tabuleiro, 0, False)
            tabuleiro[i] = ' '
            if valor > melhor_valor:
                melhor_valor = valor
                melhor_jogada = i

    if melhor_jogada is not None:
        tabuleiro[melhor_jogada] = 'I'
        comando = f"I{melhor_jogada}\n"
        arduino.write(comando.encode())
        arduino.flush()

def resetar():
    global tabuleiro
    tabuleiro = [' '] * 9
    arduino.write(b"RESET\n")
    arduino.flush()

print("Aguardando jogadas no Arduino...")

try:
    while True:
        if receber_jogada_do_arduino():
            imprimir_tabuleiro()

            if verificar_vitoria('J'):
                print("Jogador venceu!")
                arduino.write(b"WIN_HUMANO\n")
                arduino.flush()
                time.sleep(3)
                resetar()
                continue

            if tabuleiro_cheio():
                print("Empate!")
                arduino.write(b"EMPATE\n")
                arduino.flush()
                time.sleep(3)
                resetar()
                continue

            # Turno da IA (agora com Minimax agressivo)
            jogada_ia()
            time.sleep(1)
            imprimir_tabuleiro()

            if verificar_vitoria('I'):
                print("IA venceu!")
                arduino.write(b"WIN_IA\n")
                arduino.flush()
                time.sleep(3)
                resetar()
            elif tabuleiro_cheio():
                print("Empate!")
                arduino.write(b"EMPATE\n")
                arduino.flush()
                time.sleep(3)
                resetar()

except KeyboardInterrupt:
    print("\nEncerrado pelo usuário.")
    arduino.close()
