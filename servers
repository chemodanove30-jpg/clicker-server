import socket
import threading
import os

# Render сам передает нужный порт через переменную окружения PORT
HOST = '0.0.0.0'
PORT = int(os.environ.get("PORT", 8888))

rooms = {} # {code: [socket1, socket2]}

def handle_client(conn, addr):
    current_room = None
    try:
        while True:
            data = conn.recv(1024).decode('utf-8')
            if not data:
                break
            
            lines = data.strip().split('\n')
            for line in lines:
                parts = line.split(' ')
                cmd = parts[0]

                if cmd == "CREATE":
                    code = parts[1]
                    rooms[code] = [conn]
                    current_room = code
                    conn.sendall(b"CREATED\n")

                elif cmd == "JOIN":
                    code = parts[1]
                    if code in rooms and len(rooms[code]) < 2:
                        rooms[code].append(conn)
                        current_room = code
                        conn.sendall(b"JOINED\n")
                        for s in rooms[code]:
                            s.sendall(b"PLAYER_JOINED\n")
                    else:
                        conn.sendall(b"ERROR_FULL_OR_NOT_FOUND\n")

                elif cmd == "HIT":
                    if current_room and current_room in rooms:
                        for s in rooms[current_room]:
                            if s != conn:
                                s.sendall(b"HIT\n")

    except:
        pass
    finally:
        if current_room and current_room in rooms:
            if conn in rooms[current_room]:
                rooms[current_room].remove(conn)
            if len(rooms[current_room]) == 0:
                del rooms[current_room]
        conn.close()

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server.bind((HOST, PORT))
server.listen()

print(f"Сервер запущен на порту {PORT}")
while True:
    conn, addr = server.accept()
    threading.Thread(target=handle_client, args=(conn, addr)).start()
