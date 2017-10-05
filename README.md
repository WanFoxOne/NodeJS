# NodeJS

## Meilleures pratiques de sécurité pour Express en production

Le terme “production” fait référence à la phase du cycle de vie du logiciel au cours de laquelle une application ou une API est généralement disponible pour ses consommateurs ou utilisateurs finaux. En revanche, en phase de “développement”, l’écriture et le test de code se poursuite activement et l’application n’est pas ouverte pour un accès externe. Les environnements système correspondants sont respectivement appelés environnement de production et environnement de développement.

Les environnement de développement et de production sont généralement configurés différemment et leurs exigences divergent grandement. Ce qui convient parfaitement en développement peut être inacceptable en production. Par exemple, dans un environnement de développement, vous pouvez souhaiter une consignation prolixe des erreurs en vue du débogage, alors que ce type de comportement présente des risques au niveau de la sécurité en environnement de production. De plus, en environnement de développement, vous n’avez pas à vous soucier de l’extensibilité, de la fiabilité et des performances, tandis que ces éléments sont essentiels en environnement de production.

### N’utilisez pas de versions obsolètes ou vulnérables d’Express

Les versions 2.x et 3.x d’Express ne sont plus prises en charge. Les problèmes liés à la sécurité et aux performances dans ces versions ne seront pas corrigés. Ne les utilisez pas ! Si vous êtes passé à la version 4, suivez le [guide de migration](http://expressjs.com/fr/guide/migrating-4.html)

Vérifiez également que vous n’utilisez aucune des versions vulnérables d’Express répertoriées sur la page [Mises à jour de sécurité](http://expressjs.com/fr/advanced/security-updates.html). Si tel est le cas, procédez à une mise à jour vers une version stable, de préférence la plus récente.

### Utilisez TLS

Si votre application traite ou transmet des données sensibles, utilisez [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) (Transport Layer Security) afin de sécuriser la connexion et les données. Cette technologie de l’information chiffre les données avant de les envoyer du client au serveur, ce qui vous préserve des risques d’hameçonnage les plus communs (et faciles). Même si les requêtes Ajax et POST ne sont pas clairement visibles et semblent “masquées” dans les navigateurs, leur trafic réseau n’est pas à l’abri d’une [détection de paquet](https://en.wikipedia.org/wiki/Packet_analyzer) ni des [attaques d’intercepteur](https://en.wikipedia.org/wiki/Man-in-the-middle_attack).

Par ailleurs, un outil très pratique, [Let’s Encrypt](https://letsencrypt.org/about/), autorité de certification gratuite, automatisée et ouverte fournie par le groupe de recherche sur la sécurité sur Internet, [ISRG (Internet Security Research Group)](https://letsencrypt.org/isrg/), vous permet de vous procurer gratuitement un certificat TLS.

### Utilisez Helmet

[Helmet](https://www.npmjs.com/package/helmet) vous aide à protéger votre application de certaines des vulnérabilités bien connues du Web en configurant de manière appropriée des en-têtes HTTP.

Helmet n’est actuellement qu’une collection de neuf fonctions middleware plus petites qui définissent des en-têtes HTTP liés à la sécurité :

* ___csp___ définit l’en-tête Content-Security-Policy pour la protection contre les attaques de type cross-site scripting et autres injections intersites.
* ___hidePoweredBy___ supprime l’en-tête X-Powered-By.
* ___hpkp___ ajoute des en-têtes Public Key Pinning (épinglage de clé publique) pour la protection contre les attaques d’intercepteur avec de faux certificats.
* ___hsts___ définit l’en-tête Strict-Transport-Security qui imposer des connexions (HTTP sur SSL/TLS) sécurisées au serveur.
* ___ieNoOpen___ définit X-Download-Options pour IE8+.
* ___noCache___ définit des en-têtes Cache-Control et Pragma pour désactiver la mise en cache côté client.
* ___noSniff___ définit X-Content-Type-Options pour protéger les navigateurs du reniflage du code MIME d’une réponse à partir du type de contenu déclaré.
* ___frameguard___ définit l’en-tête X-Frame-Options pour fournir une protection clickjacking.
* ___xssFilter___ définit X-XSS-Protection afin d’activer le filtre de script intersites (XSS) dans les navigateurs Web les plus récents.

Installez Helmet comme n’importe quel autre module :
```Bash
npm install --save helmet
```
Puis, pour l’utiliser dans votre code :
```Javascript
var helmet = require('helmet');
app.use(helmet());
```

__Désactivez au minimum l’en-tête X-Powered-By__
Si vous ne voulez pas utiliser Helmet, désactivez au minimum l’en-tête X-Powered-By. Les intrus peuvent utiliser cet en-tête (activé par défaut) afin de détecter les applications qui exécutent Express et lancer ensuite des attaques spécifiquement ciblées.
Il est donc conseillé de neutraliser l’en-tête à l’aide de la méthode __app.disable()__ comme suit :
```Javascript
app.disable('x-powered-by');
```
> Si vous utilisez helmet.js, cette opération s’effectue automatiquement.

### Utilisez les cookies de manière sécurisée

Pour garantir que les cookies n’ouvrent pas votre application aux attaques, n’utilisez pas le nom du cookie de session par défaut et définissez de manière appropriée des options de sécurité des cookies.

Il existe deux modules principaux de session de cookie de middleware :

* ___express-session___ qui remplace le middleware express.session intégré à Express 3.x.
* ___cookie-session___ qui remplace le middleware express.cookieSession intégré à Express 3.x.

La principale différence entre ces deux modules tient à la manière dont ils sauvegardent les données de session des cookies. Le middleware [express-session](https://www.npmjs.com/package/express-session) stocke les données de session sur le serveur ; il ne sauvegarde que l’ID session dans le cookie lui-même, mais pas les données de session. Par défaut, il utilise le stockage en mémoire et n’est pas conçu pour un environnement de production. En production, vous devrez configurer un magasin de sessions évolutif (voir la liste des [magasins de sessions compatibles](https://github.com/expressjs/session#compatible-session-stores)).

En revanche, le middleware [cookie-session](https://www.npmjs.com/package/cookie-session) implémente un stockage sur cookie, c’est-à-dire qu’il sérialise l’intégralité de la session sur le cookie, et non simplement une clé de session. Utilisez-le uniquement lorsque les données de session sont relativement peu nombreuses et faciles à coder sous forme de valeurs primitives (au lieu d’objets). Même si les navigateurs sont censés prendre en charge au moins 4096 octets par cookie, pour ne pas risquer de dépasser cette limite, limitez-vous à 4093 octets par domaine. De plus, n’oubliez pas que les données de cookie seront visibles du client et que s’il n’est pas nécessaire qu’elles soient sécurisées ou illisibles, express-session est probablement la meilleure solution.

__N’utilisez pas de nom de cookie de session par défaut__

L’utilisation d’un nom de cookie de session par défaut risque d’ouvrir votre application aux attaques. Le problème de sécurité qui en découle est similaire à _X-Powered-By_ : une personne potentiellement malveillante peut l’utiliser pour s’identifier auprès du serveur et cibler ses attaques en conséquence.

Pour éviter ce problème, utilisez des noms de cookie génériques, par exemple à l’aide du middleware _express-session_ :

```Javascript
var session = require('express-session');
app.set('trust proxy', 1) app.use( session({
   secret : 'yolo',
   name : 'sessionId',
  })
);
```

__Définissez des options de sécurité de cookie__

Définissez les options de cookie suivantes pour accroître la sécurité :

- __secure__ - Garantit que le navigateur n’envoie le cookie que sur HTTPS.
- __httpOnly__ - Garantit que le cookie n’est envoyé que sur HTTP(S), pas au JavaScript du client, ce qui renforce la protection contre les attaques de type cross-site scripting.
- __domain__ - Indique le domaine du cookie ; utilisez cette option pour une comparaison avec le domaine du serveur dans lequel l’URL est demandée. S’ils correspondent, vérifiez ensuite l’attribut de chemin.
- __path__ - Indique le chemin du cookie ; utilisez cette option pour une comparaison avec le chemin demandé. Si le chemin et le domaine correspondent, envoyez le cookie dans la demande.
- __expires__ - Utilisez cette option pour définir la date d’expiration des cookies persistants.

Exemple d’utilisation du middleware cookie-session :
```Javascript
var session = require('cookie-session');
var express = require('express');
var app = express();

var expireDate = new Date( Date.now() + 60 * 60 * 1000 ); app.use(session({
  name: 'session',
  keys: ['key1', 'key2'],
  cookie: { secure: true,
            httpOnly: true,
            domain: 'example.com',
            path: 'foo/bar',
            expires: expireDate
          }
  })
);
```

### Assurez-vous que vos dépendances sont sécurisées

npm est un outil puissant et pratique de gestion des dépendances de votre application. Toutefois, les packages que vous utilisez sont susceptibles de contenir des vulnérabilités critiques en matière de sécurité qui risquent d’affecter également votre application. La sécurité de votre application est aussi forte que le “lien de plus faible” de vos dépendances.

Utilisez l’un des outils suivants, ou les deux, pour vous aider à garantir la sécurité des packages tiers que vous utilisez : [nsp](https://www.npmjs.com/package/nsp) et [requireSafe](https://requiresafe.com/). Ces deux outils effectuent globalement les mêmes opérations.

nsp est un outil de ligne de commande qui vérifie dans la base de données des vulnérabilités [Node Security Project](https://nodesecurity.io/) si votre application utilise des packages qui présentent des vulnérabilités connues. Installez-le comme suit :
```Bash
npm i nsp -g
```
Utilisez cette commande afin de soumettre le fichier npm-shrinkwrap.json pour validation à nodesecurity.io :
```Bash
nsp audit-shrinkwrap
```
Utilisez cette commande afin de soumettre le fichier package.json pour validation à nodesecurity.io :
```Bash
nsp audit-package
```
Pour auditer vos modules de noeud, utilisez requireSafe comme suit :
```Bash
npm install -g requiresafe
cd your-app
requiresafe check
```
