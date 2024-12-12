# Interfaces
ext_if = "em0"          # Interface externe (connexion à Internet via NAT)
admin_if = "em1"        # Interface pour le LAN Administration
server_if = "em2"       # Interface pour le LAN Serveur
employee_if = "em3"     # Interface pour le LAN Employé

# Réseaux
admin_net = "192.168.42.0/26"      # Sous-réseau Administration
server_net = "192.168.42.64/26"    # Sous-réseau Serveur
employee_net = "192.168.42.128/26" # Sous-réseau Employé

# Politique par défaut : tout bloquer
block all

# Permettre le trafic sur l'interface loopback (localhost)
set skip on lo

# NAT : Permettre la sortie des sous-réseaux internes vers Internet
nat on $ext_if from { $admin_net, $server_net, $employee_net } to any nat-to ($ext_if)

# Autoriser le LAN Administration à accéder au LAN Serveur sur tous les ports
pass in on $admin_if from $admin_net to $server_net

# Autoriser le LAN Employé à accéder au LAN Serveur uniquement en HTTP/HTTPS
pass in on $employee_if proto tcp from $employee_net to $server_net port { http https }

# Communication entre sous-réseaux
# Autoriser le LAN Administration à accéder au LAN Serveur et au LAN Employé
pass in on $admin_if from $admin_net to { $server_net $employee_net }

# Autoriser le LAN Serveur à accéder au LAN Administration et au LAN Employé
pass in on $server_if from $server_net to { $admin_net $employee_net }

# Autoriser le LAN Employé à accéder au LAN Administration et au LAN Serveur
pass in on $employee_if from $employee_net to { $admin_net $server_net }

# Autoriser tous les sous-réseaux à sortir vers Internet
pass out on $ext_if from { $admin_net, $server_net, $employee_net } to any

# Autoriser le firewall lui-même à sortir vers Internet
pass out on $ext_if from ($ext_if) to any

# Autoriser les requêtes ping (ICMP)
pass in proto icmp from { $admin_net, $server_net, $employee_net } to any icmp-type echoreq
pass out proto icmp from { $admin_net, $server_net, $employee_net } to any icmp-type echoreq

# Autoriser les requêtes DNS (port 53 en UDP et TCP)
pass proto { udp, tcp } from any to any port 53 keep state

# Autoriser HTTP (port 80) et HTTPS (port 443)
pass proto { udp, tcp } from any to any port 80 keep state
pass proto { udp, tcp } from any to any port 443 keep state

# Autoriser les connexions déjà établies
pass in quick on $ext_if proto tcp from any to { $admin_net, $server_net, $employee_net } flags S/SA keep state
