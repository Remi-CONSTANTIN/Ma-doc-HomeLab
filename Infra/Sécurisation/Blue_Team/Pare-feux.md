# Disclaimer
Il ne sera pas détaillé ici la configuration de mes outils de sécurité pour des raisons évidentes de sécurité (on est jamais trop prudents)

# Mon utilisation générale
J'utilise dans mon Home-Lab le pare-feu UFW sur à peu près toutes mes machines Linux.
Chacun de mes serveurs n'a d'ouvert que le strict minimum en terme de flux entrant. Cela me permet de restreindre de nombre de "portes" ouvertes et ainsi de les barricader.  
Pour m'aider à cela, j'utilise notamment CrowdSec, qui me permet d'isoler une machine infectée même s'il se trouve dans le même réseau local que mes autres machines.  
J'ai pu en experimenter l'efficacité grâce à ma supervision Checkmk s'étant fait bannir de tout mon parc car detectée en train de faire du Slow-Brutforce (Spoiler : ce n'était que la supervision du SSH)

# Cas particuliers
Toutefois, certaines applications bypass le pare-feu local de la machine comme Docker à cause des règles de routage qu'il créé sur son hôte  

Pour éviter de changer la configuration nftables de la machine, j'ai décidé de gérer la sécurité de ce genre de machine directement avec Proxmox.  
En effet, en passant par l'hyperviseur il est impossible pour ce qui est à l'intérieur de sa "coquille" de bypasser ses règles.
