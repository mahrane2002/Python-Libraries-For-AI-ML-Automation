# Selenium avec Python — Guide Complet d'Automatisation et de Web Scraping

> **Documentation de Référence** · `pip install selenium` · Selenium WebDriver (v4.0+)
>
> Ce document est un guide exhaustif conçu pour les développeurs Python, Data Scientists, ML Engineers et AI Engineers pour maîtriser l'automatisation de navigateur et le web scraping de sites dynamiques avec Selenium.

---

## Table des Matières

- [1. Introduction](#1-introduction)
  - [1.1 Qu'est-ce que Selenium ?](#11-quest-ce-que-selenium)
  - [1.2 Pourquoi utiliser Selenium ?](#12-pourquoi-utiliser-selenium)
  - [1.3 Cas d'utilisation courants](#13-cas-dutilisation-courants)
- [2. Architecture de Selenium](#2-architecture-de-selenium)
  - [2.1 Les Composants Clés](#21-les-composants-clés)
  - [2.2 Communication entre les composants](#22-communication-entre-les-composants)
  - [2.3 Diagramme de l'architecture](#23-diagramme-de-larchitecture)
- [3. Installation et Configuration](#3-installation-et-configuration)
  - [3.1 Installation avec pip](#31-installation-avec-pip)
  - [3.2 Selenium Manager (Gestion automatique des Drivers)](#32-selenium-manager-gestion-automatique-des-drivers)
  - [3.3 Vérification de l'installation](#33-vérification-de-linstallation)
- [4. Premier Script Selenium](#4-premier-script-selenium)
- [5. Web Drivers Supportés](#5-web-drivers-supportés)
- [6. Locators (Localisation des Éléments)](#6-locators-localisation-des-éléments)
  - [6.1 ID](#61-id)
  - [6.2 NAME](#62-name)
  - [6.3 CLASS_NAME](#63-class_name)
  - [6.4 TAG_NAME](#64-tag_name)
  - [6.5 CSS_SELECTOR](#65-css_selector)
  - [6.6 XPATH](#66-xpath)
  - [6.7 LINK_TEXT](#67-link_text)
  - [6.8 PARTIAL_LINK_TEXT](#68-partial_link_text)
  - [6.9 Tableau Comparatif des Locators](#69-tableau-comparatif-des-locators)
- [7. Interactions avec les Éléments](#7-interactions-avec-les-éléments)
- [8. Les Waits (Gestion de l'Asynchronisme)](#8-les-waits-gestion-de-lasynchronisme)
  - [8.1 Implicit Wait](#81-implicit-wait)
  - [8.2 Explicit Wait & Expected Conditions](#82-explicit-wait--expected-conditions)
  - [8.3 Bonnes Pratiques pour les Waits](#83-bonnes-pratiques-pour-les-waits)
- [9. Gestion des Formulaires](#9-gestion-des-formulaires)
- [10. Gestion des Fenêtres et Onglets](#10-gestion-des-fenêtres-et-onglets)
- [11. Gestion des Alertes](#11-gestion-des-alertes)
- [12. Gestion des Iframes](#12-gestion-des-iframes)
- [13. Gestion des Uploads et Téléchargements](#13-gestion-des-uploads-et-téléchargements)
- [14. Captures d'Écran (Screenshots)](#14-captures-décran-screenshots)
- [15. Navigation et Historique](#15-navigation-et-historique)
- [16. Gestion des Cookies](#16-gestion-des-cookies)
- [17. Exceptions Courantes et Résolution](#17-exceptions-courantes-et-résolution)
- [18. Mode Sans Tête (Headless Mode)](#18-mode-sans-tête-headless-mode)
- [19. Page Object Model (POM)](#19-page-object-model-pom)
- [20. Cas d'Utilisation Réels](#20-cas-dutilisation-réels)
- [21. Bonnes Pratiques Générales](#21-bonnes-pratiques-générales)
- [22. Limitations et Alternatives](#22-limitations-et-alternatives)
- [23. Conclusion](#23-conclusion)

---

## 1. Introduction

### 1.1 Qu'est-ce que Selenium ?
**Selenium** est un projet open-source regroupant une suite d'outils dédiés à l'automatisation des navigateurs web. Dans le contexte de Python, Selenium WebDriver fournit une API propre pour interagir par programmation avec les navigateurs web les plus populaires (Chrome, Firefox, Edge, Safari).

### 1.2 Pourquoi utiliser Selenium ?
Contrairement aux outils de scraping traditionnels comme `Requests` et `BeautifulSoup` (qui ne font que récupérer et parser le code HTML statique), Selenium exécute un véritable moteur de rendu de navigateur. Cela lui permet de :
* Interagir avec le contenu rendu dynamiquement par du **JavaScript** (React, Angular, Vue, etc.).
* Simuler des actions de l'utilisateur réel (clics, saisies clavier, glisser-déposer, défilement).
* Gérer les sessions authentifiées complexes, les iframes, et les popups de manière transparente.

### 1.3 Cas d'utilisation courants
* **Tests Fonctionnels et QA** : Vérifier que les fonctionnalités d'une application web marchent comme prévu.
* **Web Scraping Dynamique** : Extraire des données de sites web complexes ou d'applications monopages (SPA).
* **Automatisation des Tâches** : Remplir des formulaires répétitifs, télécharger des rapports hebdomadaires, ou effectuer des actions quotidiennes sur le web.
* **Génération de Données d'Entraînement pour l'IA** : Collecter des données massives sur des plateformes web dynamiques pour nourrir des modèles de Machine Learning.

---

## 2. Architecture de Selenium

L'architecture moderne de Selenium (depuis la version 4) repose sur un protocole standardisé par le W3C, ce qui garantit une stabilité et une compatibilité maximales.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       Architecture de Selenium W3C                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌────────────────────────┐              ┌───────────────────────────┐  │
│  │ Selenium Python Client │              │     Selenium Manager      │  │
│  │       (Votre Code)     ├─────────────►│(Gestion des Drivers/Brows)│  │
│  └───────────┬────────────┘              └───────────────────────────┘  │
│              │ (Requêtes HTTP locales)                                  │
│              ▼                                                          │
│  ┌────────────────────────┐                                             │
│  │   WebDriver Protocol   │                                             │
│  │     (Standard W3C)     │                                             │
│  └───────────┬────────────┘                                             │
│              │ (Format JSON)                                            │
│              ▼                                                          │
│  ┌────────────────────────┐                                             │
│  │     Browser Driver     │ (Ex: ChromeDriver, GeckoDriver)             │
│  └───────────┬────────────┘                                             │
│              │ (Commandes natives du navigateur)                        │
│              ▼                                                          │
│  ┌────────────────────────┐                                             │
│  │       Navigateur       │ (Ex: Google Chrome, Mozilla Firefox)        │
│  └────────────────────────┘                                             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.1 Les Composants Clés
1. **Selenium Client Library (Python)** : La bibliothèque Python fournit l'API de programmation qui traduit vos commandes en requêtes HTTP.
2. **WebDriver (Protocole W3C)** : Le protocole de communication standardisé utilisé pour dialoguer entre les clients Selenium et les Browser Drivers.
3. **Browser Driver** : Un exécutable tiers (comme `chromedriver` pour Chrome ou `geckodriver` pour Firefox) qui agit comme un traducteur entre le protocole standardisé et l'API interne du navigateur.
4. **Browser (Navigateur)** : L'instance réelle du navigateur web qui affiche et traite les pages.

### 2.2 Communication entre les composants
Dans Selenium 4, la communication se fait directement sans passer par un serveur intermédiaire (comme le JSON Wire Protocol obsolète de Selenium 3). Le script Python communique via des requêtes HTTP directes basées sur les standards W3C avec le Browser Driver, qui lui-même pilote directement le navigateur.

---

## 3. Installation et Configuration

### 3.1 Installation avec pip
Installez le package Selenium officiel à l'aide de pip :

```bash
pip install selenium
```

### 3.2 Selenium Manager (Gestion automatique des Drivers)
> [!NOTE]
> À partir de **Selenium 4.6.0**, vous n'avez plus besoin de télécharger manuellement les binaires de pilotes (`chromedriver`, etc.) ou d'utiliser des packages tiers comme `webdriver-manager`.
>
> **Selenium Manager** est un outil CLI interne écrit en Rust et intégré à la bibliothèque qui détecte automatiquement les navigateurs installés sur votre système et télécharge la version correspondante et compatible du Driver en arrière-plan.

### 3.3 Vérification de l'installation
Pour vérifier que Selenium est correctement installé et capable de lancer un navigateur, exécutez le script suivant :

```python
import selenium

print(f"Version de Selenium : {selenium.__version__}")
```

---

## 4. Premier Script Selenium

Voici un script minimal pour illustrer le cycle de vie de base d'un script Selenium.

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
import time

# 1. Initialiser le WebDriver (Ici Chrome)
# Selenium Manager gère le téléchargement automatique du driver
driver = webdriver.Chrome()

try:
    # 2. Accéder à une page web
    driver.get("https://www.example.com")
    
    # 3. Lire des informations de la page
    title = driver.title
    url = driver.current_url
    print(f"Titre de la page : {title}")
    print(f"URL actuelle : {url}")
    
    # Petite pause visuelle pour l'exemple
    time.sleep(3)
    
finally:
    # 4. Fermer proprement le navigateur et libérer les ressources
    driver.quit()
```

---

## 5. Web Drivers Supportés

Selenium offre un support natif pour les principaux navigateurs du marché :

| Navigateur | Driver Associé | Classe Python correspondante |
|---|---|---|
| **Google Chrome** | ChromeDriver | `webdriver.Chrome()` |
| **Mozilla Firefox** | GeckoDriver | `webdriver.Firefox()` |
| **Microsoft Edge** | EdgeDriver | `webdriver.Edge()` |
| **Apple Safari** | SafariDriver | `webdriver.Safari()` |

### Exemple de configuration multi-navigateurs :

```python
from selenium import webdriver

# Initialiser Firefox
# driver = webdriver.Firefox()

# Initialiser Edge
# driver = webdriver.Edge()

# Initialiser Safari (uniquement sur macOS, nécessite l'activation préalable dans Safari Developer Settings)
# driver = webdriver.Safari()
```

---

## 6. Locators (Localisation des Éléments)

Les locators permettent de cibler précisément un élément HTML sur une page afin d'interagir avec lui. Dans Selenium, toutes les méthodes de recherche d'éléments utilisent la classe `By`.

```python
from selenium.webdriver.common.by import By
```

### 6.1 ID
Cible un élément unique via son attribut `id`.

* **Exemple** :
  ```python
  element = driver.find_element(By.ID, "submit-btn")
  ```
* **Avantages** : L'ID est généralement unique au sein de la page HTML, ce qui en fait le moyen de localisation le plus rapide et le plus fiable.
* **Inconvénients** : Parfois absent, ou généré de façon dynamique par certains frameworks front-end (ex: `button-74y9a`).
* **Cas d'utilisation** : Idéal pour les boutons de validation, formulaires de connexion ou conteneurs principaux dotés d'IDs statiques.

### 6.2 NAME
Cible les éléments via leur attribut HTML `name`.

* **Exemple** :
  ```python
  element = driver.find_element(By.NAME, "username")
  ```
* **Avantages** : Très fréquent dans les formulaires et champs d'entrée HTML.
* **Inconvénients** : Plusieurs éléments peuvent partager le même attribut `name` (comme les boutons radio).
* **Cas d'utilisation** : Idéal pour cibler les formulaires, inputs de texte ou champs de recherche.

### 6.3 CLASS_NAME
Cible les éléments via leur classe CSS.

* **Exemple** :
  ```python
  element = driver.find_element(By.CLASS_NAME, "btn-primary")
  ```
* **Avantages** : Permet de cibler facilement un style ou une catégorie d'éléments.
* **Inconvénients** : Une classe CSS est rarement unique et est souvent partagée par de nombreux éléments sur la page.
* **Cas d'utilisation** : Parfait pour récupérer une liste d'éléments similaires (ex: des éléments de liste, des cartes de produits).

### 6.4 TAG_NAME
Cible les éléments par leur balise HTML (tag).

* **Exemple** :
  ```python
  links = driver.find_elements(By.TAG_NAME, "a")
  ```
* **Avantages** : Très utile pour collecter de larges volumes d'éléments du même type.
* **Inconvénients** : Extrêmement générique, ne permet pas un ciblage précis.
* **Cas d'utilisation** : Extraction de tous les liens (`a`), de toutes les images (`img`) ou de tous les paragraphes (`p`) d'un document.

### 6.5 CSS_SELECTOR
Cible les éléments à l'aide de sélecteurs CSS standard (similaire à la sélection de styles dans les feuilles de calcul CSS).

* **Exemple** :
  ```python
  element = driver.find_element(By.CSS_SELECTOR, "div.content > ul.list > li:first-child")
  ```
* **Avantages** : Extrêmement puissant, rapide, lisible et standardisé.
* **Inconvénients** : Ne permet pas de naviguer vers le haut du DOM (impossible de cibler un parent à partir d'un enfant) ou de cibler par texte.
* **Cas d'utilisation** : À utiliser en priorité pour la plupart des sélections complexes où l'ID n'est pas disponible.

### 6.6 XPATH
Cible les éléments à l'aide d'expressions XML Path Language (XPath). Permet de naviguer de manière bidirectionnelle dans le DOM.

* **Exemple** :
  ```python
  element = driver.find_element(By.XPATH, "//button[contains(text(), 'Valider')]/ancestor::div[1]")
  ```
* **Avantages** : Le locator le plus puissant et flexible. Permet de cibler des éléments basés sur leur contenu textuel, de naviguer vers les parents (`..`), les frères ou les ancêtres.
* **Inconvénients** : Plus lent à exécuter que le sélecteur CSS. Syntaxe complexe et risque de fragilité si le chemin absolu du DOM change.
* **Cas d'utilisation** : À réserver pour cibler des structures dynamiques complexes ou basées sur du texte textuel précis.

### 6.7 LINK_TEXT
Cible les balises d'ancre (`<a>`) par leur texte exact.

* **Exemple** :
  ```python
  element = driver.find_element(By.LINK_TEXT, "En savoir plus")
  ```
* **Avantages** : Très explicite et simple d'utilisation pour la navigation textuelle.
* **Inconvénients** : Sensible à la casse, aux espaces superflus et aux traductions de la page.
* **Cas d'utilisation** : Idéal pour cliquer sur des menus de navigation simples.

### 6.8 PARTIAL_LINK_TEXT
Cible les balises d'ancre (`<a>`) par une partie de leur texte.

* **Exemple** :
  ```python
  element = driver.find_element(By.PARTIAL_LINK_TEXT, "En savoir")
  ```
* **Avantages** : Plus souple que `LINK_TEXT` quand le texte contient des variables dynamiques (ex: "En savoir plus (15)").
* **Inconvénients** : Risque de correspondances multiples non souhaitées.
* **Cas d'utilisation** : Navigation au sein de listes de liens semi-dynamiques.

---

### 6.9 Tableau Comparatif des Locators

| Locator (`By.*`) | Vitesse | Flexibilité | Fragilité | Syntaxe simple | Recommandation |
|---|---|---|---|---|---|
| **ID** | ⚡ Très Rapide | ❌ Faible | 🟢 Faible | Oui | **Premier choix** |
| **NAME** | ⚡ Rapide | ❌ Faible | 🟡 Moyenne | Oui | **Excellent pour formulaires** |
| **CSS_SELECTOR**| ⚡ Rapide | 🟢 Haute | 🟡 Moyenne | Oui | **Deuxième choix (Prioritaire)** |
| **XPATH** | 🐢 Plus lent | 🏆 Maximale | 🔴 Élevée | Non | **En dernier recours / Recherche textuelle** |
| **CLASS_NAME** | ⚡ Rapide | ❌ Faible | 🔴 Élevée | Oui | À utiliser pour des groupes d'éléments |
| **LINK_TEXT** | 🟡 Moyenne | ❌ Faible | 🔴 Élevée | Oui | Utile pour la navigation basique |

---

## 7. Interactions avec les Éléments

Une fois l'élément localisé, Selenium permet d'exécuter des actions interactives équivalentes à celles d'un utilisateur réel.

```python
from selenium import webdriver
from selenium.webdriver.common.by import By

driver = webdriver.Chrome()
driver.get("https://www.example.com/form")

# Localisation
element = driver.find_element(By.ID, "search-input")

# 1. send_keys() - Saisir du texte
element.send_keys("Intelligence Artificielle")

# 2. clear() - Effacer le texte d'un champ
element.clear()

# 3. click() - Cliquer sur un élément
submit_btn = driver.find_element(By.ID, "submit-btn")
submit_btn.click()

# 4. get_attribute() - Récupérer la valeur d'un attribut HTML
class_value = submit_btn.get_attribute("class")
print(f"Classe CSS du bouton : {class_value}")

# 5. text - Récupérer le texte visible à l'intérieur de l'élément (propriété, pas une fonction)
paragraph = driver.find_element(By.CLASS_NAME, "description")
print(f"Texte du paragraphe : {paragraph.text}")

# 6. submit() - Soumettre un formulaire (applicable directement sur les balises de formulaire ou les inputs associés)
element.submit()

driver.quit()
```

---

## 8. Les Waits (Gestion de l'Asynchronisme)

Dans les applications web modernes, les éléments se chargent de manière asynchrone (Ajax, requêtes API). Tenter d'interagir avec un élément non encore présent dans le DOM provoquera une erreur. Les "Waits" permettent de gérer cette attente.

### 8.1 Implicit Wait
Définit un temps d'attente global pour toute la durée de vie du pilote. Si Selenium ne trouve pas un élément immédiatement, il va interroger le DOM régulièrement pendant la durée définie avant de lever une exception.

> [!WARNING]
> L'attente implicite ralentit les scripts lors des échecs de recherche d'éléments et est déconseillée sur les projets complexes. Il ne faut **jamais mélanger** les attentes implicites et explicites.

```python
driver = webdriver.Chrome()
# Attente implicite globale de 10 secondes
driver.implicitly_wait(10)

driver.get("https://www.example.com")
# Si l'élément met 3 secondes à apparaître, Selenium attendra 3 secondes.
element = driver.find_element(By.ID, "dynamic-element")
```

### 8.2 Explicit Wait & Expected Conditions
Permet de définir une attente ciblée pour un élément ou une condition spécifique. C'est la méthode de loin la plus propre et recommandée.

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()
driver.get("https://www.example.com")

try:
    # Attendre au maximum 15 secondes qu'un élément spécifique soit cliquable
    wait = WebDriverWait(driver, 15)
    element = wait.until(EC.element_to_be_clickable((By.ID, "login-button")))
    element.click()
    
    # Attendre que le texte d'un élément contienne une valeur attendue
    wait.until(EC.text_to_be_present_in_element((By.ID, "status"), "Succès"))
    
finally:
    driver.quit()
```

### Principales Expected Conditions (`EC.*`) :

| Méthode EC | Rôle |
|---|---|
| `presence_of_element_located` | L'élément est présent dans le DOM (pas forcément visible). |
| `visibility_of_element_located` | L'élément est présent dans le DOM et visible à l'écran. |
| `element_to_be_clickable` | L'élément est visible et activable (cliquable). |
| `text_to_be_present_in_element` | Un texte spécifique est détecté dans l'élément. |
| `title_contains` | Le titre de la page contient une chaîne spécifique. |

### 8.3 Bonnes Pratiques pour les Waits
1. **Évitez à tout prix `time.sleep()`** : Bloque le thread Python de façon fixe, rendant vos tests inutilement lents et fragiles en fonction de la vitesse du réseau.
2. **Utilisez majoritairement les Explicit Waits** : C'est la solution robuste et modulaire pour le web moderne.

---

## 9. Gestion des Formulaires

Selenium permet de manipuler facilement tous les types d'éléments d'un formulaire HTML.

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import Select

driver = webdriver.Chrome()
driver.get("https://www.example.com/forms")

# 1. Champs texte
username_field = driver.find_element(By.ID, "username")
username_field.send_keys("MonIdentifiant")

# 2. Boutons Radio
# Cliquer sur un bouton radio pour le sélectionner
radio_male = driver.find_element(By.XPATH, "//input[@type='radio' and @value='M']")
if not radio_male.is_selected():
    radio_male.click()

# 3. Cases à cocher (Checkboxes)
checkbox_terms = driver.find_element(By.ID, "agree-terms")
if not checkbox_terms.is_selected():
    checkbox_terms.click()

# 4. Listes déroulantes (<select>)
# Selenium fournit une classe utilitaire dédiée : Select
select_element = driver.find_element(By.ID, "country-dropdown")
dropdown = Select(select_element)

# Sélectionner par texte visible
dropdown.select_by_visible_text("France")

# Sélectionner par valeur d'attribut
dropdown.select_by_value("FR")

# Sélectionner par index numérique
dropdown.select_by_index(1)

driver.quit()
```

---

## 10. Gestion des Fenêtres et Onglets

Selenium gère chaque fenêtre et onglet à l'aide de descripteurs uniques appelés **Window Handles**.

```python
from selenium import webdriver
from selenium.webdriver.common.by import By

driver = webdriver.Chrome()
driver.get("https://www.example.com")

# Stocker l'identifiant de la fenêtre principale
original_window = driver.current_window_handle

# Vérifier qu'il n'y a qu'une seule fenêtre
assert len(driver.window_handles) == 1

# Ouvrir un nouvel onglet et y basculer automatiquement
driver.switch_to.new_window('tab')
driver.get("https://www.google.com")

# Ouvrir une nouvelle fenêtre et y basculer
driver.switch_to.new_window('window')
driver.get("https://www.github.com")

# Parcourir et basculer entre toutes les fenêtres ouvertes
for window_handle in driver.window_handles:
    if window_handle != original_window:
        driver.switch_to.window(window_handle)
        print(f"Titre de la fenêtre active : {driver.title}")
        break

# Fermer la fenêtre active actuelle
driver.close()

# Revenir à la fenêtre principale d'origine
driver.switch_to.window(original_window)

driver.quit()
```

---

## 11. Gestion des Alertes

Les alertes JavaScript natives bloquent l'interaction avec la page. Vous devez les gérer explicitement.

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()
driver.get("https://www.example.com/alerts")

# Déclencher une alerte
driver.find_element(By.ID, "alert-btn").click()

# Attendre que l'alerte soit présente
WebDriverWait(driver, 10).until(EC.alert_is_present())

# Basculer vers l'alerte active
alert = driver.switch_to.alert

# Récupérer le texte de l'alerte
print(f"Message de l'alerte : {alert.text}")

# Options d'interactions :
# 1. Accepter l'alerte (cliquer sur OK)
alert.accept()

# 2. Rejeter l'alerte (cliquer sur Annuler)
# alert.dismiss()

# 3. Envoyer du texte à une invite d'alerte (prompt)
# alert.send_keys("Mon entrée utilisateur")
# alert.accept()

driver.quit()
```

---

## 12. Gestion des Iframes

Les Iframes (`<iframe>`) intègrent un autre document HTML au sein de la page principale. Selenium ne peut pas accéder aux éléments d'une Iframe sans y basculer explicitement au préalable.

```python
from selenium import webdriver
from selenium.webdriver.common.by import By

driver = webdriver.Chrome()
driver.get("https://www.example.com/iframe-page")

# 1. Cibler l'iframe par son élément, son ID ou son index
iframe_element = driver.find_element(By.TAG_NAME, "iframe")

# Basculer le focus dans l'iframe
driver.switch_to.frame(iframe_element)

# Interagir avec les éléments internes à l'iframe
internal_btn = driver.find_element(By.ID, "inside-button")
internal_btn.click()

# 2. Revenir au document principal de la page
driver.switch_to.default_content()

driver.quit()
```

---

## 13. Gestion des Uploads et Téléchargements

### 13.1 Upload de Fichiers
Pour uploader un fichier, il ne faut pas tenter de cliquer sur le bouton d'upload (qui ouvre une boîte de dialogue OS non contrôlable par Selenium), mais envoyer le chemin absolu du fichier local directement sur l'élément input de type file.

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
import os

driver = webdriver.Chrome()
driver.get("https://www.example.com/upload")

# Récupérer le chemin d'un fichier local
file_path = os.path.abspath("mon_document.pdf")

# Cibler l'input HTML de type file
upload_input = driver.find_element(By.XPATH, "//input[@type='file']")

# Envoyer le chemin du fichier
upload_input.send_keys(file_path)

# Soumettre le formulaire
driver.find_element(By.ID, "submit-upload").click()
driver.quit()
```

### 13.2 Téléchargement de Fichiers (Configuration du Driver)
Vous pouvez configurer le répertoire par défaut pour les téléchargements de fichiers directement lors de la création du navigateur.

```python
from selenium import webdriver
import os

# Définir les options du navigateur Chrome
options = webdriver.ChromeOptions()
download_dir = os.path.abspath("./downloads")

prefs = {
    "download.default_directory": download_dir,
    "download.prompt_for_download": False, # Ne pas demander où enregistrer le fichier
    "directory_upgrade": True,
    "safebrowsing.enabled": True
}
options.add_experimental_option("prefs", prefs)

driver = webdriver.Chrome(options=options)
driver.get("https://www.example.com/download-page")

# Cliquer sur le lien de téléchargement
driver.find_element(By.ID, "download-link").click()
```

---

## 14. Captures d'Écran (Screenshots)

Les captures d'écran sont essentielles pour déboguer des scripts d'intégration continue (CI/CD) ou documenter des erreurs de scraping.

```python
from selenium import webdriver
from selenium.webdriver.common.by import By

driver = webdriver.Chrome()
driver.get("https://www.example.com")

# 1. Capture de la page complète (viewport)
driver.save_screenshot("full_page_screenshot.png")

# 2. Capture d'un élément spécifique
element = driver.find_element(By.CSS_SELECTOR, "h1")
element.screenshot("headline_element.png")

driver.quit()
```

---

## 15. Navigation et Historique

Selenium permet d'interagir facilement avec l'historique du navigateur web.

```python
from selenium import webdriver

driver = webdriver.Chrome()

# Accéder à la première page
driver.get("https://www.google.com")

# Accéder à une deuxième page
driver.get("https://www.github.com")

# 1. Reculer dans l'historique (Retour)
driver.back() # Revient sur Google.com

# 2. Avancer dans l'historique
driver.forward() # Revient sur GitHub.com

# 3. Rafraîchir la page actuelle
driver.refresh()

driver.quit()
```

---

## 16. Gestion des Cookies

La manipulation des cookies est particulièrement utile en scraping pour persister des sessions ou contourner des barrières d'authentification.

```python
from selenium import webdriver

driver = webdriver.Chrome()
driver.get("https://www.example.com")

# 1. Ajouter un cookie
cookie = {"name": "user_session", "value": "AI_Engineer_Token123"}
driver.add_cookie(cookie)

# Rafraîchir pour appliquer le cookie à la session active
driver.refresh()

# 2. Récupérer un cookie spécifique par son nom
session_cookie = driver.get_cookie("user_session")
print(f"Détail du cookie de session : {session_cookie}")

# 3. Récupérer tous les cookies du domaine actuel
all_cookies = driver.get_cookies()
print(f"Nombre total de cookies : {len(all_cookies)}")

# 4. Supprimer un cookie par son nom
driver.delete_cookie("user_session")

# 5. Supprimer tous les cookies de la session actuelle
driver.delete_all_cookies()

driver.quit()
```

---

## 17. Exceptions Courantes et Résolution

Lors du développement avec Selenium, vous rencontrerez inévitablement des exceptions. Voici les plus courantes et comment y remédier :

| Exception | Cause Fréquente | Solution |
|---|---|---|
| **`NoSuchElementException`** | L'élément ciblé par votre locator n'existe pas ou n'est pas encore présent dans le DOM. | Utiliser un **Explicit Wait** (`presence_of_element_located`). Vérifier l'exactitude du sélecteur. |
| **`TimeoutException`** | Un Explicit Wait a expiré sans que sa condition attendue ne soit satisfaite. | Augmenter la valeur du timeout ou vérifier si le site a changé sa structure HTML. |
| **`StaleElementReferenceException`** | L'élément avec lequel vous interagissez a été rafraîchi ou re-rendu dans le DOM par JavaScript. | Relocaliser à nouveau l'élément avant d'interagir ou utiliser des waits pour le rechargement de la page. |
| **`ElementClickInterceptedException`** | Une popup, un spinner de chargement ou un overlay invisible intercepte le clic à la place de l'élément cible. | Attendre que l'overlay disparaisse (`invisibility_of_element`) ou forcer le clic via JavaScript. |
| **`ElementNotInteractableException`** | L'élément est présent dans le DOM mais est masqué, invisible ou désactivé pour le moment. | Attendre que l'élément soit visible (`visibility_of_element_located`) ou cliquable. |

### Exemple de gestion propre des exceptions :

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.common.exceptions import NoSuchElementException, TimeoutException
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()
driver.get("https://www.example.com")

try:
    wait = WebDriverWait(driver, 5)
    element = wait.until(EC.presence_of_element_located((By.ID, "login-btn")))
    element.click()
except TimeoutException:
    print("Erreur : L'attente du bouton a expiré.")
except NoSuchElementException:
    print("Erreur : Le bouton de connexion n'a pas été trouvé dans le DOM.")
finally:
    driver.quit()
```

---

## 18. Mode Sans Tête (Headless Mode)

Le mode **headless** (sans tête) lance le navigateur en arrière-plan sans interface utilisateur graphique (GUI). Il est indispensable pour exécuter des scripts sur des serveurs, des conteneurs Docker ou des environnements de CI/CD (GitHub Actions).

```python
from selenium import webdriver

# Configurer les options du navigateur
options = webdriver.ChromeOptions()

# Activer le mode headless (Syntaxe moderne recommandée)
options.add_argument("--headless=new")

# Recommandations pour éviter les bugs en mode headless sur serveurs
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")
# Définir une taille d'écran standard pour le rendu responsive
options.add_argument("--window-size=1920,1080")

# Initialiser le driver avec les options
driver = webdriver.Chrome(options=options)
driver.get("https://www.example.com")

print(f"Mode headless activé. Titre de la page : {driver.title}")

driver.quit()
```

---

## 19. Page Object Model (POM)

Le **Page Object Model** est un patron de conception (design pattern) qui consiste à structurer le code en créant une classe Python pour chaque page de l'application web. Cela sépare le code de test de la structure HTML brute de la page, facilitant ainsi grandement la maintenance du projet.

### Structure Recommandée :
```
project/
│
├── pages/
│   ├── __init__.py
│   ├── base_page.py
│   └── login_page.py
│
└── test_login.py
```

### 1. `pages/base_page.py`
```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.remote.webdriver import WebDriver

class BasePage:
    def __init__(self, driver: WebDriver):
        self.driver = driver
        self.wait = WebDriverWait(self.driver, 10)

    def find(self, locator: tuple):
        return self.wait.until(EC.visibility_of_element_located(locator))

    def click(self, locator: tuple):
        self.find(locator).click()

    def type(self, locator: tuple, text: str):
        element = self.find(locator)
        element.clear()
        element.send_keys(text)
```

### 2. `pages/login_page.py`
```python
from selenium.webdriver.common.by import By
from pages.base_page import BasePage

class LoginPage(BasePage):
    # Locators définis sous forme de tuples (By, Value)
    USERNAME_INPUT = (By.ID, "login-email")
    PASSWORD_INPUT = (By.ID, "login-password")
    SUBMIT_BUTTON = (By.ID, "submit-btn")

    def login(self, username: str, password: str) -> None:
        self.type(self.USERNAME_INPUT, username)
        self.type(self.PASSWORD_INPUT, password)
        self.click(self.SUBMIT_BUTTON)
```

### 3. `test_login.py` (Script Principal)
```python
from selenium import webdriver
from pages.login_page import LoginPage

# Initialiser le navigateur
driver = webdriver.Chrome()
driver.get("https://www.example.com/login")

try:
    # Utiliser le Page Object Model
    login_page = LoginPage(driver)
    login_page.login("ml_engineer@example.com", "SuperSecurePassword123")
    
    # Vérification
    assert "Tableau de bord" in driver.title
    print("Test de connexion réussi !")
finally:
    driver.quit()
```

---

## 20. Cas d'Utilisation Réels

### 20.1 Scraping de Sites Dynamiques (Collecte de Données)
Extraction d'informations de produits sur une boutique en ligne chargeant ses données via JavaScript.

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import json

driver = webdriver.Chrome()
driver.get("https://www.example.com/products-dynamic")

products = []

try:
    # Attendre que les cartes de produits soient affichées dans la page
    wait = WebDriverWait(driver, 10)
    product_cards = wait.until(
        EC.presence_of_all_elements_located((By.CLASS_NAME, "product-card"))
    )
    
    # Parcourir et extraire les informations clés
    for card in product_cards:
        name = card.find_element(By.CLASS_NAME, "product-title").text
        price = card.find_element(By.CLASS_NAME, "product-price").text
        
        products.append({
            "name": name,
            "price": price
        })
        
    # Enregistrer le jeu de données sous format JSON
    with open("dataset_products.json", "w", encoding="utf-8") as f:
        json.dump(products, f, indent=4, ensure_ascii=False)
        
    print(f"Jeu de données collecté avec succès. {len(products)} produits extraits.")
finally:
    driver.quit()
```

### 20.2 Exécution de Script JavaScript Direct
Pour effectuer des actions complexes comme faire défiler la page web vers le bas afin de charger des éléments en défilement infini (infinite scroll).

```python
from selenium import webdriver
import time

driver = webdriver.Chrome()
driver.get("https://www.example.com/infinite-scroll")

# Faire défiler vers le bas de la page 3 fois de suite
for i in range(3):
    driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
    time.sleep(2) # Attente du chargement du contenu supplémentaire

print("Défilement infini simulé avec succès.")
driver.quit()
```

---

## 21. Bonnes Pratiques Générales

1. **Utilisez toujours un bloc `try...finally` ou un gestionnaire de contexte** : Cela garantit que la commande `driver.quit()` sera exécutée même en cas d'erreur dans votre logique de code, évitant ainsi d'avoir des instances de navigateurs zombies consommant votre RAM.
2. **Ne dupliquez pas les locators** : Centralisez vos sélecteurs en haut de votre script ou dans des classes dédiées (comme vu dans le Page Object Model) afin de ne pas avoir à modifier 50 lignes de code si un site change l'ID de son bouton principal.
3. **Optimisez les temps d'attente (Waits)** : Privilégiez toujours les **Explicit Waits** afin d'avoir un code résilient aux variations de bande passante réseau.
4. **Utilisez des Profils Utilisateurs Réalistes** : Pour du web scraping intensif, configurez un `User-Agent` réaliste pour éviter les blocages immédiats de type Cloudflare/anti-bot.
5. **Préférez CSS Selector à XPath** : Le sélecteur CSS est plus simple à lire, mieux toléré par la plupart des navigateurs et plus performant. Réservez XPath pour les besoins avancés (comme remonter le DOM ou cibler le texte brut).

---

## 22. Limitations et Alternatives

### Limitations de Selenium
* **Vitesse et performances** : Assez lent car il nécessite de charger l'intégralité d'un vrai navigateur (CSS, images, polices, scripts publicitaires).
* **Consommation mémoire** : Chaque instance de WebDriver consomme une quantité significative de RAM et de CPU.
* **Détection Anti-Bots** : Selenium modifie des variables d'environnement JavaScript internes (ex: `navigator.webdriver = true`) facilitant sa détection par des services de sécurité comme Cloudflare ou Akamai.

### Alternatives en Python

| Technologie | Type | Avantages | Inconvénients |
|---|---|---|---|
| **Playwright** | Automatisation de navigateur moderne | Plus rapide que Selenium, support natif de l'asynchronisme, auto-wait puissant. | Écosystème de plugins un peu plus jeune. |
| **BeautifulSoup** | Analyseur HTML statique | Très léger et extrêmement rapide pour parser du HTML brut. | Incapable d'exécuter du code JavaScript. |
| **Requests** | Client HTTP léger | Permet de faire des requêtes directes à des APIs ou serveurs de façon instantanée. | Ne rend pas l'interface visuelle ni le JavaScript. |

---

## 23. Conclusion

Selenium reste un pilier de l'automatisation web et du scraping complexe en Python en raison de sa maturité historique et de son immense communauté. En respectant les bonnes pratiques présentées dans ce guide — notamment l'usage rigoureux du Page Object Model (POM), la priorisation des sélecteurs CSS, et l'exclusion systématique des attentes statiques au profit des Explicit Waits — vous développerez des scripts robustes, fiables et faciles à maintenir pour tous vos projets d'automatisation et de traitement de données pour l'Intelligence Artificielle et le Machine Learning.
