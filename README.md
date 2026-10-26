# TP2 — Sécurisation de la gestion de pharmacie (Spring Boot + Spring Security + MySQL)

## Prérequis
- JDK 17+
- Maven 3.9+ (ou Maven intégré à l'IDE)
- MySQL 8 démarré sur `localhost:3306`

## Configuration
Adapter `spring.datasource.username` / `spring.datasource.password` dans `src/main/resources/application.properties`.
La base `pharma_security_db` est créée automatiquement. Au premier démarrage, `InitData` crée les 4 rôles, les comptes de test (mots de passe hachés en BCrypt) et des médicaments de démonstration.

## Lancement
```
mvn spring-boot:run
```
Application : http://localhost:8081

## Comptes de test
| Utilisateur | Mot de passe | Rôle | Accès |
|---|---|---|---|
| admin | admin123 | ADMIN | Tout : utilisateurs, médicaments, stocks, ventes |
| pharma | pharma123 | PHARMACIEN | Consulter les médicaments, modifier les stocks, vendre |
| caisse | caisse123 | CAISSIER | Ventes uniquement |
| client | client123 | CLIENT | Consultation des médicaments (lecture seule) |

## Matrice des autorisations
| Ressource | ADMIN | PHARMACIEN | CAISSIER | CLIENT |
|---|:-:|:-:|:-:|:-:|
| `GET /medicaments` | ✔ | ✔ | ✘ | ✔ |
| `POST /medicaments/{id}/stock` | ✔ | ✔ | ✘ | ✘ |
| `/medicaments/new`, `/{id}/edit`, `/save`, `/{id}/delete` | ✔ | ✘ | ✘ | ✘ |
| `/ventes` | ✔ | ✔ | ✔ | ✘ |
| `/admin/**` | ✔ | ✘ | ✘ | ✘ |

Contrôle à deux niveaux : règles d'URL dans `SecurityConfig` + `@PreAuthorize` sur les contrôleurs. Toute tentative non autorisée redirige vers `/access-denied` (403). Les menus et boutons sont masqués selon le rôle avec `sec:authorize`.

## Scénarios de test
1. `caisse` → accès à `/ventes` ; saisir `/medicaments` ou `/admin` dans l'URL → page « Accès refusé ».
2. `pharma` → liste des médicaments avec mise à jour du stock, sans boutons Modifier/Supprimer ; `/medicaments/new` → accès refusé ; ventes autorisées.
3. `client` → liste des médicaments en lecture seule ; `/ventes` → accès refusé.
4. `admin` → création d'un utilisateur, changement de rôle, désactivation (le compte désactivé ne peut plus se connecter), CRUD médicaments, ventes.
5. Vente avec une quantité supérieure au stock → message « Stock insuffisant », stock inchangé.

## Structure
```
config/      SecurityConfig
model/       User, Role, Medicament, Vente
repository/  UserRepository, RoleRepository, MedicamentRepository, VenteRepository
controller/  LoginController, MedicamentController, AdminController, VenteController
service/     CustomUserDetailsService, InitData, UserService, MedicamentService, VenteService
templates/   login, home, admin, medicaments, medicament-form, ventes, access-denied
```
