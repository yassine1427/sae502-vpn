# SAÉ 5.02 : accès VPN pour les télétravailleurs

Déploiement automatisé d'un serveur WireGuard qui donne aux salariés en télétravail un accès à l'intranet et au DNS interne de l'entreprise.

Un seul fichier (`ansible/group_vars/all/salaries.yml`) contient la liste des salariés autorisés. On lance le playbook, chaque salarié de la liste reçoit son fichier de connexion et son guide, et un salarié retiré de la liste perd son accès.

## Outils

- Ubuntu Server 24.04 (VM hôte)
- Docker et Docker Compose
- Ansible (dans un venv Python sur le poste de gestion) et ansible-vault
- WireGuard, Nginx (HTTPS), dnsmasq, ufw et iptables
- Sphinx pour la documentation

## Plan d'adressage

| Réseau | Plage | Rôle |
|---|---|---|
| net_interne | 172.20.0.0/24 | réseau de l'entreprise (`internal: true`) |
| net_externe | 172.30.0.0/24 | simule Internet, là où sont les salariés |
| tunnel wg0 | 10.8.0.0/24 | adresses données aux salariés connectés |

| Conteneur | Adresses | Port |
|---|---|---|
| vpn | 172.30.0.2, 172.20.0.2, 10.8.0.1 | 51820/udp (seul port publié) |
| intranet | 172.20.0.3 | 443/tcp |
| dns | 172.20.0.4 | 53/udp et tcp |
| salarie1 | 172.30.0.11, tunnel 10.8.0.11 | client de test |
| salarie2 | 172.30.0.12, tunnel 10.8.0.12 | client de test |

## Arborescence

```
ansible/
  inventaire.ini
  site.yml          déploie tout depuis une VM vide
  rotation.yml      change les clés d'un salarié ou de tous
  tests.yml         tests de validation T1 à T10
  group_vars/all/   variables, liste des salariés, vault
  roles/            docker, securite, certificats, wireguard, services, clients
docker/             Dockerfile et config des conteneurs, docker-compose.yml
docs/               documentation Sphinx
```

## Lancer le déploiement

```bash
cd ansible
ansible-playbook -i inventaire.ini site.yml --ask-vault-pass
```

## Branches

- `main` : version stable, celle qui sert pour la démo
- `dev` : travail en cours, fusionné dans `main` quand une étape marche
