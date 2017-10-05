# NodeJS

## Meilleures pratiques de sécurité pour Express en production

Le terme “production” fait référence à la phase du cycle de vie du logiciel au cours de laquelle une application ou une API est généralement disponible pour ses consommateurs ou utilisateurs finaux. En revanche, en phase de “développement”, l’écriture et le test de code se poursuite activement et l’application n’est pas ouverte pour un accès externe. Les environnements système correspondants sont respectivement appelés environnement de production et environnement de développement.

Les environnement de développement et de production sont généralement configurés différemment et leurs exigences divergent grandement. Ce qui convient parfaitement en développement peut être inacceptable en production. Par exemple, dans un environnement de développement, vous pouvez souhaiter une consignation prolixe des erreurs en vue du débogage, alors que ce type de comportement présente des risques au niveau de la sécurité en environnement de production. De plus, en environnement de développement, vous n’avez pas à vous soucier de l’extensibilité, de la fiabilité et des performances, tandis que ces éléments sont essentiels en environnement de production.

### N’utilisez pas de versions obsolètes ou vulnérables d’Express

Les versions 2.x et 3.x d’Express ne sont plus prises en charge. Les problèmes liés à la sécurité et aux performances dans ces versions ne seront pas corrigés. Ne les utilisez pas ! Si vous êtes passé à la version 4, suivez le [guide de migration](http://expressjs.com/fr/guide/migrating-4.html)

Vérifiez également que vous n’utilisez aucune des versions vulnérables d’Express répertoriées sur la page [Mises à jour de sécurité](http://expressjs.com/fr/advanced/security-updates.html). Si tel est le cas, procédez à une mise à jour vers une version stable, de préférence la plus récente.

### Utilisez TLS

Si votre application traite ou transmet des données sensibles, utilisez [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) (Transport Layer Security) afin de sécuriser la connexion et les données. Cette technologie de l’information chiffre les données avant de les envoyer du client au serveur, ce qui vous préserve des risques d’hameçonnage les plus communs (et faciles). Même si les requêtes Ajax et POST ne sont pas clairement visibles et semblent “masquées” dans les navigateurs, leur trafic réseau n’est pas à l’abri d’une [détection de paquet](https://en.wikipedia.org/wiki/Packet_analyzer) ni des [attaques d’intercepteur](https://en.wikipedia.org/wiki/Man-in-the-middle_attack).
