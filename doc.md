API REST: 

En partant du principe que la définition d’une API est connue, une API REST est une API respectant plusieurs contraintes architecturales. Nous allons étudier et expliquer ces contraintes et quel est leurs intérêts. Mais avant d’étudier ces contraintes, nous devons faire un rapide point sur qu’est-ce que manipule les API REST.


Définitions :


Définition d’une Ressource :
Une API REST manipule et met à dispositions ce qu’on appelle des ressources. Le concept de ressource est le concept clé des API RESTful.
Une ressource est un objet avec un type, des données associées, des relations à d’autres ressources et des méthodes nous permettant d’interagir avec cette ressource. Il est assez similaire à un objet de langage de POO à la seule différence qu’il n’a que peu de méthodes différentes mis à sa dispositions (limitées sur les quelques protocoles standards HTTP, GET, POST, PUT, DELETE) alors qu’un objet POO a autant de méthodes que voulu.
Des ressources peuvent être regroupées en collections, qui ne contiennent qu’un seul et même type  de ressource. Une resource existant en dehors de toute collection est appelée ressource singleton. Les Collections sont des ressources elles mêmes. 
Comme une collection est aussi une ressource, une collection peut naturellement être contenue dans une autre ressource. On peut appeler ces collections des sub-collections. 
Idem pour des sub-ressources

Définition d’un client  :

Le Client ou le logiciel est ce qui tourne sur l’ordinateur d’un utilisateur et initie la communication avec le serveur

Définition d’un serveur  :

Le serveur est ce qui offre à l’API le moyen d’accéder aux données ou aux fonctionnalités.


Les contraintes :


Donner un ID à toute ressource : 

Chaque ressource utile doit pouvoir être identifiable et accessible via une route. Cela permet de manipuler ces ressources en effectuant des opérations CRUD (Create, Read, Update, Delete) à l'aide des méthodes HTTP standards.


// Récupérer une ressource par son ID
app.get('/resources/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const resource = resources.find(r => r.id === id);

  if (resource) {
    res.json(resource);
  } else {
    res.status(404).json({ message: 'Ressource non trouvée' });
  }
});


Lier les ressources entre elles : 

Lier les ressources entre elles dans une API RESTful est une pratique courante et essentielle pour représenter les relations et les associations entre les entités. Cela permet de modéliser des scénarios plus complexes où les ressources sont interconnectées.

Il existe plusieurs façons de le faire, mais la façon conseillée dans le modèle REST est les Hyperliens (HATEOAS) : 
Le concept de HATEOAS (Hypermedia as the Engine of Application State) encourage l'utilisation de liens hypertexte pour naviguer entre les ressources liées. Chaque ressource renvoie des liens vers d'autres ressources connexes, permettant ainsi à un client d'explorer et de naviguer dans l'API de manière dynamique. Par exemple, une ressource "Utilisateur" peut renvoyer des liens vers tous les articles associés à cet utilisateur, facilitant ainsi la découverte et la navigation dans l'API.

// Récupérer tous les utilisateurs avec les liens vers leurs articles associés
app.get('/users', (req, res) => {
  const usersWithArticles = users.map(user => {
    const userArticles = articles.filter(article => article.userId === user.id);
    const userWithLinks = { ...user, articles: [] };
    
    userArticles.forEach(article => {
      userWithLinks.articles.push({
        id: article.id,
        title: article.title,
        link: `/users/${user.id}/articles/${article.id}` // Lien vers l'article
      });
    });

    return userWithLinks;
  });

Utiliser des méthodes standards :

Le concept d’utiliser des méthodes standards dans le contexte de REST se réfère à l'utilisation des méthodes HTTP standardisées pour effectuer des opérations sur les ressources d'une API. REST définit un ensemble de méthodes bien connues qui correspondent aux opérations courantes sur les données, et chaque méthode a une signification spécifique.
Donc pour interagir avec l’API, on privilégiera de genre de pratiques :

const express = require('express');
const app = express();

// Exemple de données de ressources
let resources = [
  { id: 1, name: 'Ressource 1' },
  { id: 2, name: 'Ressource 2' },
  { id: 3, name: 'Ressource 3' }
];

// Récupérer toutes les ressources
app.get('/resources', (req, res) => {
  res.json(resources);
});

// Récupérer une ressource par son ID
app.get('/resources/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const resource = resources.find(r => r.id === id);

  if (resource) {
    res.json(resource);
  } else {
    res.status(404).json({ message: 'Ressource non trouvée' });
  }
});

// Créer une nouvelle ressource
app.post('/resources', (req, res) => {
  const newResource = req.body; // Supposons que le corps de la requête contient les données de la nouvelle ressource
  newResource.id = resources.length + 1;
  resources.push(newResource);
  res.status(201).json(newResource);
});

// Mettre à jour une ressource existante
app.put('/resources/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const updatedResource = req.body; // Supposons que le corps de la requête contient les données mises à jour
  const index = resources.findIndex(r => r.id === id);

  if (index !== -1) {
    updatedResource.id = id;
    resources[index] = updatedResource;
    res.json(updatedResource);
  } else {
    res.status(404).json({ message: 'Ressource non trouvée' });
  }
});

// Supprimer une ressource
app.delete('/resources/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const index = resources.findIndex(r => r.id === id);

  if (index !== -1) {
    const deletedResource = resources.splice(index, 1)[0];
    res.json(deletedResource);
  } else {
    res.status(404).json({ message: 'Ressource non trouvée' });
  }
});

// Écouter les requêtes sur le port 3000
app.listen(3000, () => {
  console.log('Serveur démarré sur le port 3000');
});

Plutôt que d’avoir des routes qui référencent des méthodes particulières sans utiliser les protocoles.

En effet, l’intérêt est que selon le cas, chaque protocole a des fonctionnements différents (idempotence, sécurité, etc).


Avoir des représentations multiples pour les ressources :

Permettre au client d’avoir le choix sur le type des données qui sera retourné, —> meilleure accessibilité pour les clients mais aussi possibilité d’utilisation par n’importe quel navigateur web standard // l’information mise disposition par l’appli l’est pour n’importe qui / quoi 

Communiquer sans état : 

Aucune donnée de client gardées côté serveur, pas d’historique —> évite la montée en charge des serveurs en gardant un état pour CHAQUE client, pas besoin que le client soit en contact avec le même serveur 


REST : marche dans les pas de la méthodologie http.