**Coolify Ansible - Semaphore ready**  
Version adaptée pour exécuter le bootstrap Coolify depuis Semaphore UI.  
**Ce qui a été retiré de l'ancien projet**  
- inventory SmartMDM et anciennes IP ;  
- group_vars/platform.yml SmartMDM ;  
- domaine xxx;  
- certificat wildcard/TLS partagé avec Harbor ;  
- scripts et chemins /etc/xxx/... ;  
- dépendance au groupe Ansible platform.  
Le playbook cible désormais hosts: all, donc l'Inventory choisi dans le Task Template Semaphore détermine entièrement le serveur cible.  
**Fichiers**  
- playbooks/preflight.yml : teste SSH/Ansible, sudo et ressources minimales ;  
- playbooks/coolify.yml : installe Docker puis Coolify ;  
- roles/docker : installation Docker ;  
- roles/coolify : bootstrap Coolify ;  
- semaphore/variable-group.example.json : exemple de variables non secrètes ;  
- semaphore/inventory.example.ini : exemple d'Inventory statique.  
**Configuration Semaphore**  
**1. Inventory COOLIFY**  
Exemple :  
[coolify]  
 10.X.x.X  
   
Associer le credential SSH déjà créé. Si le compte SSH demande un mot de passe pour sudo, associer aussi un Sudo Credential. Si sudo fonctionne sans mot de passe, le champ peut rester vide.  
**2. Variable Group **COOLIFY  
Copier le JSON de semaphore/variable-group.example.json puis remplacer au minimum :  
- coolify_root_username  
- coolify_root_email  
Dans l'onglet **Secrets** du même Variable Group, ajouter :  
- nom : coolify_root_password  
- valeur : mot de passe admin Coolify (12 caractères minimum dans ce bootstrap, sans espaces)  
Ne pas stocker ce mot de passe dans Git.  
**3. Repository**  
Utiliser ce projet comme racine d'un repository Git dédié. Cela permet à ansible.cfg de trouver directement ./roles dans Semaphore.  
**4. Task Template de test**  
- App : Ansible  
- Inventory : COOLIFY  
- Variable Group : COOLIFY  
- Playbook : playbooks/preflight.yml  
Lancer ce template avant l'installation.  
**5. Task Template d'installation**  
- App : Ansible  
- Inventory : COOLIFY  
- Variable Group : COOLIFY  
- Playbook : playbooks/coolify.yml  
Après succès, le dashboard est normalement accessible sur :  
http://IP_DU_SERVEUR:8000  
   
**TLS / domaine**  
Le TLS SmartMDM personnalisé a volontairement été retiré. Pour un premier bootstrap, Coolify utilise son fonctionnement standard. Configure ensuite le domaine du dashboard dans Coolify et laisse son proxy intégré gérer HTTPS/Let's Encrypt.  
