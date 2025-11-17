# Documentation de l'Étape 6 : Sujets Avancés

Cette documentation couvre les sujets avancés de n8n, essentiels pour construire des workflows plus complexes, résilients et personnalisés.

## 1. Gestion des Erreurs (Error Handling)

Un workflow robuste doit anticiper et gérer les erreurs. n8n offre plusieurs stratégies pour cela.

### Stratégie 1 : Réessayer en cas d'échec (Retry on Fail)

Idéale pour les erreurs temporaires (ex: une API indisponible, une micro-coupure réseau).

*   **Où le trouver :** Dans les paramètres de n'importe quel nœud, sous l'onglet "Settings".
*   **Configuration :**
    *   **`Retry on Fail` :** Activez cette option.
    *   **`Retries` :** Le nombre de nouvelles tentatives (ex: 3).
    *   **`Wait` :** Le temps d'attente en secondes entre chaque tentative. Il est possible d'activer un "backoff" exponentiel pour augmenter ce délai à chaque nouvel échec, laissant plus de temps au service externe de récupérer.

### Stratégie 2 : Continuer en cas d'échec (Continue on Fail)

Utile pour les opérations non critiques où un échec ne doit pas bloquer tout le workflow.

*   **Où le trouver :** Dans les paramètres d'un nœud, sous l'onglet "Settings".
*   **Fonctionnement :** Si le nœud échoue, le workflow ne s'arrête pas. La sortie du nœud contiendra une propriété `error` avec les détails de l'erreur, qui pourra être traitée par les nœuds suivants (par exemple, un nœud `IF` pour vérifier si une erreur existe).

### Stratégie 3 : Le Déclencheur d'Erreur (Error Trigger)

C'est la méthode la plus centralisée et la plus puissante. Elle consiste à créer un workflow dédié uniquement à la gestion des erreurs.

1.  **Créez un "Error Workflow" :** Créez un nouveau workflow.
2.  **Ajoutez le nœud `Error Trigger` :** Ce nœud sera le point de départ de ce workflow.
3.  **Liez-le aux autres workflows :** Dans les paramètres de vos workflows "principaux", allez dans "Settings" et sélectionnez votre "Error Workflow" dans le champ `Error Workflow`.
4.  **Actions :** Maintenant, si l'un de vos workflows principaux échoue, il déclenchera l'exécution du "Error Workflow". Ce dernier reçoit toutes les informations sur l'échec (nom du workflow, nœud en erreur, message d'erreur, données d'entrée). Vous pouvez alors :
    *   Envoyer une notification sur Slack ou par e-mail.
    *   Enregistrer l'erreur dans une base de données ou un Google Sheet.
    *   Tenter une logique de récupération plus complexe.

## 2. Nœud de Code (Code Node)

Lorsque les expressions et les nœuds standards ne suffisent plus, le nœud `Code` vous donne un contrôle total en vous permettant d'écrire du code JavaScript.

### Concepts Clés

*   **Accès aux données d'entrée :** Les données des nœuds précédents sont accessibles via la variable `items`, qui est un tableau d'objets. Chaque objet `item` contient la propriété `json` avec vos données.
    ```javascript
    // Boucle sur chaque item reçu en entrée
    for (const item of items) {
      // Accède à la propriété 'email' de l'item courant
      let email = item.json.email;
      console.log(email);
    }
    ```
*   **Retourner des données :** Votre code **doit** retourner un tableau d'objets respectant la structure de n8n.
    ```javascript
    const nouveauxItems = [];
    for (const item of items) {
      const prenom = item.json.fullName.split(' ')[0];
      // Crée un nouvel item avec seulement le prénom
      nouveauxItems.push({
        json: {
          firstName: prenom
        }
      });
    }
    // Ces nouveaux items seront envoyés au nœud suivant
    return nouveauxItems;
    ```
*   **Accès aux données d'un nœud spécifique :** Vous pouvez récupérer la sortie de n'importe quel nœud précédent par son nom.
    ```javascript
    // Récupère la sortie du nœud nommé "Get Users"
    const usersData = $nodes["Get Users"].json;
    ```

## 3. Webhooks

Un webhook est un moyen pour une application externe de notifier votre workflow n8n en temps réel lorsqu'un événement se produit.

### Recevoir un Webhook

1.  **Utilisez le nœud `Webhook`** comme déclencheur de votre workflow.
2.  **Configurez-le :**
    *   **`HTTP Method` :** `GET` pour simplement recevoir un appel, `POST` pour recevoir des données dans le corps de la requête.
    *   **`Path` :** Personnalisez l'URL pour la rendre plus lisible (ex: `my-service/new-user`).
3.  **Récupérez l'URL :** Le nœud vous fournit une URL de test et une URL de production. C'est cette URL que vous devrez fournir à l'application externe (ex: Stripe, GitHub, etc.).
4.  **Testez :** Cliquez sur "Listen for Test Event" et utilisez un outil comme `curl` ou Postman pour envoyer une requête à l'URL de test. Les données reçues apparaîtront dans la sortie du nœud.

### Répondre à un Webhook

Parfois, le service qui appelle le webhook attend une réponse immédiate.

1.  **Changez le mode de réponse :** Dans les paramètres du nœud `Webhook`, sous "Respond", sélectionnez **"Using 'Respond to Webhook' node"**.
2.  **Ajoutez un nœud `Respond to Webhook`** à la fin de votre logique de traitement.
3.  **Configurez la réponse :**
    *   **`Respond With` :** `JSON` ou `HTML`.
    *   **`Status Code` :** `200` pour un succès, `400` pour une erreur client, etc.
    *   **`Response Body` :** Définissez ici les données à renvoyer, en utilisant des expressions pour inclure des résultats du workflow.
    ```json
    {
      "status": "success",
      "userId": "{{ $json.createdUserId }}"
    }
    ```

## 4. Nœuds Personnalisés (Custom Nodes)

Pour les développeurs qui veulent intégrer un service non supporté par n8n ou créer une logique très spécifique et réutilisable.

### Prérequis

*   Excellente connaissance de **JavaScript/TypeScript**.
*   Environnement de développement **Node.js** et **npm/yarn**.
*   Une instance de n8n tournant en local pour le développement.

### Structure d'un Nœud Personnalisé

Un nœud est un package `npm` qui suit une structure spécifique. Le plus simple est de cloner le [n8n-nodes-starter](https://github.com/n8n-io/n8n-nodes-starter).

*   **Fichier `.node.ts` :** Le cœur de votre nœud. C'est une classe TypeScript qui définit :
    *   `description` : Les métadonnées du nœud (nom, icône, groupe).
    *   `properties` : Les champs qui apparaissent dans le panneau des paramètres du nœud (inputs, options, etc.).
    *   `execute()` : La méthode qui contient la logique à exécuter.
*   **La méthode `execute()` :**
    *   Elle est asynchrone (`async`).
    *   Elle accède aux paramètres via `this.getNodeParameter()`.
    *   Elle accède aux données d'entrée via `this.getInputData()`.
    *   Elle doit retourner les données formatées pour le nœud suivant via `this.prepareOutputData()`.

Le développement de nœuds personnalisés est un sujet vaste. La meilleure ressource est la **documentation officielle de n8n** qui guide pas à pas dans la création, le test et la publication de votre nœud.
