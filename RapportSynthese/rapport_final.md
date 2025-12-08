
# 📘 Rapport Synthèse – Exploration de la technologie n8n et de l’intelligence artificielle

## 1. Introduction

Dans le cadre du Projet 3, notre équipe a exploré **n8n**, une plateforme open-source permettant d’automatiser des processus, connecter des services et manipuler des données grâce à des workflows visuels.  
Nous avons également utilisé différentes **IA** (ChatGPT, DeepSeek, Gemini) pour accélérer notre apprentissage, résoudre des problèmes, générer du code et rédiger de la documentation.

Notre travail s’appuie sur :

- la documentation technique de n8n,  
- un workflow Webhook + Code JavaScript,  
- un workflow d’analyse automatique de courriels,  
- notre journal de bord hebdomadaire,  
- et plusieurs expérimentations guidées par l’IA.

---

## 2. Technologie explorée : n8n

### 2.1 Définition de n8n

n8n est un outil d'automatisation open-source permettant :

- de connecter des API, services web et applications ;
- d’automatiser des tâches répétitives ;
- de transformer des données via un système d’items JSON ;
- d’ajouter du code JavaScript dans un workflow pour une logique avancée.

Il offre plus de **400 intégrations**, une interface visuelle simple, et peut être hébergé localement via Docker.

### 2.2 Fonctionnement interne

n8n repose sur quatre concepts clés :

- **Triggers (déclencheurs)** : Webhook, Cron, Email, etc.  
- **Nodes d’action** : Discord, HTTP Request, Gmail, etc.  
- **Items JSON** : structure de données circulant entre les nœuds.  
- **Expressions** : récupération dynamique des valeurs d’un nœud précédent.

À cela s’ajoute la possibilité d’exécuter du **JavaScript personnalisé** dans un node "Code".

---

## 3. Principales découvertes techniques

### 3.1 Workflow Webhook + transformation JavaScript

Nous avons conçu un workflow capable de :

1. recevoir des paramètres via une requête GET,  
2. extraire les données de l’URL,  
3. transformer les données via JavaScript,  
4. renvoyer une réponse JSON structurée.

Exemple du code utilisé :

```js
const data = items[0].json;
const query = data.query || {};

return [
  {
    json: {
      message_recu: query.message || null,
      ville_recu: query.ville || null,
      toutes_les_donnees: query
    }
  }
];
```

Cette expérience nous a permis de comprendre en profondeur :

- comment circulent les données,  
- comment manipuler des items,  
- comment structurer une réponse via un node Code.

---

### 3.2 Workflow d’analyse automatique des courriels

Nous avons créé un flux plus avancé utilisant :

- **Gmail** → Récupération des 3 derniers courriels,  
- **LLM Chain + DeepSeek** → Résumé, importance, réponse suggérée,  
- **Discord Webhook** → Publication automatique du résultat.

Ce workflow montre la force de n8n : **orchestrer plusieurs services + IA dans un seul flux automatisé**.

### 3.3 Bonnes pratiques apprises

Au fil du projet, nous avons identifié des pratiques essentielles :

- Nommer clairement chaque node.  
- Tester avec *Execute Once* avant activation.  
- Prévoir un chemin d’erreur (Error Handling).  
- Diviser les workflows pour éviter la complexité excessive.  
- Utiliser du JavaScript seulement si nécessaire.  

Ces pratiques ont grandement amélioré la stabilité de nos workflows.

---

## 4. Apport de l’IA dans le processus

### 4.1 Compréhension accélérée

L’IA nous a permis de :

- résumer des guides techniques complexes,  
- comprendre la structure interne des items,  
- expliquer des erreurs dans les nodes Code,  
- apprendre plus rapidement la logique des workflows.

L’utilisation de Gemini CLI, décrite dans notre journal, a aussi permis de compléter nos recherches.

---

### 4.2 Support dans le développement

L’IA a été un outil clé pour :

- générer du JavaScript propre,  
- corriger les erreurs dans la structure JSON,  
- créer des prompts optimisés pour DeepSeek,  
- proposer des alternatives lorsque des nodes ne fonctionnaient pas comme prévu.

Elle a servi de **mentor technique**, particulièrement utile pour comprendre des concepts nouveaux.

---

### 4.3 Aide à la rédaction et à la présentation

Nous avons utilisé l’IA pour :

- rédiger et structurer la documentation,  
- produire des explications pédagogiques,  
- améliorer la clarté du texte et de la présentation,  
- préparer le rapport synthèse.

L’IA a donc amplifié notre capacité à documenter efficacement le projet.

---

## 5. Réflexion critique : limites et potentiel

### 5.1 Limites de n8n

Malgré sa puissance, n8n présente certaines limites :

- Les workflows deviennent complexes et difficiles à maintenir lorsque trop de nodes sont utilisés.  
- Certains messages d’erreur sont peu clairs.  
- Les nodes Code exigent des compétences en JavaScript.  
- L’auto-hébergement nécessite des ressources et une configuration réseau adéquate.  
- Les API externes peuvent imposer des limites (rate-limits).

---

### 5.2 Limites de l’IA

Nous avons constaté certaines limites :

- des réponses parfois incorrectes ou inventées,  
- manque de précision si le prompt est mal formulé,  
- code généré parfois non fonctionnel,  
- nécessité de valider manuellement chaque solution.

L’IA ne remplace pas l’analyse humaine, mais elle la complète.

---

### 5.3 Potentiel combiné de n8n + IA

L’intégration de l’IA dans n8n ouvre de nouveaux horizons :

- agents intelligents autonomes,  
- automatisation complète des chaînes de décision,  
- analyse en temps réel de données métiers,  
- notifications intelligentes basées sur le contexte,  
- automatisation complète de la gestion de courriels et du support technique.

n8n devient alors un **chef d’orchestre**, et l’IA un **penseur stratégique**.

---

## 6. Conclusion

Notre exploration a démontré que n8n est une technologie puissante, flexible et accessible.  
Combinée à l’intelligence artificielle, elle permet :

- d’automatiser efficacement des tâches répétitives,  
- d’intégrer des sources de données variées,  
- de produire des analyses intelligentes,  
- de documenter et prototyper rapidement.

Ce projet nous a permis d’acquérir des compétences essentielles en automatisation, API, intégration IA, JavaScript et documentation technique.

