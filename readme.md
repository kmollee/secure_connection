# Go: secure connection

take a note from [secure-connections](https://github.com/lizrice/secure-connections)

example: go using SSL or TTL througn connection


## SSL? TTL?

HTTP over (SSL or TTL): HTTPS

HTTP(HyperText Transfer Protocol) 超文本傳輸協定


SSL(Secure Sockets Layer) 安全通訊協定 and TLS(Transport Layer Security) 傳輸層安全性協定
常見的名詞 但有時搞不清楚有何差別， 基本上是同一個協定不過 TLS 是 SSL 的下一代, 換個稱呼

目的是為網際網路通訊，提供安全及資料完整性保障。

further more: [WIKI](https://en.wikipedia.org/wiki/Transport_Layer_Security)

## public key? private key?

- Public key 公鑰: can be  freely distributed and is used to encrypt 可以自由散播, 用來加密
- Private key 私鑰: msut be kept private and is used to decrypt 私人擁有, 拿來解密 或 來做 簽 數位簽章


傳輸

用 public key 加密後 只有 private 解的開, 所以能保證這段的安全性

```
"Hello" -> public key -> (encrypted)  -> private key -> (decrypt) -> "Hello"
```

### Public/private key signatures

- Private key: must be kept private and is used to sign message 簽署簽章
- Public Key: is used to virfy signature 驗證數位簽章


簽章

```
"this is 100 from my bank" -> (private key) -> sign or encrypt -> pubic key -> verify
```

不過這些都還需要第三方來保證這個 Public key 真的是該持有人擁有, 所有就有了 Certificate Authority(CA)

## CA

X.509 certificate

- Subject name
- Subject's public key
- Issuer(CA) name
- Validity

Certificate sign by issuer (CA)



1. 建立 Private Key
2. 產生憑證簽署要求 – Certificate Signing Request(CSR)
    CSR 的原理就像你用文字檔打了一份關於你公司/個人資料的文件，然後把 Public Key（公開金鑰）跟這份文件一起壓縮程一個檔案，傳給憑證中心，憑證中心解開後，就可以看到你的資料跟 Public Key，他會驗證身份資料，透過自動驗證、紙本或是電話（在”種類 – 分類 1 “的部份有提到）的方式來驗證，沒問題後，就會把你的 Public Key 進行簽署，然後把簽署後的資料傳回給你，這就是最終憑證。

    ```sh
    openssl req -nodes -newkey rsa:2048 -sha256 -keyout domain.key -out domain.csr
    ```

    中間需要填寫一些資訊，**千萬不要亂填**

    產生完 我們有

    1. private key
    2. csr 憑證資料(contain public key and owner information)

3. 安裝憑證

    憑證中心收到你的 CSR 檔之後，就會進行驗證、收費，一段時間後就會把憑證簽發給你，這時候你就可以安裝憑證。

    安裝憑證的時候你只需要兩個（或三個）東西，你的私有金鑰、憑證中心簽發給你的憑證，這兩個是必備，憑證中心可能會再給一個中繼憑證(pem)，所以是三個：

    最後走完 CA 認證流程我們有

    1. Private Key
    2. csr 憑證資料(contain public key and owner information)
    3. 中繼憑證 ca_name.crt or ca_name.pem



    SSLCertificateKeyFile 指向你的私有金鑰檔案位置
    SSLCertificateFile 指向你的憑證檔案位置
    SSLCACertificateFile 指向你的中繼憑證檔案位置



而在 CA server 的部分，這屬於第三方公證單位的運用，要介紹的重點在於：

1. 建置 CA server
2. 簽發憑證、報廢憑證
3. 設定 CA 簽證的 policy

CA server 種類

首先有些觀念必須說明，即是 CA server 的種類有兩種：

1. Self-Signed CA
    亦可稱為 root CA。由於此種 CA 所發的憑證(certificate)是不經由任何上層 CA 所認證，而是以「自行認證」的方式進行認證。因此像是最上層的商業 CA，或是自行架設內部認證用的 CA，都可以屬於此類。
2. Signed CA
    不 同於 Self-Signed CA，此種 CA 所發佈的憑證，可被上層的 CA 進行認證，而兩種 CA 的關係則是「Parent CA <==> Child CA」。 而通常設定上層 CA 時，除非是內部使用，不然使用商業 CA 是必須付費的!



## CLI tools

- openssl
    -see contents of certificate: openssl x509 -text
    ```
    openssl x509 -text -in ./cert.pem
    ```
- cfssl
    - comprehensive toolkit
- mkcert
    - local development
    - install CA into your system and browser
- minica
    - easy genration of key and certs
  
    ```
    go get -v github.com/jsha/minica
    ```


what the hell are all these `.crt`, `.key`, `.csr` and `.pem` files?

- `.pem` (Private-enhanced Electionic Mail) Base64 encoded DER certificate, enclosed between `------BEGIN CERTIFICATE---` and `----END CERTIFICATE----`
- `.cer`, `.crt`, `.der` 通常是 DER format, 但是 base64 encode certicates 也很常用

## 開始建立連線

### server side need

- a private key
- a certificate for your identity

`ListenAndServeTLS(cert, keky)`
    - or `TLSConfig.Certificates`
    - or `TLSConfig.GetCertificate`

### client side need

- a private key
- a certificate for your identity
- a ca, add into pool, to verify cert from server

- tls.Dial
    - or make HTTP request to `https`
- My need to add CA  cert to `TLSConfig.RootCAs`
- TLSConfig.InsecureSkipVerify
    - don't check server's certificate


# TODO

- [ ] add dockerfile



# refer

- [Certificate Authority(CA) 憑證簡介](http://mistech.pixnet.net/blog/post/80751019-certificate-authority%28ca%29-%E6%86%91%E8%AD%89%E7%B0%A1%E4%BB%8B)
- [私有金鑰、CSR 、CRT 與 中繼憑證](https://blog.rsync.tw/ssl-key-csr-crt-pem/)
- [secure-connections](https://github.com/lizrice/secure-connections)
- [How does a public key verify a signature?](https://stackoverflow.com/questions/18257185/how-does-a-public-key-verify-a-signature)




