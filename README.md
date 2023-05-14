# Kerberos
# Authentification Openssh avec Kerberos
Ceci montre comment s'authentifier avec l'outil de connectivité openssh en utilisant Gasspi et Kerberos pour activer SSO sans mot de passe.

## C'est quoi le Kerberos 
Kerberos est un protocole d'authentification réseau largement utilisé qui fournit une authentification et une autorisation sécurisées pour les clients et les serveurs dans un environnement informatique distribué. Il utilise un système de ticket pour vérifier les identités et permet l'authentification mutuelle entre les clients et les serveurs.
![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/blob/main/Kerberos.png) 
# Architecture de Kerberos
L'architecture Kerberos se compose de trois composants principaux : le client, le serveur et le serveur d'authentification Kerberos (KDC). Le KDC est responsable de l'émission et de la vérification des tickets, tandis que le client et le serveur utilisent ces tickets pour s'authentifier et accéder aux ressources réseau en toute sécurité. Le protocole utilise la cryptographie à clef symétrique et l'authentification mutuelle pour fournir la vérification d'identité sécurisée sur les systèmes distribués.

![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/blob/main/Kerberos_arch.png)

# Openssh
OpenSSH est une implémentation libre et open-source du protocole Secure Shell (SSH) utilisé pour accéder en toute sécurité aux serveurs et appareils distants. Il fournit une communication cryptée entre le client et le serveur, permettant le transfert de données sécurisé et l’exécution de commandes à distance. OpenSSH comprend une suite d’outils, notamment ssh, scp et sftp, qui permettent aux utilisateurs de se connecter en toute sécurité à des serveurs distants, de transférer des fichiers et de gérer des systèmes distants.

## Requirements

**Servers:** KDC + Service (Ubuntu), Client (Ubuntu)

**Tools:**  Kerberos , Openssh server

## Dns Configuration

Dans cette section, nous allons créer notre **DNS (espace de nom de domaine)** afin de connecter les deux serveurs *KDC/Service et Client* Machines en utilisant des adresses IP.

Dans mon cas :
* KDC server (Virtual Machine) *IP address*: **192.168.88.133**

* Client server (Virtual Machine)*IP address*: **192.168.88.134**

1. Setting Hostname for each machine:

**KDC Machine**
```bash
hostnamectl --static set-hostname kdc.uc.tn 

![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/blob/main/Kerberos/preConfig/changing%20hastname%20of%20Machine%202%20to%20Client.png)

**Client Machine**
```bash
hostnamectl --static set-hostname client.uc.tn

```
![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/blob/main/Kerberos/preConfig/changing%20hastname%20of%20Machine%202%20to%20Client.PNG)

Now we are going to create the Dns by some changes in the */etc/hosts* file using the command
```bash
sudo nano /etc/hosts

```

now we are going to add the information below:
```bash
<KDC_IP_ADDRESS>    kdc.uc.tn       kdc
<CLIENT_ IP_ADDRESS>    client.uc.tn    client


```
![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/blob/main/Kerberos/preConfig/Dns%20Machine1.PNG)


## KDC Machine Configuration
In This section we are going to configure our **key destribution center** starting by those two commands:
```bash
$ sudo apt-get update
$ sudo apt-get install krb5-kdc krb5-admin-server krb5-config
```
Some configuration will be displayed :

1. configuration of the realm : 'UC.TN' *(must be all uppercase)*

![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/kdc/realm%20config.PNG)

2.  configuration of the Kerberos server : 'kdc.uc.tn' 

![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/kdc/Serveur%20kdc%20config.PNG)

3.  configuration of the Admin server : 'kdc.uc.tn'

![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/kdc/admin%20server%20config.PNG)

4. initialization the KDC database

Inorder to initialize the KDC database we need to set the master key using the command:

```bash
sudo krb5_newrealm

![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/kdc/intialize%20UC.tn%20Database.PNG)

4. Adding principles

+ root/admin principal

To manage the users and services in a Kerberos realm, they are defined as principals. An administrator user must be created manually to manage these principals.

```bash
    sudo kadmin.local
    kadmin.local:  add_principal root/admin
```
![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/kdc/adding%20root_admin%20principle.PNG)

 To provide full access to the Kerberos database, the admin principal *`root/admin`* must be granted all access rights using the **`/etc/krb5kdc/kadm5.acl`** configuration file.

```bash
    sudo nano /etc/krb5kdc/kadm5.acl
```

  In this file we need to add this ligne 
  
  ```bash
    */admin@INSAT.TN    *
```
![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/kdc/krb5acl.PNG)

now , we need to restart krb5-admin-server using the command 

  ```bash
   sudo service krb5-admin-server restart
```
+ Adding the User principle

```bash
    sudo kadmin.local
    kadmin.local:  add_principal user
```

![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/kdc/adding%20user%20principle.PNG)

+ Adding host principle

 ```bash
    sudo kadmin.local
    kadmin.local:  add_principal host/kdc.uc.tn
```

![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/kdc/adding%20host%20principle.PNG)

**configurating keytab in kdc machine**

In this section we are going to the Keytab inorder to authenticate users and services without requiring users to enter their passwords.

* Adding the admine principel to keytab 
* 
 ```bash
      $ ktutil 
   ktutil:  add_entry -password -p root/admin@UC.TN -k 1 -e aes256-cts-hmac-sha1-96
   Password for postgres/pg.insat.tn@INSAT.TN: 
   ktutil:  wkt postgres.keytab
```

![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/kdc/configure%20keytab%20adding%20admin%20principle.PNG)

* Adding host principel to keytab

```bash
      $ ktutil 
   ktutil:  ktadd host/kdc.uc.tn
```

![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/kdc/adding%20host%20principle%20to%20the%20key%20tab.PNG)

* Adding other principels to keytab 

```bash
      $ ktutil 
   ktutil: ktadd -k /etc/krb5kdc/kadm5.keytab kadmin/admin
   ktutil: ktadd -k /etc/krb5kdc/kadm5.keyteb kadmin/changepw
```
![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/kdc/adding%20some%20principle%20for%20keytab%20in%20the%20new%20version.PNG)

* list of entries in the keytab 

```bash
      $ ktutil 
   ktutil: rkt /etc/krb5kdc/kadim5.keytab
   ktutil: l
```
![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/kdc/vifing%20kadmin%20principel%20is%20added%20to%20keyfile.PNG)

**Configuration of Openssh server**

First we are going to start by installing the openssh-server using those commands:

```bash
   sudo apt-get update
   sudo apt-get install openssh-server
```
now we need to enable Gassapi which is the *Generic Security Services Application* Programming Interface, which is a standard interface for securely exchanging authentication and authorization data between networked applications.

In order to enable the Gassapi we need to change  the configuration file in */etc/ssh/ssh_config*

```bash
   sudo nano /etc/ssh/ssh_config
```
![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/kdc/configure%20ssh-config%20by%20enabling%20GassAPI.PNG)

## KDC Machine Configuration

In This section we are going to configure our **Client Machine** starting by those two commands:


```bash
$ sudo apt-get update
  sudo apt-get install krb5-user libpam-krb5 libpam-ccreds
```

Like the kdc machine some configuration will be displayed:
* the realm : 'UC.TN' (must be all uppercase)
* the Kerberos server : 'kdc.uc.tn'
* the administrative server : 'kdc.uc.tn'


**configuration of Openssh-server in the client**

```bash
   sudo apt-get update
   sudo apt-get install openssh-server
```
then the configuration file in */etc/ssh/ssh_config*

![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/client/changing%20config%20file%20for%20openssh.PNG)

**Creating a User**

In this section we are going to create a user that will use the openssh service

```bash
   sudo adduser user
```

![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/client/adding%20a%20user.PNG)

**Generating a ticket for user**
 
 In this section we are going to generate a ticket to user which is essentially a cryptographic token that contains the user's identity
  using those commands :
```bash
   kinit 
   klist 
```

*kinit*: in order to initalize the ticket 
*keylist*: in order to display the key's list

![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/client/generating%20a%20ticket%20granting%20ticket%20(TGT).PNG)

+ Now the user can authentficate to the *kdc.uc.tn* machine without a password

![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/tree/main/Kerberos/client/connecting%20without%20password.png)

## Acknowledgements

 - [Usefull video: Teckwall](https://www.youtube.com/watch?v=vx2vIA2Ym14&list=RDCMUC_wmzB9ziSW5KbZxyDfZV_w&index=1)
 - [Usefull github repo : yosra270](https://github.com/yosra270/postgresql-auth-with-kerberos/blob/main/README.md)
 
