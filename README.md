# Communication-client-serveur-s-curis-e-avec-mTLS-en-Python
# Introduction
Dans cet atelier, nous utilisons mTLS (mutual TLS), c’est-à-dire une authentification mutuelle.
Contrairement au TLS classique où seul le serveur présente un certificat, ici le serveur et le client
présentent chacun un certificat valide avant d’échanger des données.

# 1. Préparation du dossier de travail
<img width="366" height="166" alt="image" src="https://github.com/user-attachments/assets/830bcac3-71b1-460b-ac0a-2f384fd699d1" />

# 2. Génération des certificats avec mkcert
Le certificat et la clé privé ont été généré par le client. 

# 3. Développement du serveur mTLS
Dans ce TP fait en binôme, j'ai été le serveur.
```python
import socket
import ssl

HOST = "0.0.0.0"
PORT = 8643

# Contexte TLS du serveur
context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
context.load_cert_chain(certfile="server.crt", keyfile="server.key")

# Le serveur vérifie le certificat du client
context.load_verify_locations(cafile="rootCA.pem")
context.verify_mode = ssl.CERT_REQUIRED

# Création du socket serveur
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind((HOST, PORT))
server_socket.listen(1)

print(f"Serveur mTLS en attente sur {HOST}:{PORT} ...")

# Attente d'une seule connexion
client_socket, client_address = server_socket.accept()
print("Client connecté :", client_address)

# Passage en TLS mutuel
mtls_socket = context.wrap_socket(client_socket, server_side=True)

# Affichage du certificat du client
client_cert = mtls_socket.getpeercert()
print("Certificat du client :")
print(client_cert)

# Envoi d'un seul message
message = "Bonjour client, connexion mTLS établie."
mtls_socket.send(message.encode())

# Réception d'une seule réponse
response = mtls_socket.recv(2048)
print("Réponse du client :", response.decode())

# Fermeture
mtls_socket.close()
server_socket.close()

print("Connexion terminée.")
```
# 4. Résultat attendu
<img width="1382" height="267" alt="image" src="https://github.com/user-attachments/assets/9d52400a-2bc4-4ef5-b247-2f62d9a99b04" />

# 5. Questions de compréhension
- 1. Quelle différence faites-vous entre TLS classique et mTLS ?
La différence est que dans TLDS, seule le client verifie le certificat du serveur alors que dans mTLS, il y a un contrôle du client vers le serveur et le serveur vers le client

- 2. Pourquoi le serveur charge-t-il rootCA.pem ?
 Il le charge pour verifier le certificat du client

- 3. Pourquoi le serveur charge-t-il rootCA.pem ?
 Il le charge pour verifier le certificat du serveur

- 4. Quel est le rôle de client.crt et client.key ?
     client.crt: correspond au certificat numérique du client
     client.key: correspond à la clé privé du client
- 5. Pourquoi la connexion échoue-t-elle si le client ne présente pas de certificat ?
Parce que dans le code du serveur on a dit que le certificat du client était requis
- 6. Pourquoi check_hostname = True reste-t-il utile côté client ?
Car il doit s'assurer qu'il est connecté au serveur 


