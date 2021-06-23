# onionice

## require

* libc 
* libuv

## start server

```bash
# local ipv4 6666
./onionice

# local ipv4 6060
./onionice 127.0.0.1 6060

# local ipv6 6600
./onionice ::1 6600

# other example
./onionice 66.66.66.66 6666
```

## connect

```python
import socket

def recvall (sock):
    BUFF_SIZE = 4096
    msg = b''
    while True:
        part = sock.recv(BUFF_SIZE)
        msg += part
        if len(part) < BUFF_SIZE:
            break
    return msg

def makeclient ():
    global client
    client = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    client.connect(("127.0.0.1",6666))
    return 0

def sendstr (str):
    client.send(str.encode('utf-8'))
    data = recvall(client)
    return data.decode()

makeclient()

message0 = chr(0).join(["set","0","abc","xyz"])
sendstr(message0)

message1 = chr(0).join(["get","abc"])
sendstr(message1)

sendstr("rem"+chr(0)+"abc")
```
