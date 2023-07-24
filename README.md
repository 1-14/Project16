# Project16

## sm2 2P decrypt简介

### 结构

![image](https://github.com/1-14/Project16/blob/main/2.png)

原理如上图所示

## 关键代码展示

### 参数设定

```
#参数设定
q = "8542D69E4C044F18E8B92435BF6FF7DE457283915C45517D722EDB8B08F1DFC3"
a = "787968B4FA32C3FD2417842E73BBFEFF2F3C848B6831D7E0EC65228B3937E498"
b = "63E4C6D3B23B0C849CF84241484BFE48F61D59A5B16BA06E6E12D1DA27C5249A"
x_G = "421DEBD61B62EAB6746434EBC3CC315E32220B3BADD50BDC4C4E6C147FEDD43D"
y_G = "0680512BCBB42C07D47349D2153B70C4E5D7FDFCBFA36EA1A85841B9E46E09A2"
G = (x_G, y_G)
n = "8542D69E4C044F18E8B92435BF6FF7DD297720630485628D5AE74EE7C32E79B7"
#私钥
d_1 = hex(random.randint(1, int_hex(n) - 1))[2:]
d_2 = hex(random.randint(1, int_hex(n) - 1))[2:]
d_1_inv = inv(int_hex(d_1), int_hex(n))
d_2_inv = inv(int_hex(d_2), int_hex(n))
d = hex((d_1_inv * d_2_inv - 1))[2:]
P = mul_ECC(G, int_hex(d))
v = 256

C1 = ('3c8465b81aa67d75c07555024bc42b885e106b493afc70e7d2a3371a80fda3be',
      'e570c12918ff58744d2251a18cf6371ef56c9d714f8abc6e8d70af1bf73f760')
C2 = '706a1b7699ebb2be70063653162459bd1'
C3 = '757ce752698fb9faf84a8deaababceecf817d48590e91317565e9aa71dc2459e'
```


### Alice

Alice在解密过程中，接收到Bob发送的密文C，首先根据C1的横纵坐标长度截取出C1的横纵坐标，然后进行ECC点运算验证C1是否合法。接下来，Alice使用自己的私钥d_1对C1进行点乘得到x2和y2，并根据x2和y2计算KDF生成的密钥t。
通过对C2和t进行异或运算，Alice得到解密后的明文M__，然后使用x2、M__和y2计算C3并与接收到的C3进行比较，从而验证消息的完整性。

```
if __name__ == '__main__':
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_address = ('localhost', 8000)
    server_socket.bind(server_address)

    server_socket.listen(1)
    print('Waiting for connecting...')

    client_socket, client_addr = server_socket.accept()
    print('Bob has connected!')

    #step1
    d_1 = d_1

    #step2
    d_1_inv = inv(int_hex(d_1), int_hex(n))
    T_1 = mul_ECC(C1, d_1_inv)
    client_socket.sendall(T_1[0].encode())
    time.sleep(2)
    client_socket.sendall(T_1[1].encode())

    #step3
    x = client_socket.recv(2048).decode()
    y = client_socket.recv(2048).decode()
    T_2 = (x, y)

    #step4
    y_inv = hex(-int_hex(C1[1]))[3:]
    C1_inv = (C1[0], y_inv)
    x_2, y_2 = add_ECC(T_2, C1_inv)

    klen = 132
    t = KDF(x_2 + y_2, klen)

    M__ = hex(int_hex(C2) ^ int_hex(t))[2:]

    tmp = x_2 + M__ + y_2
    u = sm3_hash(tmp)
    print(f'M: {M__}')

    client_socket.close()
    server_socket.close()
```

### Bob

Bob生成一个随机数d_2，然后将Alice发送的T_1点与d_2的逆元进行点乘运算得到T_2，并将T_2的横纵坐标分别发送给Alice。

```
if __name__ == '__main__':
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_address = ('localhost', 8000)

    print('Connecting to Alice...')

    try:
        client_socket.connect(server_address)
        print('Connect to Alice successfully!')
    except Exception:
        print('Failed to connect to Alice!')
        sys.exit()

    #step1
    d_2 = d_2

    #step2
    x = client_socket.recv(2048).decode()
    y = client_socket.recv(2048).decode()
    T_1 = (x, y)

    #step3
    d_2_inv = inv(int_hex(d_2), int_hex(n))
    T_2 = mul_ECC(T_1, d_2_inv)
    client_socket.sendall(T_2[0].encode())
    time.sleep(2)
    client_socket.sendall(T_2[1].encode())

    client_socket.close()
```

## 结果展示

![image](https://github.com/1-14/Project16/blob/main/1.png)










