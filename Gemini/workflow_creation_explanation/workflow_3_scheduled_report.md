# Guide Détaillé : Créer un Workflow de Rapport de Données Automatisé

Ce document détaille la création du troisième workflow : la génération et l'envoi automatisés d'un rapport de données (par exemple, un résumé des ventes) à une fréquence régulière.

**Objectif final :** Chaque matin à 9h, le workflow récupère les données de ventes de la veille, calcule des statistiques clés (chiffre d'affaires total, nombre de transactions), et envoie un résumé par e-mail à l'équipe de direction.

---

### Étape 1 : Le Déclencheur - Planifier l'Exécution

**Pourquoi :** Contrairement aux workflows précédents qui réagissaient à un événement, celui-ci doit s'exécuter de manière proactive à un moment précis. Le nœud `Schedule` (ou `Cron` pour des planifications plus complexes) est conçu exactement pour cela.

**Comment :**

1.  **Ajouter le nœud `Schedule`**
    *   **Sous-étape :** Créez un nouveau workflow et ajoutez le nœud `Schedule` comme déclencheur.
    *   **Explication :** Ce nœud démarrera le workflow automatiquement selon le planning que vous définissez.

2.  **Configurer le planning**
    *   **Sous-étape :** Dans les paramètres du nœud, configurez le `Mode`. Pour un rapport quotidien, choisissez `Every Day`.
    *   **Sous-étape :** Définissez l'`Hour` à `9`.
    *   **Explication :** Avec cette configuration, le workflow se lancera tous les jours, à 9h00 précises, heure du serveur n8n. Vous pouvez ajuster cela pour un rapport hebdomadaire (`Every Week`) ou mensuel.

---

### Étape 2 : La Récupération - Obtenir les Données Brutes

**Pourquoi :** Pour créer un rapport, il faut d'abord avoir les données à analyser. Cette étape consiste à se connecter à la source de données (une base de données, une API de vente comme Stripe, ou une API interne) pour récupérer les informations brutes.

**Comment (Exemple avec une API interne) :**

1.  **Ajouter le nœud `HTTP Request`**
    *   **Sous-étape :** Ajoutez un nœud `HTTP Request` après le nœud `Schedule`.
    *   **Explication :** Ce nœud est universel et permet de se connecter à n'importe quelle API REST, ce qui en fait un excellent choix pour interagir avec des services internes ou des plateformes qui n'ont pas de nœud n8n dédié.

2.  **Configurer la requête API**
    *   **Sous-étape :** Dans le champ `URL`, entrez l'adresse de l'API qui fournit les données. Par exemple : `https://api.monentreprise.com/sales/yesterday`.
    *   **Sous-étape :** Si l'API nécessite une authentification (ce qui est courant), allez dans la section `Authentication` et configurez-la. Pour une clé API, vous choisiriez `Header Auth` et ajouteriez le nom de l'en-tête (ex: `X-API-KEY`) et la valeur de votre clé (stockée de préférence dans un credential).
    *   **Explication :** Cette requête va demander à votre système toutes les transactions de la veille.

3.  **Tester et inspecter les données**
    *   **Sous-étape :** Exécutez le nœud. En sortie, vous devriez voir une liste d'items, où chaque item représente une vente individuelle avec des détails comme `amount`, `productId`, `timestamp`, etc.

---

### Étape 3 : Le Traitement - Calculer le Résumé

**Pourquoi :** Une liste de 500 transactions n'est pas un rapport. C'est une donnée brute. L'équipe de direction a besoin de chiffres clés. Nous devons donc traiter cette liste pour en extraire un résumé pertinent. Le nœud `Code` est idéal pour ce type de calculs.

**Comment :**

1.  **Ajouter le nœud `Code`**
    *   **Sous-étape :** Ajoutez un nœud `Code` après le nœud `HTTP Request`.
    *   **Explication :** Ce nœud nous permet d'écrire du code JavaScript pour effectuer des opérations complexes sur l'ensemble des données reçues, ce qui est parfait pour des agrégations.

2.  **Écrire le script de calcul**
    *   **Sous-étape :** Dans le panneau du nœud, collez un script qui va boucler sur tous les items (les ventes), calculer le total et retourner **un seul item** contenant le résumé.

    ```javascript
    // 'items' contient toutes les ventes du nœud précédent.
    const sales = items.map(item => item.json);

    let totalRevenue = 0;
    for (const sale of sales) {
      // Assurez-vous que 'amount' est un nombre
      totalRevenue += parseFloat(sale.amount) || 0;
    }

    const transactionCount = sales.length;

    // On ne retourne qu'un seul item avec les résultats des calculs.
    // Ce nouvel item sera la seule donnée envoyée au nœud suivant.
    return [{
      json: {
        totalRevenue: totalRevenue.toFixed(2), // Arrondi à 2 décimales
        transactionCount: transactionCount,
        reportDate: new Date().toLocaleDateString('fr-FR') // Ajoute la date du jour
      }
    }];
    ```
    *   **Explication :** Ce script transforme une liste de nombreuses ventes en un objet unique et propre contenant les trois chiffres clés dont nous avons besoin pour notre rapport.

---

### Étape 4 : L'Envoi - Envoyer le Rapport par E-mail

**Pourquoi :** Le rapport est prêt, il ne reste plus qu'à le livrer aux bonnes personnes.

**Comment :**

1.  **Ajouter le nœud `Email Send`** (ou `Gmail`, `Microsoft Outlook`, etc.)
    *   **Sous-étape :** Ajoutez ce nœud après le nœud `Code`.
    *   **Sous-étape :** Configurez vos identifiants de messagerie (SMTP, Gmail, etc.).

2.  **Rédiger l'e-mail du rapport**
    *   **Sous-étape :** Remplissez les champs :
        *   `To`: L'adresse e-mail du ou des destinataires (ex: `direction@monentreprise.com`).
        *   `Subject`: Utilisez des expressions pour un titre dynamique. Ex: `Rapport de Ventes du {{ $json.reportDate }}`.
        *   `HTML`: Cochez cette option pour un e-mail bien formaté.
    *   **Sous-étape :** Dans le champ `HTML`, rédigez le corps de l'e-mail en y insérant les données du résumé provenant du nœud `Code`.

    ```html
    <h1>Rapport Quotidien des Ventes</h1>
    <p>Bonjour,</p>
    <p>Voici le résumé des ventes pour la journée du <strong>{{ $json.reportDate }}</strong> :</p>
    <ul>
        <li><strong>Chiffre d'affaires total :</strong> {{ $json.totalRevenue }} €</li>
        <li><strong>Nombre de transactions :</strong> {{ $json.transactionCount }}</li>
    </ul>
    <p>Bonne journée,</p>
    <p>Votre assistant n8n.</p>
    ```
    *   **Explication :** L'e-mail est clair, professionnel et, surtout, entièrement dynamique. Les chiffres seront mis à jour à chaque exécution.

---

### Étape 5 : Activation

**Pourquoi :** Le workflow est prêt à travailler pour vous.

**Comment :**

1.  **Activer le Workflow**
    *   **Sous-étape :** Basculez le bouton en haut à droite sur `Active`.
    *   **Explication :** C'est tout. Chaque matin à 9h, le workflow se déclenchera, récupérera les données, les calculera, et enverra le rapport. Vous venez de remplacer une tâche manuelle potentiellement longue et fastidieuse par un processus entièrement automatisé et fiable.
