---
title: Infrastructure immuable
id: immutable-infrastructure
full_link:
short_description: >
  L’infrastructure immuable désigne l’infrastructure informatique (machines virtuelles, conteneurs, appliances réseau) qui ne peut pas être modifiée une fois déployée.

aka: 
tags:
- architecture
---

L’infrastructure immuable désigne l’infrastructure informatique (machines virtuelles, conteneurs, appliances réseau) qui ne peut pas être modifiée après déploiement.

<!--more-->

L’immutabilité peut être appliquée via un processus automatisé qui écrase les changements non autorisés ou via un système qui n’autorise pas de modifications.  
Les {{< glossary_tooltip text="Containers" term_id="container" >}} sont un bon exemple d’infrastructure immuable : toute modification persistante nécessite la création d’une nouvelle version du conteneur ou la recréation à partir de son image.

En empêchant ou en détectant les changements non autorisés, l’infrastructure immuable facilite la sécurité et simplifie l’exploitation, car les administrateurs peuvent raisonner sur un état stable et prévisible.  
Elle s’accompagne généralement de l’infrastructure-as-code, où toute l’automatisation est versionnée (par ex. dans Git), offrant ainsi un historique durable de chaque changement autorisé.
