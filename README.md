<div align="center">

**Projet principal** — conçu, développé, déployé et exploité de bout en bout

<img src="logo.png" alt="OptiFuel" width="420" />

**Plateforme de gestion pour le transport routier**
<br />
Prix du carburant en temps réel · Rapports de voyage et paie · Répertoire clients · États-Unis ↔ Canada

![CI](https://img.shields.io/badge/CI-passing-brightgreen?logo=githubactions&logoColor=white)
![Deploy](https://img.shields.io/badge/Deploy-passing-brightgreen?logo=githubactions&logoColor=white)
![E2E](https://img.shields.io/badge/E2E-passing-brightgreen?logo=githubactions&logoColor=white)
![Hébergement](https://img.shields.io/badge/hébergé_sur-Kubernetes_k3s-D50C2D?logo=k3s&logoColor=white)
![Surveillance](https://img.shields.io/badge/surveillance-Cloudflare_Worker-F38020?logo=cloudflareworkers&logoColor=white)
![Sauvegardes](https://img.shields.io/badge/sauvegardes-toutes_les_heures-2E7D32)
![Reprise](https://img.shields.io/badge/reprise_après_sinistre-~4_min-123F6D)
![Loi 25](https://img.shields.io/badge/Loi_25-conforme-2E7D32)

![Next.js](https://img.shields.io/badge/Next.js-16-000000?logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React-19-149ECA?logo=react&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5.9-3178C6?logo=typescript&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-4-06B6D4?logo=tailwindcss&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-4169E1?logo=postgresql&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-6.19-2D3748?logo=prisma&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-8-FF4438?logo=redis&logoColor=white)
![Python](https://img.shields.io/badge/worker-Python_3.11-3776AB?logo=python&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-k3s-FFC61C?logo=k3s&logoColor=black)
![Vitest](https://img.shields.io/badge/tests-Vitest-6E9F18?logo=vitest&logoColor=white)
![Playwright](https://img.shields.io/badge/E2E-Playwright-2EAD33)
![Sentry](https://img.shields.io/badge/erreurs-Sentry-362D59?logo=sentry&logoColor=white)
![Google Maps](https://img.shields.io/badge/cartes-Google_Maps-4285F4?logo=googlemaps&logoColor=white)
![Cloudflare R2](https://img.shields.io/badge/stockage-Cloudflare_R2-F38020?logo=cloudflare&logoColor=white)
![Resend](https://img.shields.io/badge/courriel-Resend-000000?logo=resend&logoColor=white)

🌐 **[optifuel.cloud](https://optifuel.cloud)**

</div>

> **Code source privé.** Ce dépôt présente le projet : ce qu'il fait, son architecture et ses choix d'exploitation. Démonstration ou visite du code possible sur demande.

---

## ⚡ En bref

OptiFuel est une application web **en production**, utilisée au quotidien par les chauffeurs et l'administration d'un transporteur routier opérant entre le Québec et les États-Unis. Conçue, développée, déployée et exploitée de bout en bout par une seule personne.

| | |
|---|---|
| ⛽ **Prix du carburant** | États-Unis et Canada, par réseau, mis à jour automatiquement à partir des avis de prix des fournisseurs ; carte de chaleur, favoris |
| 🧭 **Itinéraire** | Trajet à plusieurs arrêts, stations disponibles le long du parcours, géolocalisation |
| 📄 **Rapport de voyage** | Manifeste scanné (PDF ou photo) lu automatiquement, paie calculée, PDF officiel envoyé depuis l'application |
| 👥 **Répertoire clients** | Collaboratif entre chauffeurs : notes, photos, évaluations, doublons détectés |
| 🏆 **Classement** | Chauffeur du mois, podium, chauffeur de l'année |
| 🛠️ **Backoffice** | Suivi des rapports, utilisateurs et rôles, signalements, statistiques d'utilisation |
| 📱 **Mobile** | Application installable (PWA), pensée pour la cabine du camion |

---

## 🏗️ Architecture

```mermaid
flowchart LR
    U([👤 Chauffeurs · Admins])
    PF([📧 Avis de prix<br/>fournisseurs])

    subgraph PROD["☸️ Kubernetes"]
        N[Application web<br/>Next.js]
        W[Traitement asynchrone<br/>Python]
        DB[(PostgreSQL)]
        Q[(Redis)]
    end

    subgraph OUT["☁️ Hors du serveur"]
        S[(Sauvegardes<br/>chiffrées)]
        M[Surveillance<br/>externe]
        DR[Serveur de secours<br/>créé à la demande]
    end

    U --> N
    PF --> W
    N --> DB
    N -- tâches --> Q --> W --> DB
    DB --> S
    M -. vérifie .-> N
    S -. restauration .-> DR
```

- **Application web** (Next.js, TypeScript) : interface chauffeur et backoffice, API, contrôle d'accès.
- **Service de traitement** (Python) : lecture de documents (manifestes, avis de prix), génération des PDF, calcul de paie — découplé du site par une file de tâches.
- **Tout ce qui doit survivre à une panne vit ailleurs** : sauvegardes, surveillance et serveur de secours sont hors du serveur de production.

---

## 📡 Fiabilité et exploitation

| | |
|---|---|
| 🔍 **Surveillance** | Vérification externe toutes les 2 minutes (site, traitement, fraîcheur des sauvegardes et des prix), alerte en ~4 min |
| 💾 **Sauvegardes** | Toutes les heures, hors serveur, rétention sur 3 ans ; restauration testée régulièrement |
| 🔁 **Reconstruction** | Production complète reconstruite sur un serveur neuf en ~3 min 30 |
| 🚑 **Reprise après sinistre** | Bascule vers un autre fournisseur, au Québec, en ~4 min — aucun serveur de secours payé en attente |
| 🔄 **Bascule planifiée** | Migration entre serveurs **sans perte de données**, page de maintenance, retour arrière automatique en cas d'échec |
| 🧪 **Exercices** | Chaque scénario se répète sur un environnement d'exercice, sans toucher la production |
| 🚀 **Déploiement** | CI/CD GitHub Actions, environnements dev et production séparés, images versionnées pour un retour arrière ciblé |

---

## 🛡️ Sécurité et conformité

- Contrôle d'accès par rôle, vérifié côté serveur sur chaque route
- Révocation de session immédiate (suspension ou suppression d'un compte)
- Secrets jamais en clair, copie de secours chiffrée
- Conteneurs sans privilèges administrateur
- **Loi 25 (Québec)** : politique de confidentialité, droit à la suppression de compte, rétention des données documentée
- Validation stricte des fichiers importés
- Journal d'audit des actions administratives

---

## 🧪 Qualité

Trois niveaux de tests automatisés, lancés en CI à chaque modification :

- **Unitaires** — logique métier (paie, prix, règles)
- **Intégration** — routes API contre de vraies bases jetables
- **Bout en bout** — parcours utilisateur dans un vrai navigateur (inscription, client, rapport de voyage)

---

## 🧱 Stack technique

| Couche | Technologies |
|---|---|
| Front-end | Next.js 16 (App Router), React 19, TypeScript, Tailwind CSS, Radix UI |
| Back-end | API Next.js, NextAuth, Prisma |
| Traitement | Python (lecture PDF/images, génération PDF) |
| Données | PostgreSQL, Redis |
| Cartographie | Google Maps Platform |
| Infrastructure | Kubernetes (k3s), Docker, GitHub Actions |
| Cloud | Cloudflare (Workers, R2), Resend, Sentry |
| Tests | Vitest, Playwright, Testcontainers, pytest |

---

<div align="center">

**Développé par [Tranquility42](https://github.com/Tranquility42)**

</div>
