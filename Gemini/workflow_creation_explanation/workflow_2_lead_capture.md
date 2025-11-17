# Guide Détaillé : Créer un Workflow de Capture de Leads et Notification

Ce document détaille la création du deuxième workflow : la capture de données depuis un formulaire web, leur sauvegarde dans Google Sheets, et l'envoi d'une notification instantanée à une équipe via Slack.

**Objectif final :** Lorsqu'un utilisateur soumet un formulaire sur votre site, ses informations sont automatiquement ajoutées à une feuille de calcul et un message est posté sur Slack pour que l'équipe commerciale puisse réagir rapidement.

---

### Étape 1 : Le Déclencheur - Capturer les Données du Formulaire

**Pourquoi :** Votre formulaire web (Typeform, Tally, Webflow, ou un formulaire personnalisé) est externe à n8n. Nous avons besoin d'un moyen pour que ce formulaire envoie les données à notre workflow dès qu'il est soumis. Le `Webhook` est l'outil parfait pour cela : il fournit une URL unique que des services externes peuvent appeler pour envoyer des informations.

**Comment :**

1.  **Ajouter le nœud `Webhook`**
    *   **Sous-étape :** Créez un nouveau workflow et ajoutez le nœud `Webhook` comme premier nœud.
    *   **Explication :** Ce nœud se met en attente d'une requête HTTP. Lorsqu'il en reçoit une, il démarre le workflow et rend les données de la requête disponibles pour les nœuds suivants.

2.  **Configurer le Webhook**
    *   **Sous-étape :** Dans les paramètres du nœud, laissez la `HTTP Method` sur `POST`, car les formulaires envoient généralement des données via cette méthode.
    *   **Sous-étape :** Copiez l'`URL` de `Test`. C'est cette adresse que nous allons utiliser pour simuler une soumission de formulaire et apprendre à n8n la structure des données.

3.  **Simuler une soumission pour le test**
    *   **Sous-étape :** Cliquez sur `Listen for Test Event`. Le nœud est maintenant en attente.
    *   **Sous-étape :** Utilisez un outil comme Postman, Insomnia, ou même la commande `curl` dans un terminal pour envoyer une requête `POST` à l'URL de test que vous avez copiée. Assurez-vous d'envoyer un corps de requête (body) au format JSON qui imite les données de votre formulaire.
        ```bash
        curl -X POST -H "Content-Type: application/json" -d '{"name": "Marie Curie", "email": "marie@example.com", "message": "Bonjour, je suis intéressée par vos services."}' [VOTRE_URL_DE_TEST_WEBHOOK]
        ```
    *   **Explication :** Cette étape est **cruciale**. En recevant ces données de test, n8n "apprend" la structure des données (`name`, `email`, `message`). Cela rendra l'utilisation des expressions pour mapper ces données dans les étapes suivantes beaucoup plus facile, car l'éditeur pourra vous les suggérer. Une fois la requête envoyée, le nœud `Webhook` affichera les données reçues, généralement imbriquées dans un objet `body`.

---

### Étape 2 : Le Stockage - Sauvegarder le Lead

**Pourquoi :** Les notifications sont utiles, mais vous avez besoin d'une base de données centralisée de tous vos leads pour le suivi à long terme. Un Google Sheet est une solution simple et efficace pour cela.

**Comment :**

1.  **Préparer votre Google Sheet**
    *   **Sous-étape :** Créez une nouvelle feuille de calcul dans Google Sheets. Nommez les en-têtes de colonnes pour qu'ils correspondent à vos données, par exemple : `Nom`, `Email`, `Message`, `Date de Soumission`.

2.  **Ajouter le nœud `Google Sheets`**
    *   **Sous-étape :** Ajoutez un nœud `Google Sheets` après le nœud `Webhook`.
    *   **Sous-étape :** Connectez vos identifiants (Credentials) Google. Comme pour Twitter, c'est une autorisation unique et sécurisée.
    *   **Sous-étape :** Configurez le nœud :
        *   `Resource`: `Row`
        *   `Operation`: `Append or Update` (pour ajouter une nouvelle ligne)

3.  **Mapper les données aux colonnes**
    *   **Sous-étape :** Sélectionnez votre `Spreadsheet` et votre `Sheet` dans les listes déroulantes.
    *   **Sous-étape :** Le nœud affichera les en-têtes de colonnes que vous avez créés. C'est ici que la magie opère. Pour chaque colonne, nous allons utiliser une expression pour y insérer la donnée correspondante du webhook.
        *   Pour la colonne `Nom` : `{{ $json.body.name }}`
        *   Pour la colonne `Email` : `{{ $json.body.email }}`
        *   Pour la colonne `Message` : `{{ $json.body.message }}`
        *   Pour la colonne `Date de Soumission` (bonus) : `{{ $now }}` (la variable `$now` de n8n insère la date et l'heure actuelles).
    *   **Explication :** Nous disons à n8n : "Pour chaque exécution, prends la valeur du champ `name` qui se trouve dans le `body` des données du webhook, et insère-la dans la colonne `Nom` de mon Google Sheet".

---

### Étape 3 : La Notification - Alerter l'Équipe

**Pourquoi :** La vitesse est essentielle dans la gestion des leads. Une notification instantanée sur un canal de communication partagé comme Slack permet à n'importe quel membre disponible de l'équipe de prendre en charge le nouveau lead immédiatement.

**Comment :**

1.  **Ajouter le nœud `Slack`**
    *   **Sous-étape :** Ajoutez un nœud `Slack` après le nœud `Google Sheets`.
    *   **Sous-étape :** Connectez vos identifiants Slack.

2.  **Configurer le message**
    *   **Sous-étape :**
        *   `Resource`: `Message`
        *   `Operation`: `Post`
    *   **Sous-étape :** Choisissez le `Channel` où le message doit être posté (ex: `#leads`).
    *   **Sous-étape :** Dans le champ `Text`, rédigez un message clair et informatif en utilisant des expressions pour le personnaliser. Le formatage Markdown de Slack fonctionne ici.
        ```
        :tada: *Nouveau Lead Entrant !* :tada:

        *Nom :* {{ $json.body.name }}
        *Email :* {{ $json.body.email }}
        *Message :*
        > {{ $json.body.message }}
        ```
    *   **Explication :** Ce message est bien plus utile qu'un simple "Nouveau lead". Il donne à l'équipe toutes les informations nécessaires pour évaluer le lead sans avoir à ouvrir le Google Sheet.

---

### Étape 4 : Activation et Déploiement

**Pourquoi :** Le workflow est testé et fonctionnel. Il est temps de le brancher au vrai formulaire.

**Comment :**

1.  **Utiliser l'URL de Production**
    *   **Sous-étape :** Retournez au nœud `Webhook` et copiez cette fois l'URL de `Production`.
    *   **Sous-étape :** Allez dans les paramètres de votre formulaire web (dans l'interface de Typeform, Tally, etc.) et collez cette URL dans la section "Webhook" ou "Intégrations".

2.  **Activer le Workflow**
    *   **Sous-étape :** Activez votre workflow en utilisant le bouton en haut à droite.
    *   **Explication :** Votre système est maintenant en place. Chaque fois qu'un utilisateur soumettra le formulaire, le service du formulaire appellera l'URL de production de votre webhook, ce qui déclenchera le workflow. Le lead sera sauvegardé et l'équipe notifiée, le tout en quelques secondes et sans aucune intervention manuelle.
