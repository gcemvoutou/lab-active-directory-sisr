questions de réflexion

**1. Pourquoi est-il impératif d'avoir au moins 2 contrôleurs de domaine en entreprise ?**

Un contrôleur de domaine (DC) unique constitue un point de défaillance unique (SPOF) : s'il tombe en panne, plus aucune authentification, résolution DNS liée au domaine ou application de GPO n'est possible, ce qui paralyse l'ensemble du système d'information. Avec au moins deux DC, la réplication Active Directory assure une redondance.

**2. Quelle est la différence entre un groupe Global et un groupe Domaine Local ? Donner un exemple d'usage de chacun.**

Un **groupe global** peut contenir des membres uniquement issus de son propre domaine, mais peut recevoir des permissions sur des ressources situées dans n'importe quel domaine de la forêt : il sert typiquement à regrouper des *utilisateurs* ayant un point commun (ex. `GG-Commercial` regroupe tous les membres du service commercial).

Un **groupe de domaine local** peut contenir des membres issus de n'importe quel domaine de la forêt (y compris d'autres groupes globaux), mais ne peut recevoir des permissions que sur des ressources de son propre domaine : il sert à *donner des accès* à une ressource précise (ex. un groupe `DL-Partage-Commun-RW` auquel on attribue les droits NTFS sur le partage `Commun`, et dans lequel on place le groupe global `GG-AllUsers`).

**3. Qu'est-ce que la transitivité des relations d'approbation ? Donner un exemple concret.**

Une relation d'approbation (*trust*) permet aux utilisateurs d'un domaine de s'authentifier et d'accéder à des ressources d'un autre domaine. Elle est dite **transitive** lorsque, si le domaine A approuve le domaine B, et que B approuve le domaine C, alors A approuve automatiquement C, sans qu'il soit nécessaire de créer une relation directe entre A et C.

Exemple concret : dans une forêt Active Directory comportant un domaine racine `technord.local` et un domaine enfant `filiale.technord.local`, la relation d'approbation parent-enfant créée automatiquement est transitive et bidirectionnelle. Si un troisième domaine `partenaire.technord.local` est ajouté comme enfant de `filiale.technord.local`, les utilisateurs de `technord.local` peuvent, grâce à la transitivité, accéder aux ressources de `partenaire.technord.local` sans qu'une relation d'approbation explicite n'ait été configurée entre les deux.

**4. Pourquoi ne doit-on pas attribuer les permissions directement aux comptes utilisateurs (méthode AGDLP) ?**

Attribuer des permissions directement à chaque compte utilisateur devient rapidement ingérable : pour 9 utilisateurs cela reste possible, mais pour une entreprise de plusieurs centaines de comptes, chaque changement de droit ou arrivée/départ d'employé nécessiterait de modifier individuellement les permissions sur chaque ressource concernée — source d'erreurs, d'oublis et de failles de sécurité (droits non révoqués lors d'un départ, par exemple).

Le modèle **AGDLP** (Accounts → Global groups → Domain Local groups → Permissions) consiste à placer les comptes (A) dans des groupes globaux (G) reflétant leur fonction métier, ces groupes globaux dans des groupes de domaine local (DL) représentant un besoin d'accès, et à attribuer les permissions (P) au niveau du groupe de domaine local. Pour modifier les droits d'un utilisateur, il suffit alors de changer son appartenance aux groupes globaux — la gestion devient centralisée, traçable et beaucoup moins sujette aux erreurs.

**5. Qu'arriverait-il si le serveur DNS du domaine tombait en panne ? Quelles conséquences sur les connexions ?**

Active Directory repose entièrement sur le DNS pour localiser les contrôleurs de domaine (via des enregistrements SRV) et pour la résolution de noms (Kerberos utilise des noms, pas des adresses IP). Si le DNS du domaine tombe en panne :

- Les nouvelles authentifications échoueraient, car les postes ne pourraient plus localiser un contrôleur de domaine ni obtenir de ticket Kerberos
- L'application des GPO échouerait également (récupération des objets GPO depuis le DC impossible)
- L'accès aux partages réseau désignés par leur nom (`\\SRV-AD01\...`) échouerait
- Les sessions déjà ouvertes pourraient continuer à fonctionner un certain temps grâce à la mise en cache des informations d'identification (*cached credentials*), mais toute nouvelle ouverture de session ou tout accès nécessitant une nouvelle vérification serait impacté

C'est pourquoi, en production, le service DNS est lui aussi répliqué sur plusieurs contrôleurs de domaine.

**6. Quelle est la différence entre un profil local et un profil itinérant ? Quand préférer l'un ou l'autre ?**

Un **profil local** est stocké uniquement sur le disque du poste sur lequel l'utilisateur s'est connecté : ses paramètres, son bureau et ses documents (s'ils y sont stockés) ne sont disponibles que sur cette machine.

Un **profil itinérant** est stocké sur un partage serveur et copié sur le poste local à chaque connexion, puis resynchronisé vers le serveur à la déconnexion : l'utilisateur retrouve donc le même environnement (bureau, paramètres d'applications, etc.) quel que soit le poste du domaine sur lequel il se connecte.

Le profil itinérant est préférable pour des utilisateurs mobiles, changeant régulièrement de poste (exemple : commerciaux utilisant différents postes selon leur présence sur site). Le profil local reste préférable pour des postes fixes attribués à un seul utilisateur, ou lorsque les profils sont volumineux et que la bande passante réseau est limitée, car la copie d'un gros profil à chaque connexion/déconnexion peut considérablement ralentir les sessions.
