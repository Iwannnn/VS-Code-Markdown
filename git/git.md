git总是报乱七八糟的链接不上的错误 记录一下



错误:```fatal: unable to access 'https://github.com/Iwannnn/multhelp.git/': Encountered end of file```

解决:```git config --global --unset https.proxy```


错误:```OpenSSL SSL_read: Connection was reset, errno 10054 ```

解决:``` git config --global http.sslVerify "false" ```


错误:```OpenSSL SSL_connect: Connection was reset in connection to github.com:443```
结局:感觉是网络的问题，给git设置代理，vpn结局了