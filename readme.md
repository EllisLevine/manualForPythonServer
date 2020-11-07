# Multi-threaded Server and Single-threaded Client

## How to start?

## Instructions

1. Starts server by entering the following command: `python3 server.py`.

2. Enter the IP adress for the server.

3. Enter the port you which to connect to.

4. Start the Client by entering the following command: `python3 client.py`.

5. Enter the IP address for the server (same as in server.py).

6. Enter the port for the server (same as in server.py).

## Server `server.py`

This is the server program that simulates a session when connected to a client.

### Importing Libraries

```python
import socket
import threading
import time
import datetime
import subprocess
```

### Setting up variables

```python
LOCALHOST = socket.gethostname()
PORT = 2250
flag_ip = False
flag_port=False
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
```

### Input validation

```python
print("enter IP address:")
Ip = input()

if (Ip == "127.0.0.1"):
    flag_ip = True

while (flag_ip == False):
    print("Wrong IP Address")
    print("enter IP address Again:")
    Ip = input()
    if (Ip == "127.0.0.1"):
        flag_ip = True


print("\nEnter Port Number: ")
Port = int(input())

if int(Port) >= 1025 and int(Port) <= 4998:
    flag_port = True

while (flag_port == False):
    print("\nWrong Port Number")
    print("Enter Port Number Again: ")
    Port = int(input())
    if int(Port) >= 1025 and int(Port) <= 4998:
        flag_port = True

server.bind((Ip, Port))

print("\nServer started")
print("Waiting for client(s) request.. \n")

clientReqs = []
opr = ""
```

### Event loop

```python
while True:
    server.listen()
    newsock = server.accept()[0]
    print("client connected from server side")

    clientReqs.append(newsock)

    for clientsock in clientReqs:
        data = clientsock.recv(1024)
        opr = data.decode()

        if opr == "exit":
            print ("\nClient disconnected...")
            server.close()
            break


        elif opr == "datetime":
            timestamp = datetime.datetime.now().strftime("%d/%m/%Y, %I:%M:%S %p")
            clientsock.send(timestamp.encode())


        elif opr == "uptime":
            res = subprocess.check_output('uptime',shell=True)
            clientsock.send(res)

        elif opr == "memory":
            res = subprocess.check_output('free -h',shell=True)
            clientsock.send(res)

        elif opr == "netstat":
            res = subprocess.check_output('netstat',shell=True)
            clientsock.send(res)

        elif opr == "users":
            res = subprocess.check_output('w',shell=True)
            clientsock.send(res)

        elif opr == "processes":
            res = subprocess.check_output('ps',shell=True)
            clientsock.send(res)

        clientsock.close()
        clientReqs.pop(0)

    if opr == "exit":
        break
```

## Client `client.py`

This is the client program that simulates multiple session when connected to a server.It also returns the following data:

- Running time - Time it took for one single request
- Total time - Time for all client requests to be finished
- Average time - Average time for each cleint request
- Command output - The output for each command

### Setting up libraries

```python
import socket
import threading
from threading import Lock
import time

```

### Clients Thread

```python
client_runtime = []
SERVER = 0
class ClientThread(threading.Thread):
    def __init__(self, outdata):
        threading.Thread.__init__(self)
        client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.csocket = client
        mutex.acquire()
        self.csocket.connect((SERVER, PORT))

        start =  time.time()*1000
        print("\nclient connected")
        self.csocket.sendall(bytes(out_data, 'UTF-8'))

        if out_data == 'exit':
            self.csocket.close()
            return None

        print (self.csocket.recv(8192).decode())
        self.csocket.close()

        end = time.time()*1000

        print("running time: ",  round(end-start, 4), " ms")
        client_runtime.append(end - start)

        mutex.release()
        return None
```

### Boolean values

```python

flag_command = False
flag_ip = False
flag_port = False

```

### Event loop

```python
while(True):

    mutex = Lock()

    # available commands
    my_command_list = ["processes", "memory", "netstat", "users", "datetime",  "uptime", "exit"]

    print("\nEnter command:")
    print("\nprocesses | memory | netstat | users | datetime | uptime |exit")
    out_data = input()

    # checking if command is valid
    for i in range(0, len(my_command_list)):
        if(my_command_list[i] == out_data):
            print("\ncommand found")
            flag_command = True

    while(flag_command == False):
        print("\nthis command is not in list")
        print("Enter Command from list: ")
        print("\nprocesses | memory | netstat | users | datetime | uptime | exit")
        out_data = input()

    for i in range(0, len(my_command_list)):
        if(my_command_list[i] == out_data):
            flag_command = True


    if(out_data == "exit"):
        if SERVER == 0:
            break

        client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        client.connect((SERVER, PORT))
        client.sendall(bytes(out_data, 'UTF-8'))
        break
```

### Validating IP address and PORT(change to your specs)

```python
    print("\nenter server IP address:")
    SERVER = input()

    if(SERVER == "127.0.0.1"):
        flag_ip = True

    while(flag_ip == False):
        print("\nWrong server IP Address")
        print("\nenter server IP address Again:")
        SERVER = input()
        if(SERVER == "127.0.0.1"):
            flag_ip = True

    print("\nEnter Port Number: ")
    PORT = int(input())

    if int(PORT) >= 1025 and int(PORT) <= 4998:
        flag_port = True

    while(flag_port == False):
        print("\nWrong Port Number")
        print("Enter Port Number Again: ")
        Port = int(input())
        if int(PORT) >= 1025 and int(Port) <= 4998:
            flag_port = True

    # getting required data
    print("\nEnter number of clients:")
    clientNum = int(input())

    threads = []

    # multiple threads
    for i in range(0, clientNum):
        newthread = ClientThread(out_data)
        threads.append(newthread)
        newthread.start()

    for x in threads:
        x.join()

    print("\nTotal running time: ", round(sum(client_runtime), 4), " ms")
    print("Avg running time: ", round(sum(client_runtime) /
          len(client_runtime), 4), " ms")

    client_runtime = []
```
