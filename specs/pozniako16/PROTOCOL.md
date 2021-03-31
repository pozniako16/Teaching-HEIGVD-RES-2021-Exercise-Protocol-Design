# 1. Introduction
## 1.1 Objectifs du protocole
Le protocole CALC a eté concu pour que des clients puissent soumettre des opérations
arithméetiques a des serveurs de calcul, au travers d'un réseau TCP/IP. Tout serveur
conforme a cette spécification DOIT (MUST) étre capable de réaliser des additions et des
multiplications. Il PEUT (MAY) offrir des operations supplémentaires. La liste des
opérations supportées par le serveur est fournie au client lors de la connexion.
## 1.2 Fonctionnement général

Le protocole CALC utilise le protocole TCP comme protocole de transport. Le port 3101 a
été choisi comme port par défaut. Apres avoir accepté une demande de connexion, le
serveur envoie la liste des opérations qu'il supporte au client. Le client peut alors
envoyer un nombre arbitraire d'opérations.
Pour chaque operation, il envoie un message COMPUTE dans lequel il fournit une OPERATION
et deux OPERANDES. Quand le serveur recoit un message COMPUTE correct, il fait le calcul
et renvoie un message RESULT en fournissant le résultat. Si ne message recu n'est pas
correct, il envoie un message ERROR en spécifiant la nature de l'‘erreur.
Quand le client n'a plus de calcul a faire, il envoie un message QUIT et ferme la
connexion TCP.
Le protocole CALC est sans état: le serveur n'a pas besoin de garder des données en
 mémoire entre entre le traitement de deux messages envovés par un client.
 
# 2. Syntaxe des messages

Le protocole CALC définit 5 messages: WELCOME, COMPUTE, RESULT, ERROR et QUIT. Le
protocole est un protocole "ligne a ligne” (par opposition a un protocole binaire). Les
clients et les serveurs DOIVENT utiliser la séquence CRLF (ASCII 13, ASCII 1@) pour
indiquer la fin d'une ligne.

## 2.1 WELCOME

Ce message est envoye du serveur au client, immédiatement apres que la connexion TCP ait
eté établie. Le message est constitue de plusieurs lignes. Apres avoir lu la premiere
ligne, le client doit lire une ligne ('- AVAILABLE OPERATIONS'), puis toutes les lignes
jusqu'a ce qu'il rencontre une ligne dont la valeur est '- END OPERATIONS’). Chaque ligne
lue indique le nom d'une opération mathématique supportée par le serveur, suivi d’un
espace, suivi du nombre d’opérandes a envoyer avec 1’opération.

**<ins>Exemple:</ins>**

WELCOME CRLF
- AVAILABLE OPERATIONS CRLF
- ADD 2 CRLF
- MULT 2 CRLF
- DIV 2 CRLF
- END OPERATIONS CRLF

## 2.2 COMPUTE
Ce message est envoye du client au serveur, pour soumettre une operation. Le message est
défini par une opération et deux opérandes. Les valeurs sont séparées par un espace.

**<ins>Exemple</ins>**: COMPUTE ADD 2 45 CRLF

## 2.3 RESULT 

Ce message est envoyé du serveur au client, pour retourner le résultat d'une opération.
Le message est défini par la chaine RESULT, suivie d'un espace et du résultat calculé par
le serveur.

**<ins>Exemple</ins>**: RESULT 47 CRLF

## 2.4 ERROR

Ce message est envoyé du serveur au client, dans le cas ov un message envoyé par celui-ci
ne peut pas étre traité. Le message est défini par la chaine 'ERROR', d'une code d’erreur
et d'une description textuelle de l'erreur. Ces 3 éléments sont séparés par un espace.

**<ins>Exemples</ins>**: ERROR 3@@ UNKNOWN OPERATION CRLF

## 2.5 QUIT

Ce message est envoyé du client vers le serveur, pour indiquer que la session est
terminée.

# 3. Element spécifiques

## 3.1. Opérations supportées

Au minimum, le serveur DOIT implémenter les opérations ADD et MULT, permettant a un
client de soumettre des requétes pour calculer des additions, respectivement des
multiplications.

## 3.2. Extensibilite

Chaque serveur PEUT implémenter des opérations supplémentaires. La liste de toutes les
operations supportées par le serveur est envoyée au client lors de la connexion, dans le
message WELCOME.

## 3.3. Traitement d’erreurs

Si le client envoie une requéte invalide, le serveur DOIT répondre en envoyant un message
ERROR. Les code d'erreur standards suivants sont définis: ‘'30@' (UNKNOWN OPERATION) et
"40@ (SYNTAX ERROR). Le serveur PEUT envoyer un code différent, mais il DOIT alors
commencer par le caractére ‘9’.

# 4. Exemples

S: WELCOME CRLF

S: - OPERATIONS CRLF

S: - ADD 2 CRLF

S: - MULT 2 CRLF

S: - DIV 2 CRLF

S: - ENDOFOPERATIONS CRLF
C: ADD 1 4 CRLF

S: RESULT 5S CRLF

C: ADD 7 9 CRLF

S: RESULT 16 CRLF

C: QUIT CRLF

--------------------------

S: WELCOME CRLF

S: - OPERATIONS CRLF

S: - ADD 2 CRLF

S: - MULT 2 CRLF

S: - ENDOFOPERATIONS CRLF
C: DIV 1 4 CRLF

S: ERROR 30@ UKNOWN OPERATION CRLF

C: KIDKIDKFIKSDIFK CRLF

S: ERROR 400 SYNTAX ERROR CRLF