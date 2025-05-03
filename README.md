# Enquete Sondage QRCode Forms  Alerte  Traitement

Parfait. Voici la **version détaillée 3x plus complète** de la **solution 2** basée sur **Microsoft Forms + QR Code + Python + intégration dans l’application back-office**.

---

# ✅ Solution Complète : Déclaration mobile des événements via Microsoft Forms + QR Code + Intégration Python

---

## 🧭 1. Introduction Générale

Dans une organisation où la sécurité, les incidents, les comportements à risque et les non-conformités doivent être suivis rigoureusement, il est indispensable de permettre aux collaborateurs, même non-techniques, de **déclarer rapidement un événement** quel que soit le lieu ou l’appareil.

La solution proposée s’appuie sur **Microsoft Forms**, un outil simple d’usage, mobile-friendly, **intégré dans Microsoft 365**, combiné à un script Python permettant de **connecter ces données à l’application de gestion des événements existante**.

---

## 📌 2. Contexte opérationnel

Votre application actuelle repose sur un modèle de données structuré avec SQLAlchemy (Python) pour gérer différents types d’événements : accidents, incidents, comportements à risque, etc.
Cependant, le **mode de saisie initial** reste limité à des utilisateurs identifiés ou à des imports manuels, souvent postérieurs à l’événement.

**Objectif :** intégrer un canal de déclaration **rapide, sécurisé, universel, mobile**, tout en **respectant les formats du modèle `Event`** existant.

---

## 🔍 3. Enjeux et défis

| Enjeu                 | Description                                                                                   | Conséquence si ignoré                     | Solution                                        |
| --------------------- | --------------------------------------------------------------------------------------------- | ----------------------------------------- | ----------------------------------------------- |
| Accessibilité         | Les agents sur le terrain doivent pouvoir déclarer depuis leur mobile sans login complexe     | Sous-déclaration des événements critiques | Formulaire Microsoft Forms + QR Code            |
| Intégrité des données | Les champs doivent correspondre strictement au modèle `Event` (type, description, site, etc.) | Erreurs à l’import ou perte d'information | Standardiser les intitulés du formulaire        |
| Automatisation        | Éviter les extractions manuelles depuis Forms                                                 | Charges manuelles et erreurs humaines     | Script Python automatique ou via API            |
| Sécurité des données  | Respecter le cadre RGPD et ISO (responsabilité, limitation, finalité)                         | Risques juridiques et de confiance        | Hébergement sécurisé via O365 + accès restreint |
| Intégration technique | Aligner les formats Forms → SQLAlchemy                                                        | Données corrompues ou rejetées            | Mapping précis + traitement des exceptions      |

---

## 🎯 4. Objectifs fonctionnels et techniques

### 🔧 Fonctionnels

1. Permettre à **tout agent** (interne ou sous-traitant) de déclarer un événement rapidement
2. Rendre le formulaire **universel via QR code ou lien court**
3. Garantir une **cohérence totale des données collectées**
4. Offrir une **visibilité immédiate** dans le back-office de traitement
5. Simplifier les workflows d’analyse, traitement et clôture

### 💻 Techniques

1. Utiliser Microsoft Forms comme frontend mobile
2. Lier Forms à Excel (stocké dans OneDrive/SharePoint)
3. Développer un **script Python** qui lit les réponses et les injecte dans la base `Event`
4. Gérer les erreurs, doublons, types inconnus, ou formats de date invalides
5. Intégrer un **job planifié** (cron, Airflow, ou workflow Azure) pour automatiser le traitement

---

## 🧩 5. Architecture de la solution

```text
Utilisateur terrain
     │
     ▼
QR code ou lien Forms
     │
     ▼
Microsoft Forms (interface mobile)
     │
     ▼
Excel (automatique via Forms) [OneDrive / SharePoint]
     │
     ▼
Python Script
     │
     ▼
Base de données (Event SQLAlchemy Model)
     │
     ▼
Application Backoffice (Flask/Django/FastAPI)
     │
     ▼
Traitement, analyse, clôture
```

---

## 📝 6. Conception du Formulaire Microsoft Forms

Créer un formulaire dans [https://forms.office.com](https://forms.office.com) avec les **champs alignés au modèle** suivant :

| Champ Forms                | Exemple                | Type                               | Correspondance dans modèle `Event` |
| -------------------------- | ---------------------- | ---------------------------------- | ---------------------------------- |
| Type d’événement           | AM, PA, AS…            | Choix multiple ou liste déroulante | `type_code`                        |
| Description de l’événement | Chute d’un échafaudage | Paragraphe                         | `description`                      |
| Site                       | Centrale X             | Texte                              | `site`                             |
| Département                | Maintenance            | Texte                              | `department`                       |
| Processus concerné         | Travaux en hauteur     | Texte                              | `processus`                        |
| Date de l’événement        | 2025-04-28             | Date                               | `date_occurrence`                  |

Configurer :

* ✅ Obligatoire sur tous les champs
* ✅ Réponses anonymes ou "Utilisateurs internes uniquement"
* ✅ Génération de QR code via "Partager > QR Code"

---

## 🔄 7. Extraction automatique via Python

### 🧪 Étape 1 – Récupérer le fichier Excel

* Cliquer sur "Ouvrir dans Excel"
* Le fichier sera disponible sur OneDrive (exemple : `/Users/nom/OneDrive/Formulaires Sécurité/Export.xlsx`)

### 📜 Étape 2 – Script Python d’importation

```python
import pandas as pd
from datetime import datetime
from app import db
from app.models import Event

EXCEL_PATH = '/Users/nom/OneDrive/Formulaires Sécurité/Export.xlsx'

def map_type_code(label):
    mapping = {
        'Accident Mortel': 'AM',
        'Presqu\'accident': 'PA',
        'Accident avec arrêt': 'AA',
        'Accident sans arrêt': 'AS',
        'Comportement à risque': 'CR',
        'Situation dangereuse': 'SD',
        'Incident': 'I'
    }
    return mapping.get(label.strip(), 'I')

def import_events():
    df = pd.read_excel(EXCEL_PATH)

    for i, row in df.iterrows():
        try:
            event = Event(
                type_code=map_type_code(row['Type d’événement']),
                description=row['Description de l’événement'],
                site=row['Site'],
                department=row['Département'],
                processus=row['Processus concerné'],
                date_occurrence=pd.to_datetime(row['Date de l’événement']),
                created_at=datetime.utcnow(),
                status='déclaré',
                severity='moyenne'
            )
            db.session.add(event)
        except Exception as e:
            print(f"⚠️ Erreur ligne {i} : {e}")

    db.session.commit()
```

---

## ⏱️ 8. Automatisation

### Option 1 : Cron Job (local ou serveur Linux)

```bash
0 * * * * /usr/bin/python3 /chemin/script/import_events.py
```

### Option 2 : Azure Logic App

* Déclencheur : "Fichier modifié dans OneDrive"
* Action : "Appeler un webhook ou un endpoint Python"
* Sécurisé et managé par Azure

---

## 📊 9. Résultat attendu dans l’application

* Les événements sont **automatiquement visibles** dans le back-office
* Le responsable peut :

  * Requalifier l’événement
  * Ajouter une analyse ou des actions correctives
  * Clôturer via un bouton dans l’interface (`closed_at = now()`)

---

## 📎 10. Avantages de la solution

| Avantage                               | Détail                                                                  |
| -------------------------------------- | ----------------------------------------------------------------------- |
| 💡 Ultra simple                        | Aucun développement frontend à faire                                    |
| 📱 Mobile-native                       | Compatible tous smartphones via QR                                      |
| 📶 Fonctionne offline (temporairement) | Soumission dès retour réseau                                            |
| 🔒 Sécurisé                            | Intégré au SSO Microsoft (optionnel)                                    |
| 🔁 Automatisable                       | Zéro intervention humaine avec cron                                     |
| 🔄 Réutilisable                        | Formulaire duplicable pour d’autres cas d’usage (audits, qualité, etc.) |

---

## 🏁 11. Conclusion

Cette solution permet une **déclaration rapide, sans friction**, des événements de sécurité ou incidents sur le terrain, tout en **garantissant une intégration parfaite avec le modèle Python existant**.
Elle est **simple à mettre en œuvre, peu coûteuse**, et respecte les enjeux de sécurité, de conformité et de performance opérationnelle.

---

Souhaitez-vous maintenant que je vous fournisse :

✅ Le fichier Excel modèle avec les bons en-têtes ?
✅ Un script Python adapté à votre structure exacte ?
✅ Un guide pas à pas PDF à partager avec vos équipes internes ?

Voici **trois variantes et alternatives** à la solution principale basée sur **Microsoft Forms + QR Code + Python + intégration dans le back-office**. Chaque variante est adaptée à différents **niveaux de sécurité**, **contextes technologiques**, ou **ressources disponibles**.

---

## 🔁 Variante 1 : **Google Forms + Google Sheets + Apps Script + Python**

### 🔹 Description :

Utilise **Google Forms** pour collecter les données via mobile ou QR Code, stockées automatiquement dans **Google Sheets**. Ensuite, un **script Python** récupère les données via l’API Google Sheets et les injecte dans l’application Python.

### 🔧 Stack :

* **Frontend** : Google Forms
* **Backend source** : Google Sheets (réponses)
* **Accès** : via l’API Google Sheets avec authentification OAuth
* **Integration Python** : via `gspread`, `oauth2client`, ou API REST

### ✅ Avantages :

* 100% gratuit et accessible sans licence Microsoft
* Très rapide à déployer (moins de 30 minutes)
* Compatible mobile / QR code
* Écosystème collaboratif Google (Docs, Drive)

### ❗ Limites :

* Moins sécurisé pour les données sensibles (RGPD)
* Moins d'intégration native dans un environnement Microsoft
* API Google à configurer (OAuth 2.0)

---

## 🔁 Variante 2 : **Jotform / Typeform + Webhook + API Flask**

### 🔹 Description :

Utiliser un **form builder avancé** comme [Typeform](https://www.typeform.com) ou [Jotform](https://www.jotform.com), avec **webhook intégré** pour appeler directement votre endpoint Python dès qu’une réponse est soumise.

### 🔧 Stack :

* **Frontend** : Typeform ou Jotform
* **Webhook** : Appel HTTP vers `/api/events/import`
* **Backend Python** : route Flask ou FastAPI traitant la charge JSON

### ✅ Avantages :

* UX très soignée, fluide et moderne
* Webhook = traitement en temps réel
* Pas besoin de cron ou lecture de fichier

### ❗ Limites :

* Compte pro requis pour les webhooks (freemium limité)
* Configuration API et endpoint à sécuriser (JWT, IP filtering…)

---

## 🔁 Variante 3 : **Power Apps + Power Automate + API Python (Azure)**

### 🔹 Description :

Utilisation de **Microsoft Power Apps** pour créer un formulaire mobile personnalisé et connecté, combiné avec **Power Automate** (anciennement Flow) pour envoyer les données vers une **API Python déployée sur Azure**.

### 🔧 Stack :

* **Frontend** : Power Apps (interface no-code, personnalisable)
* **Workflow** : Power Automate (intégration visuelle, logique métier)
* **Backend** : Azure Function / Flask API ou App Service

### ✅ Avantages :

* UX plus complète que Forms (avec logique conditionnelle, filtres dynamiques…)
* Sécurité entreprise (AAD, SSO, logs)
* Intégration directe avec Azure, Teams, SharePoint

### ❗ Limites :

* Nécessite licences Power Apps (plan par utilisateur ou par application)
* Montée en compétence sur Power Platform
* Dépendance à l’écosystème Microsoft

---

## 📊 Tableau comparatif synthétique

| Variante                   | Technologie              | Niveau de sécurité | Déploiement | Automatisation   | Coût                 |
| -------------------------- | ------------------------ | ------------------ | ----------- | ---------------- | -------------------- |
| **Microsoft Forms** (base) | Forms + Excel + Python   | 🌕🌕🌕             | ⚙️ Facile   | ✅ Cron ou script | 🟢 Inclus M365       |
| **Google Forms**           | GForms + Sheets + Python | 🌕🌕               | ⚙️ Facile   | ✅ via API Google | 🟢 Gratuit           |
| **Typeform / Jotform**     | Form + Webhook + API     | 🌕🌕🌕             | ⚙️ Moyen    | ✅ Temps réel     | 🟡 Freemium          |
| **Power Apps**             | Power Platform           | 🌕🌕🌕🌕           | ⚙️ Avancé   | ✅ Temps réel     | 🔴 Payant (licences) |

---

Souhaitez-vous que je développe une **variante complète** (ex : avec Google Forms ou Power Apps) ou que je vous fournisse **un exemple de webhook Flask/API** pour Typeform ?

