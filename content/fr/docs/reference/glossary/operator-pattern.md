---
title: Modèle Operator
id: operator-pattern
full_link: /docs/concepts/extend-kubernetes/operator/
short_description: >
  Un contrôleur spécialisé utilisé pour gérer une ressource personnalisée.

aka:
tags:
- architecture
---

Le [modèle Operator](/docs/concepts/extend-kubernetes/operator/) est une architecture qui relie un {{< glossary_tooltip term_id="controller" >}} à une ou plusieurs ressources personnalisées.

<!--more-->

Vous pouvez étendre Kubernetes en ajoutant des contrôleurs à votre cluster, en plus des contrôleurs intégrés fournis par Kubernetes.

Si une application en cours d’exécution agit comme un contrôleur et dispose d’un accès API pour effectuer des actions sur une ressource personnalisée définie dans le plan de contrôle, c’est un exemple du modèle Operator.
