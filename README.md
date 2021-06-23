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

## nosql

```
use ascii char #\Nul

set [timeout,key,val]
    timeout 0  is without timeout   
            60 is with a minute timeout  
    key and val is string
send: "set\x0020\x00abc\x00def"
back: "0[onionice✅]:t\r\n"

let [timeout0,key0,val0,timeout1,key1,val1,timeout2,key2,val2,......]
    set multi kv at once
send: "let\x000\x00123\x00456\x000\x00231\x00564\x000\x00312\x00645"
back: "0[onionice✅]:t\r\n"

get [key]
    get the val in string
    "NIL" means no value
send: "get\x00123"
back: "456\r\n"

got [key0,val0,key1,val1,key3,val3,......]
    got multi val at once
    ......\r\n
    ......\r\n
    ......\r\n
    \r\n
send: "got\x00123\x00231\x00312"
back: "456\r\n564\r\n645\r\n\r\n"

del [key]
    delete the key and val
send: "del\x00123"
back: "0[onionice✅]:t\r\n"

rem [key0,val0,key1,val1,key2,val2,......]
    delete multi kv
send: "rem\x00123\x00231\x00312"
back: "0[onionice✅]:t\r\n"

"see"

"push"  
"take"   

"math"   
"more"    
"less"    

```

