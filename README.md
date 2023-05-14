# Kerberos
# Authentification Openssh avec Kerberos
Ceci montre comment s'authentifier avec l'outil de connectivité openssh en utilisant Gasspi et Kerberos pour activer SSO sans mot de passe.

## C'est quoi le Kerberos 
Kerberos est un protocole d'authentification réseau largement utilisé qui fournit une authentification et une autorisation sécurisées pour les clients et les serveurs dans un environnement informatique distribué. Il utilise un système de ticket pour vérifier les identités et permet l'authentification mutuelle entre les clients et les serveurs.
![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/blob/main/Kerberos.png) 
# Architecture de Kerberos
L'architecture Kerberos se compose de trois composants principaux : le client, le serveur et le serveur d'authentification Kerberos (KDC). Le KDC est responsable de l'émission et de la vérification des tickets, tandis que le client et le serveur utilisent ces tickets pour s'authentifier et accéder aux ressources réseau en toute sécurité. Le protocole utilise la cryptographie à clef symétrique et l'authentification mutuelle pour fournir la vérification d'identité sécurisée sur les systèmes distribués.
![App Screenshot](https://github.com/JawherBenjabeur/Kerberos/blob/main/Kerberos%20arch.png)
