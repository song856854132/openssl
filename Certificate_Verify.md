# Delta certificate chain verify
###### tags: `62443`

> This article is aimed at focusing on the reliability of openssl verifying the certificates. In openssl source code, it was quite complex machanism to verify cert from buildup chain to verify chain and print out at the end. Unfortunately, in present, we cannot trace and explain the code line by line to prove the capability of verifying the certificates.
>![](https://i.imgur.com/WlT9d0k.jpg)
> So it worth a complete doc to prove we can actually verify signature in  certificate by issuer's public key that already done by openssl. 


## deltagrid-iot-dev.deltaww.com
### First, obtain the certificate chain of deltagrid-iot-dev.deltaww.com
```shell=
$ openssl s_client -showcerts -connect deltagrid-iot-dev.deltaww.com:443                                    130 ⨯
CONNECTED(00000003)
depth=2 C = US, ST = New Jersey, L = Jersey City, O = The USERTRUST Network, CN = USERTrust RSA Certification Authority
verify return:1
depth=1 C = GB, ST = Greater Manchester, L = Salford, O = Sectigo Limited, CN = Sectigo RSA Organization Validation Secure Server CA
verify return:1
depth=0 C = TW, postalCode = 11491, L = Taipei City, street = "No. 186, Ruiguang Rd.,", O = Delta Electronics inc., OU = IT, CN = *.deltaww.com
verify return:1
---
Certificate chain
 0 s:C = TW, postalCode = 11491, L = Taipei City, street = "No. 186, Ruiguang Rd.,", O = Delta Electronics inc., OU = IT, CN = *.deltaww.com
   i:C = GB, ST = Greater Manchester, L = Salford, O = Sectigo Limited, CN = Sectigo RSA Organization Validation Secure Server CA
-----BEGIN CERTIFICATE-----
MIIHHTCCBgWgAwIBAgIRAIEhZhbRSS4uSt5TAeLTAKwwDQYJKoZIhvcNAQELBQAw
gZUxCzAJBgNVBAYTAkdCMRswGQYDVQQIExJHcmVhdGVyIE1hbmNoZXN0ZXIxEDAO
BgNVBAcTB1NhbGZvcmQxGDAWBgNVBAoTD1NlY3RpZ28gTGltaXRlZDE9MDsGA1UE
AxM0U2VjdGlnbyBSU0EgT3JnYW5pemF0aW9uIFZhbGlkYXRpb24gU2VjdXJlIFNl
cnZlciBDQTAeFw0yMDA4MDMwMDAwMDBaFw0yMjEwMjIyMzU5NTlaMIGaMQswCQYD
VQQGEwJUVzEOMAwGA1UEERMFMTE0OTExFDASBgNVBAcTC1RhaXBlaSBDaXR5MR8w
HQYDVQQJExZOby4gMTg2LCBSdWlndWFuZyBSZC4sMR8wHQYDVQQKExZEZWx0YSBF
bGVjdHJvbmljcyBpbmMuMQswCQYDVQQLEwJJVDEWMBQGA1UEAwwNKi5kZWx0YXd3
LmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMKb8cWJQzrhLD4X
Yhvwezv+wy4G7H5cENNSUvI91NNwh1z0BoEINnm+WEBgdeYC0AQivJSn+7Oupt6g
jWPMlDgSSQT5+dphe2Yy/z+lU5b+E4nn7aQ8y0IYWVOhQPOp5pIKaRz27cuiE/Kt
fwTPGh/te0vVhbXCkrEzCaQFgck9ja/WLwGL+R0vFxmGjSl2iWm42HA/62Zr/wr/
kliBd7zJcObSW5K12JpCT9NdkdZfiQhrsmOIU6XXMrr3DsbFl0+MeDF7jCkgcWXt
0PqHEq7+3+w8gBnKgEhm8auJ5N8h9AmekzqDxc5NXre+mQt8N7D2NlVDM0YrUYLS
bcHQsyUCAwEAAaOCA18wggNbMB8GA1UdIwQYMBaAFBfZ1iUnZ/kxwklD2TA2RIxs
qU/rMB0GA1UdDgQWBBSkUZpvMHxUVG3VE1e6V9v/t2Dr7zAOBgNVHQ8BAf8EBAMC
BaAwDAYDVR0TAQH/BAIwADAdBgNVHSUEFjAUBggrBgEFBQcDAQYIKwYBBQUHAwIw
SgYDVR0gBEMwQTA1BgwrBgEEAbIxAQIBAwQwJTAjBggrBgEFBQcCARYXaHR0cHM6
Ly9zZWN0aWdvLmNvbS9DUFMwCAYGZ4EMAQICMFoGA1UdHwRTMFEwT6BNoEuGSWh0
dHA6Ly9jcmwuc2VjdGlnby5jb20vU2VjdGlnb1JTQU9yZ2FuaXphdGlvblZhbGlk
YXRpb25TZWN1cmVTZXJ2ZXJDQS5jcmwwgYoGCCsGAQUFBwEBBH4wfDBVBggrBgEF
BQcwAoZJaHR0cDovL2NydC5zZWN0aWdvLmNvbS9TZWN0aWdvUlNBT3JnYW5pemF0
aW9uVmFsaWRhdGlvblNlY3VyZVNlcnZlckNBLmNydDAjBggrBgEFBQcwAYYXaHR0
cDovL29jc3Auc2VjdGlnby5jb20wJQYDVR0RBB4wHIINKi5kZWx0YXd3LmNvbYIL
ZGVsdGF3dy5jb20wggF+BgorBgEEAdZ5AgQCBIIBbgSCAWoBaAB3AEalVet1+pEg
MLWiiWn0830RLEF0vv1JuIWr8vxw/m1HAAABc7M63BkAAAQDAEgwRgIhAJBM/713
3wqY4PzXdiuy5Eoe+VYbQE1ss5HCMdMLg7EFAiEAjP40+qEVCrtpJ91c24v4i+Dp
KKDod6nuaaIni+OoyfwAdgDfpV6raIJPH2yt7rhfTj5a6s2iEqRqXo47EsAgRFwq
cwAAAXOzOt1qAAAEAwBHMEUCIQCIvQdQY+iakz/LIH3dqnPQLwirJ7u8qcCX3eBQ
ub88vwIgYG/y/vEwlR0Wyl2xAQFx94ix6oAxH0NjwmChb5gbjNEAdQBvU3asMfAx
GdiZAKRRFf93FRwR2QLBACkGjbIImjfZEwAAAXOzOtwSAAAEAwBGMEQCICylRVoz
JmsJaCmM051dYPS0WuAXc5kQ+NIMdZwtZb8pAiBTMHPnnbiJqglc9LN0kdQGHxaP
O+Yew1CHFCv2/VXV1TANBgkqhkiG9w0BAQsFAAOCAQEANTWJX/meGfTgKdSjBChY
V03YtQ2qJfpOOd1q7NX0G9kKqZawVDzxIw5jYOIJTM/fNreaZBp7pks7FiyV3j78
PBSCfnRfrgMvsYTwBZM6kbSSWGF0RpSKtXADqegcnwM4Chaxd9oaIe50k3PhRaRK
2vjYn+h9O16dkuGrpMimtX5k10rYLphqQltdMeMHg3sw60dYG40voei+4VYNqeav
zYQTmA7jraCL2C/XpRKMFZs2qi8pPOCL9pU10IIJ4OSV9psCDA1pkqnrkRDrh3of
kzUMsJeKEuNGOdGqbFcIjr89mU1SW6A74vzgca7ZFPzo5A85m05IzZx+txsd5rJW
7Q==
-----END CERTIFICATE-----
 1 s:C = GB, ST = Greater Manchester, L = Salford, O = Sectigo Limited, CN = Sectigo RSA Organization Validation Secure Server CA
   i:C = US, ST = New Jersey, L = Jersey City, O = The USERTRUST Network, CN = USERTrust RSA Certification Authority
-----BEGIN CERTIFICATE-----
MIIGGTCCBAGgAwIBAgIQE31TnKp8MamkM3AZaIR6jTANBgkqhkiG9w0BAQwFADCB
iDELMAkGA1UEBhMCVVMxEzARBgNVBAgTCk5ldyBKZXJzZXkxFDASBgNVBAcTC0pl
cnNleSBDaXR5MR4wHAYDVQQKExVUaGUgVVNFUlRSVVNUIE5ldHdvcmsxLjAsBgNV
BAMTJVVTRVJUcnVzdCBSU0EgQ2VydGlmaWNhdGlvbiBBdXRob3JpdHkwHhcNMTgx
MTAyMDAwMDAwWhcNMzAxMjMxMjM1OTU5WjCBlTELMAkGA1UEBhMCR0IxGzAZBgNV
BAgTEkdyZWF0ZXIgTWFuY2hlc3RlcjEQMA4GA1UEBxMHU2FsZm9yZDEYMBYGA1UE
ChMPU2VjdGlnbyBMaW1pdGVkMT0wOwYDVQQDEzRTZWN0aWdvIFJTQSBPcmdhbml6
YXRpb24gVmFsaWRhdGlvbiBTZWN1cmUgU2VydmVyIENBMIIBIjANBgkqhkiG9w0B
AQEFAAOCAQ8AMIIBCgKCAQEAnJMCRkVKUkiS/FeN+S3qU76zLNXYqKXsW2kDwB0Q
9lkz3v4HSKjojHpnSvH1jcM3ZtAykffEnQRgxLVK4oOLp64m1F06XvjRFnG7ir1x
on3IzqJgJLBSoDpFUd54k2xiYPHkVpy3O/c8Vdjf1XoxfDV/ElFw4Sy+BKzL+k/h
fGVqwECn2XylY4QZ4ffK76q06Fha2ZnjJt+OErK43DOyNtoUHZZYQkBuCyKFHFEi
rsTIBkVtkuZntxkj5Ng2a4XQf8dS48+wdQHgibSov4o2TqPgbOuEQc6lL0giE5dQ
YkUeCaXMn2xXcEAG2yDoG9bzk4unMp63RBUJ16/9fAEc2wIDAQABo4IBbjCCAWow
HwYDVR0jBBgwFoAUU3m/WqorSs9UgOHYm8Cd8rIDZsswHQYDVR0OBBYEFBfZ1iUn
Z/kxwklD2TA2RIxsqU/rMA4GA1UdDwEB/wQEAwIBhjASBgNVHRMBAf8ECDAGAQH/
AgEAMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjAbBgNVHSAEFDASMAYG
BFUdIAAwCAYGZ4EMAQICMFAGA1UdHwRJMEcwRaBDoEGGP2h0dHA6Ly9jcmwudXNl
cnRydXN0LmNvbS9VU0VSVHJ1c3RSU0FDZXJ0aWZpY2F0aW9uQXV0aG9yaXR5LmNy
bDB2BggrBgEFBQcBAQRqMGgwPwYIKwYBBQUHMAKGM2h0dHA6Ly9jcnQudXNlcnRy
dXN0LmNvbS9VU0VSVHJ1c3RSU0FBZGRUcnVzdENBLmNydDAlBggrBgEFBQcwAYYZ
aHR0cDovL29jc3AudXNlcnRydXN0LmNvbTANBgkqhkiG9w0BAQwFAAOCAgEAThNA
lsnD5m5bwOO69Bfhrgkfyb/LDCUW8nNTs3Yat6tIBtbNAHwgRUNFbBZaGxNh10m6
pAKkrOjOzi3JKnSj3N6uq9BoNviRrzwB93fVC8+Xq+uH5xWo+jBaYXEgscBDxLmP
bYox6xU2JPti1Qucj+lmveZhUZeTth2HvbC1bP6mESkGYTQxMD0gJ3NR0N6Fg9N3
OSBGltqnxloWJ4Wyz04PToxcvr44APhL+XJ71PJ616IphdAEutNCLFGIUi7RPSRn
R+xVzBv0yjTqJsHe3cQhifa6ezIejpZehEU4z4CqN2mLYBd0FUiRnG3wTqN3yhsc
SPr5z0noX0+FCuKPkBurcEya67emP7SsXaRfz+bYipaQ908mgWB2XQ8kd5GzKjGf
FlqyXYwcKapInI5v03hAcNt37N3j0VcFcC3mSZiIBYRiBXBWdoY5TtMibx3+bfEO
s2LEPMvAhblhHrrhFYBZlAyuBbuMf1a+HNJav5fyakywxnB2sJCNwQs2uRHY1ihc
6k/+JLcYCpsM0MF8XPtpvcyiTcaQvKZN8rG61ppnW5YCUtCC+cQKXA0o4D/I+pWV
idWkvklsQLI+qGu41SWyxP7x09fn1txDAXYw+zuLXfdKiXyaNb78yvBXAfCNP6CH
MntHWpdLgtJmwsQt6j8k9Kf5qLnjatkYYaA7jBU=
-----END CERTIFICATE-----
 2 s:C = US, ST = New Jersey, L = Jersey City, O = The USERTRUST Network, CN = USERTrust RSA Certification Authority
   i:C = US, ST = New Jersey, L = Jersey City, O = The USERTRUST Network, CN = USERTrust RSA Certification Authority
-----BEGIN CERTIFICATE-----
MIIF3jCCA8agAwIBAgIQAf1tMPyjylGoG7xkDjUDLTANBgkqhkiG9w0BAQwFADCB
iDELMAkGA1UEBhMCVVMxEzARBgNVBAgTCk5ldyBKZXJzZXkxFDASBgNVBAcTC0pl
cnNleSBDaXR5MR4wHAYDVQQKExVUaGUgVVNFUlRSVVNUIE5ldHdvcmsxLjAsBgNV
BAMTJVVTRVJUcnVzdCBSU0EgQ2VydGlmaWNhdGlvbiBBdXRob3JpdHkwHhcNMTAw
MjAxMDAwMDAwWhcNMzgwMTE4MjM1OTU5WjCBiDELMAkGA1UEBhMCVVMxEzARBgNV
BAgTCk5ldyBKZXJzZXkxFDASBgNVBAcTC0plcnNleSBDaXR5MR4wHAYDVQQKExVU
aGUgVVNFUlRSVVNUIE5ldHdvcmsxLjAsBgNVBAMTJVVTRVJUcnVzdCBSU0EgQ2Vy
dGlmaWNhdGlvbiBBdXRob3JpdHkwggIiMA0GCSqGSIb3DQEBAQUAA4ICDwAwggIK
AoICAQCAEmUXNg7D2wiz0KxXDXbtzSfTTK1Qg2HiqiBNCS1kCdzOiZ/MPans9s/B
3PHTsdZ7NygRK0faOca8Ohm0X6a9fZ2jY0K2dvKpOyuR+OJv0OwWIJAJPuLodMkY
tJHUYmTbf6MG8YgYapAiPLz+E/CHFHv25B+O1ORRxhFnRghRy4YUVD+8M/5+bJz/
Fp0YvVGONaanZshyZ9shZrHUm3gDwFA66Mzw3LyeTP6vBZY1H1dat//O+T23LLb2
VN3I5xI6Ta5MirdcmrS3ID3KfyI0rn47aGYBROcBTkZTmzNg95S+UzeQc0PzMsNT
79uq/nROacdrjGCT3sTHDN/hMq7MkztReJVni+49Vv4M0GkPGw/zJSZrM233bkf6
c0Plfg6lZrEpfDKEY1WJxA3Bk1QwGROs0303p+tdOmw1XNtB1xLaqUkL39iAigmT
Yo61Zs8liM2EuLE/pDkP2QKe6xJMlXzzawWpXhaDzLhn4ugTncxbgtNMs+1b/97l
c6wjOy0AvzVVdAlJ2ElYGn+SNuZRkg7zJn0cTRe8yexDJtC/QV9AqURE9JnnV4ee
UB9XVKg+/XRjL7FQZQnmWEIuQxpMtPAlR1n6BB6T1CZGSlCBst6+eLf8ZxXhyVeE
Hg9j1uliutZfVS7qXMYoCAQlObgOK6nyTJccBz8NUvXt7y+CDwIDAQABo0IwQDAd
BgNVHQ4EFgQUU3m/WqorSs9UgOHYm8Cd8rIDZsswDgYDVR0PAQH/BAQDAgEGMA8G
A1UdEwEB/wQFMAMBAf8wDQYJKoZIhvcNAQEMBQADggIBAFzUfA3P9wF9QZllDHPF
Up/L+M+ZBn8b2kMVn54CVVeWFPFSPCeHlCjtHzoBN6J2/FNQwISbxmtOuowhT6KO
VWKR82kV2LyI48SqC/3vqOlLVSoGIG1VeCkZ7l8wXEskEVX/JJpuXior7gtNn3/3
ATiUFJVDBwn7YKnuHKsSjKCaXqeYalltiz8I+8jRRa8YFWSQEg9zKC7F4iRO/Fjs
8PRF/iKz6y+O0tlFYQXBl2+odnKPi4w2r78NBc5xjeambx9spnFixdjQg3IM8WcR
iQycE0xyNN+81XHfqnHd4blsjDwSXWXavVcStkNr/+XeTWYRUc+ZruwXtuhxkYze
Sf7dNXGiFSeUHM9h4ya7b6NnJSFd5t0dCy5oGzuCr+yDZ4XUmFF0sbmZgIn/f3gZ
XHlKYC6SQK5MNyosycdiyA5d9zZbyuAlJQG03RoHnHcAP9Dc1ew91Pq7P8yF1m9/
qS3fuQL39ZeatTXaw2ewh0qpKJ4jjv9cJ2vhsE/zB+4ALtRZh8tSQZXq9EfX7mRB
VXyNWQKV3WKdwrnuWih0hKWbt5DHDAff9Yk2dDLWKMGwsAvgnEzDHNb842m1R0aB
L6KCq9NjRHDEjf8tM7qtj3u1cIiuPhnPQCjY/MiQu12ZIvVS5ljFH4gxQ+6IHdfG
jjxDah2nGN59PRbxYvnKkKj9
-----END CERTIFICATE-----
---
Server certificate
subject=C = TW, postalCode = 11491, L = Taipei City, street = "No. 186, Ruiguang Rd.,", O = Delta Electronics inc., OU = IT, CN = *.deltaww.com

issuer=C = GB, ST = Greater Manchester, L = Salford, O = Sectigo Limited, CN = Sectigo RSA Organization Validation Secure Server CA

---
No client certificate CA names sent
Peer signing digest: SHA512
Peer signature type: RSA
Server Temp Key: ECDH, P-256, 256 bits
---
SSL handshake has read 5608 bytes and written 457 bytes
Verification: OK
---
New, TLSv1.2, Cipher is ECDHE-RSA-AES256-GCM-SHA384
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES256-GCM-SHA384
    Session-ID: BE0BAAC89844D1435EFEACFDB2947075B7436BF8FC7B59D6AF391CA5EA75CCB7
    Session-ID-ctx: 
    Master-Key: 7342CD395ED061E7C141CE8E08C8B6ED7BF2A85C5D59AF7836AFB2C0DAFD8D0C55D029FF26AD20C105B6460D187A1797
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 300 (seconds)
    TLS session ticket:
    0000 - 21 1f 04 71 c4 83 64 cc-89 a7 e5 d6 94 eb 87 53   !..q..d........S
    0010 - cb e3 5b 36 7b 26 f0 8b-18 17 de 1a c1 a4 d1 ae   ..[6{&..........
    0020 - 90 1a 7c 5d 01 c9 b2 2d-d3 c7 49 61 6e 50 88 08   ..|]...-..IanP..
    0030 - a6 01 d7 cb 1e 84 3f 83-2d 7f 4c 58 3b be 33 d5   ......?.-.LX;.3.
    0040 - 17 b6 a7 e0 f8 d7 76 75-80 99 7c 31 1e 98 21 2e   ......vu..|1..!.
    0050 - dd c8 e8 dc 03 b3 c6 52-58 b0 ac 92 52 70 30 3e   .......RX...Rp0>
    0060 - ce ff 0a 9b 24 e8 07 d0-65 e9 44 1e fd 1c 6e 0f   ....$...e.D...n.
    0070 - b7 06 6c 09 91 70 56 c4-0e c0 c2 6b 94 17 00 25   ..l..pV....k...%
    0080 - d8 19 6f 94 b8 c4 74 49-f4 87 69 55 e8 7e 8e 40   ..o...tI..iU.~.@
    0090 - d4 1f e4 4f 3e c6 db 87-ce c1 9e 8a 0b fe 9c 29   ...O>..........)
    00a0 - c7 9a 18 bc 7c e9 a5 e6-76 f1 a2 d2 1f b6 d6 1d   ....|...v.......
    00b0 - 21 9b b3 ae af 15 fa f6-af 8e ca 40 53 25 02 2b   !..........@S%.+
    00c0 - 7f 62 83 00 51 06 eb 0c-be d1 b7 58 87 27 ac ea   .b..Q......X.'..

    Start Time: 1644320380
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
    Extended master secret: no
```
### Make it into 3 disdinct certs
```shell=
$ ls
delta_end.crt -> End-entity cert
intermediate.crt -> Intermeidate cert
rootCA.crt -> Root cert
```
![](https://i.imgur.com/RhZNoad.png)
And we are going to verify the **root CA's signature in Intermidiate cert** by using **root CA public key**.

### Get the public key of issuer(Root CA)
```shell=
$ openssl x509 -in rootCA.crt -noout -pubkey > rootCA_pub.pem 
$ cat rootCA_pub.pem 
-----BEGIN PUBLIC KEY-----
MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAgBJlFzYOw9sIs9CsVw12
7c0n00ytUINh4qogTQktZAnczomfzD2p7PbPwdzx07HWezcoEStH2jnGvDoZtF+m
vX2do2NCtnbyqTsrkfjib9DsFiCQCT7i6HTJGLSR1GJk23+jBvGIGGqQIjy8/hPw
hxR79uQfjtTkUcYRZ0YIUcuGFFQ/vDP+fmyc/xadGL1RjjWmp2bIcmfbIWax1Jt4
A8BQOujM8Ny8nkz+rwWWNR9XWrf/zvk9tyy29lTdyOcSOk2uTIq3XJq0tyA9yn8i
NK5+O2hmAUTnAU5GU5szYPeUvlM3kHND8zLDU+/bqv50TmnHa4xgk97Exwzf4TKu
zJM7UXiVZ4vuPVb+DNBpDxsP8yUmazNt925H+nND5X4OpWaxKXwyhGNVicQNwZNU
MBkTrNN9N6frXTpsNVzbQdcS2qlJC9/YgIoJk2KOtWbPJYjNhLixP6Q5D9kCnusS
TJV882sFqV4Wg8y4Z+LoE53MW4LTTLPtW//e5XOsIzstAL81VXQJSdhJWBp/kjbm
UZIO8yZ9HE0XvMnsQybQv0FfQKlERPSZ51eHnlAfV1SoPv10Yy+xUGUJ5lhCLkMa
TLTwJUdZ+gQek9QmRkpQgbLevni3/GcV4clXhB4PY9bpYrrWX1Uu6lzGKAgEJTm4
Diup8kyXHAc/DVL17e8vgg8CAwEAAQ==
-----END PUBLIC KEY-----
```

### Get the signature part in binary(Intermidiate Cert)
```shell=
$ SIGNATURE_INTERMEDIATE=$(openssl x509 -in intermediate.crt -text -noout -certopt ca_default -certopt no_validity -certopt no_serial -certopt no_subject -certopt no_extensions -certopt no_signame | grep -v 'Signature Algorithm' | tr -d '[:space:]:')
$ cat $SIGNATURE_INTERMEDIATE 
cat: 4e134096c9c3e66e5bc0e3baf417e1ae091fc9bfcb0c2516f27353b3761ab7ab4806d6cd007c204543456c165a1b1361d749baa402a4ace8cece2dc92a74a3dcdeaeabd06836f891af3c01f777d50bcf97abeb87e715a8fa305a617120b1c043c4b98f6d8a31eb153624fb62d50b9c8fe966bde661519793b61d87bdb0b56cfea6112906613431303d20277351d0de8583d37739204696daa7c65a162785b2cf4e0f4e8c5cbebe3800f84bf9727bd4f27ad7a22985d004bad3422c5188522ed13d246747ec55cc1bf4ca34ea26c1deddc42189f6ba7b321e8e965e844538cf80aa37698b6017741548919c6df04ea377ca1b1c48faf9cf49e85f4f850ae28f901bab704c9aebb7a63fb4ac5da45fcfe6d88a9690f74f268160765d0f247791b32a319f165ab25d8c1c29aa489c8e6fd3784070db77ecdde3d15705702de64998880584620570567686394ed3226f1dfe6df10eb362c43ccbc085b9611ebae1158059940cae05bb8c7f56be1cd25abf97f26a4cb0c67076b0908dc10b36b911d8d6285cea4ffe24b7180a9b0cd0c17c5cfb69bdcca24dc690bca64df2b1bad69a675b960252d082f9c40a5c0d28e03fc8fa959589d5a4be496c40b23ea86bb8d525b2c4fef1d3d7e7d6dc43017630fb3b8b5df74a897c9a35befccaf05701f08d3fa087327b475a974b82d266c2c42dea3f24f4a7f9a8b9e36ad91861a03b8c15: File name too long
$ echo ${SIGNATURE_INTERMEDIATE} | xxd -r -p > intermediate_sig.bin

```
### Decrypt the signature with issuer public key, and we get hash
```shell=
$ openssl rsautl -verify -inkey rootCA_pub.pem -in intermediate_sig.bin -pubin > intermediate_DECsig.bin 
$ openssl asn1parse -inform DER -in intermediate_DECsig.bin 
    0:d=0  hl=2 l=  65 cons: SEQUENCE          
    2:d=1  hl=2 l=  13 cons: SEQUENCE          
    4:d=2  hl=2 l=   9 prim: OBJECT            :sha384
   15:d=2  hl=2 l=   0 prim: NULL              
   17:d=1  hl=2 l=  48 prim: OCTET STRING      [HEX DUMP]:5D2164164EB6F3820B9B8D7A5601B60EBF70D832D2029C6C3C966DB107DFCB1BA12A25700F26CA3D5B36E4A6CB576CFA
```

### Obtain the cert hash (excluding RSA signature part)
```shell=
$ openssl asn1parse -in intermediate.crt -strparse 4 -out intermediate_body.bin
$ openssl dgst -sha384 intermediate_body.bin 
SHA384(intermediate_body.bin)= 5d2164164eb6f3820b9b8d7a5601b60ebf70d832d2029c6c3c966db107dfcb1ba12a25700f26ca3d5b36e4a6cb576cfa
```

### Summary
Compare both hash value, we can officially said that it's signed by the acutal rootCA of Delta:100:.
However, the fact was openssl s_client have already done this for us after connecting to target server. There was a series action behind it, from build chain to internal verify.
![](https://i.imgur.com/keCNpqR.png)

![](https://i.imgur.com/PjYleoO.png)

![](https://i.imgur.com/hAVW9R4.png)

![](https://i.imgur.com/suG4QSA.png)

![](https://i.imgur.com/yAcNWuV.png)

![](https://i.imgur.com/9MgFGMu.png)


