# Documents de tests n8n – Groupe 4318

Ce dossier contient deux documents techniques produits dans le cadre du cours  
**Exploration de nouvelles technologies**, présenté à **Nicolas Payre**, le 26 novembre 2025.

Les fichiers décrivent deux workflows n8n distincts, chacun visant à explorer une fonctionnalité différente de la plateforme.

---

## 📌 1. Test n8n – Webhook + Code JavaScript

Ce document présente la création d’un workflow permettant :

- de recevoir des données via un **Webhook HTTP (GET)** ;
- d’extraire les paramètres envoyés dans l’URL (query params) ;
- de transformer les données dans un nœud **Code (JavaScript)** ;
- de renvoyer une réponse JSON propre et structurée.

Le document inclut :
- captures d’écran du workflow ;
- configuration du Webhook ;
- exemple d’appel HTTP ;
- explication du script JavaScript utilisé ;
- résultat final attendu.

---

## 📌 2. Workflow n8n – Analyse automatique de courriels (Gmail + IA + Discord)

Ce document décrit un workflow automatisé qui :

1. lit automatiquement les trois derniers courriels reçus dans Gmail ;
2. prépare un prompt structuré ;
3. utilise **DeepSeek** pour générer une analyse en JSON (résumé, importance, réponse suggérée) ;
4. envoie les résultats dans un **salon Discord** via webhook.

Le document comprend :
- explication de chaque nœud (Gmail, LLM Chain, DeepSeek, Discord) ;
- structure du JSON généré ;
- exemple de résultat final ;
- objectif pédagogique du test.

---

## 👥 Auteurs (Groupe 4318)

- **Després, Alexy**  
- **Hisier, Yvan**  
- **Kouassi, Louis-Gérard**

---

## 📅 Date
**26 novembre 2025**

---

## 📝 Objectif du dossier

Ce dossier sert de référence pour comprendre les tests n8n réalisés durant le cours.  
Il regroupe toute la documentation Word produite pour les deux workflows afin de faciliter la consultation et l’historique dans GitHub.
