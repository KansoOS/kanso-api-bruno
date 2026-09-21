# Kansos API

Cette collection Bruno documente les endpoints de l’API Kansos pour l’authentification et la gestion MFA/TOTP.

Le projet contient des requêtes pré-configurées pour tester rapidement les routes exposées par le backend sur `localhost:3000`.

## Prérequis

- Bruno Desktop
- Un backend API démarré sur `http://localhost:3000`
- Une base de données fonctionnelle côté serveur

## Environnement

La collection contient un environnement nommé `dev` avec la variable suivante :

- `host = localhost:3000`

Les requêtes utilisent donc des URLs de type :

- `http://localhost:3000/v1/...`

## Structure de la collection

- `User/Log in.yml` : connexion standard
- `User/Sign up.yml` : inscription
- `User/TOTP/TOTP Setup.yml` : génération du secret TOTP
- `User/TOTP/TOTP Enable.yml` : activation du TOTP
- `User/TOTP/TOTP Log in.yml` : connexion avec code TOTP
- `User/TOTP/TOTP Recovery.yml` : connexion avec code de secours

## Authentification

### 1. Inscription

Endpoint : `POST /v1/auth/signup`

Formulaire attendu :

- `email`
- `password`

Exemple :

```http
POST /v1/auth/signup
Content-Type: application/x-www-form-urlencoded

email=test@test.test&password=test1234
```

### 2. Connexion standard

Endpoint : `POST /v1/auth/login`

Formulaire attendu :

- `email`
- `password`

Exemple :

```http
POST /v1/auth/login
Content-Type: application/x-www-form-urlencoded

email=test@test.test&password=test1234
```

Réponse attendue (typique) :

```json
{
  "accessToken": "...",
  "mfaToken": "..."
}
```

Les variables de session Bruno récupérées automatiquement sont :

- `accessToken`
- `mfaToken`

### 3. Activation TOTP

Endpoint : `POST /v1/auth/totp/setup`

Cette route doit être appelée avec un `Bearer token` valide.

```http
POST /v1/auth/totp/setup
Authorization: Bearer <accessToken>
```

Réponse attendue (typique) :

```json
{
  "secret": "JBSWY3DPEHPK3PXP"
}
```

Le secret est enregistré dans la variable Bruno :

- `totpSecret`

### 4. Activation du TOTP utilisateur

Endpoint : `POST /v1/auth/totp/enable`

Formulaire attendu :

- `code`

Exemple :

```http
POST /v1/auth/totp/enable
Authorization: Bearer <accessToken>
Content-Type: application/x-www-form-urlencoded

code=123456
```

Cette requête calcule automatiquement le code TOTP dynamique avec le secret récupéré précédemment et enregistre le premier code de récupération dans :

- `recoveryCode`

### 5. Connexion avec code TOTP

Endpoint : `POST /v1/auth/login/totp`

Formulaire attendu :

- `mfaToken`
- `code`

Exemple :

```http
POST /v1/auth/login/totp
Content-Type: application/x-www-form-urlencoded

mfaToken=<mfaToken>&code=123456
```

En cas de succès, la réponse contient généralement un nouvel `accessToken`.

### 6. Connexion avec code de récupération

Endpoint : `POST /v1/auth/login/recovery`

Formulaire attendu :

- `mfaToken`
- `recoveryCode`

Exemple :

```http
POST /v1/auth/login/recovery
Content-Type: application/x-www-form-urlencoded

mfaToken=<mfaToken>&recoveryCode=<recoveryCode>
```

Cette route permet de se reconnecter en utilisant un code de secours, sans passer par le TOTP.

## Flux recommandé

1. `POST /v1/auth/signup` pour créer un compte
2. `POST /v1/auth/login` pour obtenir le `accessToken` et le `mfaToken`
3. `POST /v1/auth/totp/setup` pour générer le secret TOTP
4. `POST /v1/auth/totp/enable` pour activer le MFA
5. `POST /v1/auth/login/totp` pour se connecter avec le code TOTP

## Variables Bruno utilisées

- `host` : adresse du backend
- `accessToken` : jeton d’accès JWT
- `mfaToken` : token temporaire pour l’étape MFA
- `totpSecret` : secret TOTP de l’utilisateur
- `totpCode` : code TOTP calculé dynamiquement
- `recoveryCode` : code de secours récupéré lors de l’activation

## Sécurité

- Les tokens d’authentification doivent être conservés de manière sécurisée.
- Le secret TOTP ne doit jamais être exposé publiquement.
- Les codes de récupération doivent être stockés dans un endroit sûr et ne pas être réutilisés inutilement.

## Dépannage

- Vérifier que le backend est bien lancé sur `http://localhost:3000`
- Vérifier que les variables Bruno sont correctement définies dans l’environnement `dev`
- Vérifier que les requêtes utilisent `Content-Type: application/x-www-form-urlencoded` pour les endpoints de formulaire
- Vérifier le token `Bearer` sur les routes protégées

## Remarque

Cette collection est un support de test et de documentation pour l’API. Les réponses exactes dépendront de la logique métier et de la version du backend utilisé.

---

Pour commencer rapidement, ouvrez la collection dans Bruno puis sélectionnez l’environnement `dev` avant d’exécuter les requêtes.
