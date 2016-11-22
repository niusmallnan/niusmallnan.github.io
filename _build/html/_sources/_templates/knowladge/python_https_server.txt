=======================================
python实现精简的https服务器
=======================================
有时我们在处理客户问题时，需要快速模拟客户的运行环境，比如客户的服务的HTTPS的，我们需要测试这种协议在我们的网络上是否会有问题。

首先生成证书::

    # 生成rsa密钥
    $ openssl genrsa -des3 -out server.key 1024
    # 去除掉密钥文件保护密码
    $ openssl rsa -in server.key -out server.key
    # 生成ca对应的csr文件
    $ openssl req -new -key server.key -out server.csr
    # 自签名
    $ openssl x509 -req -days 1024 -in server.csr -signkey server.key -out server.crt
    $ cat server.crt server.key > server.pem

python运行代码如下::
    
    #!/usr/bin/python

    import BaseHTTPServer, SimpleHTTPServer
    import ssl

    httpd = BaseHTTPServer.HTTPServer(('localhost', 4443), SimpleHTTPServer.SimpleHTTPRequestHandler)
    httpd.socket = ssl.wrap_socket (httpd.socket, certfile='/path/to/server.pem', server_side=True)
    httpd.serve_forever()







