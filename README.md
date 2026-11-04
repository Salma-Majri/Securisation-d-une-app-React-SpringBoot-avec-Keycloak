# Atelier Pratique : OAuth 2.0 et OpenID Connect

**Module :** Sécurité des systèmes distribués  
**Thème :** Sécurisation d'une application React + Spring Boot avec Keycloak

## Description

Ce projet illustre la mise en place d'une architecture de sécurité distribuée moderne. Il met en œuvre les protocoles **OAuth 2.0** et **OpenID Connect (OIDC)** pour authentifier une application frontend (React) et autoriser l'accès à une API backend (Spring Boot) via un serveur d'identité centralisé (**Keycloak**).

L'application développée est un système de gestion de demandes (ex : demandes de congé) avec deux niveaux d'accès :

- **USER** : peut créer et consulter ses propres demandes.
- **ADMIN** : peut consulter toutes les demandes, les valider ou les refuser.

##  Architecture Technique

| Composant                    | Technologie   | Port local |
| :--------------------------- | :------------ | :--------- |
| **Serveur d'identité**       | Keycloak      | `8080`     |
| **API (Resource Server)**    | Spring Boot   | `8081`     |
| **Frontend (Client public)** | React / Vite  | `5173`     |

##  Prérequis

- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Java 17 (ou 21)](https://adoptium.net/)
- [Node.js](https://nodejs.org/) & npm
- Un IDE (IntelliJ IDEA, VS Code) et Postman pour les tests

---

##  Installation et Démarrage

### 1. Lancer Keycloak (Docker)

Créez un dossier `atelier-oauth2`. Dans ce dossier, créez un fichier `docker-compose.yml` :

```yaml
services:
  keycloak:
    image: quay.io/keycloak/keycloak:26.8.1
    container_name: keycloak-atelier
    command: start-dev
    environment:
      KC_BOOTSTRAP_ADMIN_USERNAME: admin
      KC_BOOTSTRAP_ADMIN_PASSWORD: admin
    ports:
      - "127.0.0.1:8080:8080"
    volumes:
      - keycloak_data:/opt/keycloak/data

volumes:
  keycloak_data:
```

Démarrez le conteneur :

```bash
docker compose up -d
```

Accédez à l'interface d'administration : <http://localhost:8080> (identifiants : `admin` / `admin`).

### 2. Lancer le Backend (Spring Boot)

Ouvrez un terminal dans le dossier du projet Spring Boot (`demandes-api`) et lancez l'application :

```bash
# Sur Linux/Mac
./mvnw spring-boot:run

# Sur Windows
.\mvnw.cmd spring-boot:run
```

L'API sera accessible sur : <http://localhost:8081>.

### 3. Lancer le Frontend (React)

Ouvrez un terminal dans le dossier du projet React (`demandes-front`) et lancez l'application :

```bash
npm install
npm run dev
```

L'application sera accessible sur : <http://localhost:5173>.

---

##  Configuration de Keycloak

1. **Créer un Realm** : `enset-realm`

2. **Créer un Client** :
   - Client ID : `react-client`
   - Client authentication : **Off** (client public)
   - Standard flow : **On**
   - Valid redirect URIs : `http://localhost:5173/*`
   - Web origins : `http://localhost:5173`
   - PKCE : `S256`

3. **Créer les Rôles** : `USER` et `ADMIN` (Realm roles)

4. **Créer les Utilisateurs** :
   - `alice` (mot de passe : `alice123`, rôle : `USER`)
   - `bob` (mot de passe : `bob123`, rôles : `USER`, `ADMIN`)

---

##  Endpoints de l'API

| Méthode | Endpoint                   | Rôle requis  | Description                                   |
| :------ | :------------------------- | :----------- | :-------------------------------------------- |
| `GET`   | `/api/public`              | Aucun        | Endpoint public (test)                        |
| `GET`   | `/api/profil`              | USER / ADMIN | Retourne l'ID et le username extraits du JWT  |
| `GET`   | `/api/demandes`            | USER / ADMIN | Liste les demandes de l'utilisateur connecté  |
| `POST`  | `/api/demandes`            | USER / ADMIN | Crée une nouvelle demande                     |
| `GET`   | `/api/admin/demandes`      | ADMIN        | Liste toutes les demandes                     |
| `PATCH` | `/api/admin/demandes/{id}` | ADMIN        | Modifie le statut (`VALIDEE` / `REFUSEE`)     |

### Exemple : extraction du profil (Spring Boot)

```java
@GetMapping("/api/profil")
public Map<String, Object> getProfil(@AuthenticationPrincipal Jwt jwt) {
    return Map.of(
            "id", jwt.getSubject(),
            "username", jwt.getClaimAsString("preferred_username")
    );
}
```

---

##  Fonctionnalités Clés

- **Authentification sécurisée** : flux Authorization Code avec PKCE (Proof Key for Code Exchange) via `keycloak-js`.
- **Validation des JWT** : Spring Boot agit comme Resource Server et valide la signature cryptographique, l'émetteur (`issuer-uri`) et l'expiration des tokens.
- **Contrôle d'accès basé sur les rôles (RBAC)** : mapping des rôles `realm_access` de Keycloak vers les autorités Spring Security (`ROLE_USER`, `ROLE_ADMIN`).
- **Gestion des demandes** : interface complète pour la création, la validation et le refus des demandes.

---

## Auteur

**Réalisé par :** Majri Salma  
**Encadré par :** Abdelmajid BOUSSELHAM