# Explication CI/CD et KPIs pour BobApp

URL du dépôt GitHub : https://github.com/J200W/BobApp

URL DockerHub front : https://hub.docker.com/repository/docker/j200w/bobapp-front/general  
URL DockerHub back : https://hub.docker.com/repository/docker/j200w/bobapp-back/general  

URL SonarCloud front : https://sonarcloud.io/project/overview?id=j200w_BobApp_front  
URL SonarCloud back : https://sonarcloud.io/project/overview?id=j200w_BobApp_back  

## 1. Explication CI/CD pour BobApp

### Backend CI/CD Pipeline

#### Fichier: `.github/workflows/back-end.yml`

**Nom du Workflow**: CI/CD Backend

**Déclencheurs**:
- `push` dans la branche `main`.
- `pull_request` avec les types `opened`, `synchronize`, et `reopened`.

**Jobs**:

1. **test-backend-and-sonar**
   - **runs-on**: `ubuntu-latest`
   - **Objectif**: Tester le backend et analyser avec SonarCloud

   **Étapes**:
   - **Récupérer le code source**:
     ```yaml
       uses: actions/checkout@v4
     ```
   - **Initialisation du JDK 17**:
     ```yaml
       uses: actions/setup-java@v4
       with:
         distribution: 'zulu'
         java-version: 17
     ```
   - **Mettre en cache les packages Sonar**:
     ```yaml
       uses: actions/cache@v4
       with:
         path: ~/.sonar/cache
         key: ${{ runner.os }}-sonar
         restore-keys: ${{ runner.os }}-sonar
     ```
   - **Mettre en cache les packages Maven**:
     ```yaml
       uses: actions/cache@v4
       with:
         path: ~/.m2
         key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
         restore-keys: ${{ runner.os }}-m2
     ```
   - **Build avec Maven**:
     ```yaml
       run: mvn clean install -B -q
     ```
   - **Exécution des tests unitaires et génération du rapport de couverture**:
     ```yaml
       run: mvn test jacoco:report
     ```
   - **Upload du rapport de couverture**:
     ```yaml
       uses: actions/upload-artifact@v4
       with:
         name: jacoco-report
         path: back/target/site/jacoco
         overwrite: true
     ```
   - **Scanner le code avec SonarCloud**:
     ```yaml
       env:
         SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
         GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
       run: mvn -B -q verify org.sonarsource.scanner.maven:sonar-maven-plugin:4.0.0.4121:sonar -Dsonar.projectKey=${{ secrets.SONAR_PROJECT_KEY_BACK }}
     ```

2. **docker_build_and_push**
   - **needs**: `test-backend-and-sonar`
   - **runs-on**: `ubuntu-latest`
   - **Objectif**: Construire et déployer l'image Docker du backend.

   **Étapes**:
   - **Récupérer le code source**:
     ```yaml
     - name: Checkout
       uses: actions/checkout@v4
     ```
   - **Initialisation de Docker Buildx**:
     ```yaml
     - name: Initialisation de Docker Buildx
       uses: docker/setup-buildx-action@v3.3.0
     ```
   - **Connexion à Docker Hub**:
     ```yaml
     - name: Connexion à Docker Hub
       uses: docker/login-action@v3.2.0
       with:
         username: ${{ secrets.DOCKER_USERNAME }}
         password: ${{ secrets.DOCKER_PASSWORD }}
     ```
   - **Build et Push l'image docker du backend**:
     ```yaml
     - name: Build et Push l'image docker du backend
       uses: docker/build-push-action@v6.2.0
       with:
         context: ./back
         file: ./back/Dockerfile
         push: true
         tags: ${{ secrets.DOCKER_USERNAME }}/bobapp-back:latest
     ```

### Frontend CI/CD Pipeline

#### Fichier: `.github/workflows/front-end.yml`

**Nom du Workflow**: CI/CD Frontend

**Déclencheurs**:
- `push` dans la branche `main`.
- `pull_request` avec les types `opened`, `synchronize`, et `reopened`.

**Jobs**:

1. **test-frontend-and-sonar**
   - **runs-on**: `ubuntu-latest`
   - **Objectif**: Tester le frontend et analyser avec SonarCloud

   **Étapes**:
   - **Récupérer le code source**:
     ```yaml
       uses: actions/checkout@v2
     ```
   - **Initialisation de Node.js 21**:
     ```yaml
       uses: actions/setup-node@v2
       with:
         node-version: '21'
     ```
   - **Installation des dépendances**:
     ```yaml
       run: npm install
     ```
   - **Build de l'application**:
     ```yaml
       run: npm run build
     ```
   - **Exécution des tests unitaires et génération du rapport de couverture**:
     ```yaml
       run: npm test
     ```
   - **Scanner le code avec SonarCloud**:
     ```yaml
       uses: SonarSource/sonarcloud-github-action@master
       env:
         SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
         GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
     ```

2. **docker_build_and_push**
   - **needs**: `test-frontend-and-sonar`
   - **runs-on**: `ubuntu-latest`
   - **Objectif**: Construire et déployer l'image Docker du frontend.

   **Étapes**:
   - **Récupérer le code source**:
     ```yaml
     - name: Checkout
       uses: actions/checkout@v2
     ```
   - **Initialisation de Docker Buildx**:
     ```yaml
     - name: Initialisation de Docker Buildx
       uses: docker/setup-buildx-action@v3.3.0
     ```
   - **Connexion à Docker Hub**:
     ```yaml
     - name: Connexion à Docker Hub
       uses: docker/login-action@v3.2.0
       with:
         username: ${{ secrets.DOCKER_USERNAME }}
         password: ${{ secrets.DOCKER_PASSWORD }}
     ```
   - **Build et Push l'image docker du frontend**:
     ```yaml
     - name: Build et Push l'image docker du frontend
       uses: docker/build-push-action@v6.2.0
       with:
         context: ./front
         file: ./front/Dockerfile
         push: true
         tags: ${{ secrets.DOCKER_USERNAME }}/bobapp-front:latest
     ```


### 2. Key Performance Indicators (KPIs)

Afin d'améliorer la qualité de BobApp et d'assurer une gestion optimale du code, voici les KPIs proposés, basés sur le Quality Gate "Sonar Way" de SonarQube. Ces KPIs permettront de surveiller et maintenir la qualité du code, tout en réduisant les bugs et vulnérabilités.

#### 1. Couverture de Code

**Description** : Ce KPI mesure le pourcentage de code testé. Une couverture élevée indique que la majorité du code est passée par des tests, ce qui réduit les risques de bugs non détectés.

**Seuil Minimum** : 80%

**Justification** : Le Quality Gate "Sonar Way" préconise une couverture minimale de 80%, ce qui représente un bon équilibre entre l'effort de test et la détection des bugs potentiels. Ce seuil garantit que la majeure partie du code est testée tout en laissant de la flexibilité pour les parties difficiles à tester.

#### 2. Fiabilité du Code

**Description** : Ce KPI évalue la fiabilité du code en se basant sur le nombre de bugs détectés. SonarQube attribue une note de A à E, A étant la meilleure.

**Seuil Minimum** : A

**Justification** : Un score de fiabilité de A signifie qu'aucun bug critique n'a été introduit. Cela est crucial pour maintenir la stabilité de l'application et offrir une bonne expérience utilisateur. Réduire les bugs récurrents augmentera la satisfaction des utilisateurs.

#### 3. Sécurité du Code

**Description** : Ce KPI mesure la sécurité du code en fonction des vulnérabilités détectées. SonarQube attribue une note de A à E, A étant la meilleure.

**Seuil Minimum** : A

**Justification** : Une note de sécurité de A indique qu'aucune nouvelle vulnérabilité critique n'a été détectée. La sécurité est primordiale pour protéger les données des utilisateurs et maintenir leur confiance dans l'application.

#### 4. Maintenabilité du Code

**Description** : Ce KPI évalue la maintenabilité du code en se basant sur la dette technique. SonarQube attribue une note de A à E, A étant la meilleure.

**Seuil Minimum** : A

**Justification** : Un score de maintenabilité de A garantit que le code est bien structuré et facile à maintenir. Cela réduit le temps de correction des bugs et facilite les évolutions futures du code, essentiel pour Bob, qui dispose de peu de temps pour gérer l'application.

#### 5. Révision des Points Sensibles de Sécurité

**Description** : Ce KPI s'assure que tous les points sensibles de sécurité (Security Hotspots) identifiés ont été revus.

**Seuil Minimum** : 100%

**Justification** : Examiner tous les points sensibles de sécurité garantit une gestion proactive des risques potentiels, améliorant la sécurité globale de l'application.

---

### 3. Analyse des Métriques et des Retours Utilisateurs

#### Analyse des Métriques

L'analyse des métriques de SonarQube et des rapports de couverture de code fournit des informations précieuses sur la qualité du code.

**Backend** :

- **Couverture des Instructions** : 32% (en dessous du seuil de 80%)
- **Couverture des Branches** : 50%
- **Complexité Cyclomatique** : 15
- **Lignes Manquantes** : 45
- **Méthodes Manquantes** : 18
- **Classes Manquantes** : 2

**Analyse** : La couverture de code backend est faible, exposant à un risque de bugs élevé. Un effort est nécessaire pour augmenter la couverture de test, en particulier pour les parties critiques.

**Frontend** :

- **Couverture des Déclarations** : 76.92%
- **Couverture des Branches** : 100%
- **Couverture des Fonctions** : 57.14%
- **Couverture des Lignes** : 83.33%

**Analyse** : La couverture des déclarations du frontend est proche du seuil recommandé de 80%, mais celle des fonctions reste faible à 57.14%. Un effort doit être fait pour tester les fonctions critiques.

#### Analyse des Retours Utilisateurs

Les retours utilisateurs fournissent des informations qualitatives qui complètent les métriques. Voici les principaux problèmes identifiés :

1. **Bug inexistant de suggestion de blague**  
   - **Note** : 1 étoile  
   - **Action** : Considérer l'ajout de cette fonctionnalité à l'avenir.

2. **Bug inexistant de post de vidéo**  
   - **Note** : 2 étoiles  
   - **Action** : Évaluer la pertinence d’ajouter cette fonctionnalité.

3. **Absence de réponses par email**  
   - **Note** : 1 étoile  
   - **Action Prioritaire** : Encourager l’utilisation d’issues sur GitHub pour une meilleure gestion des problèmes.

4. **Déception et suppression des favoris**  
   - **Note** : 2 étoiles  
   - **Action Prioritaire** : Améliorer la qualité générale et ajouter des fonctionnalités attrayantes.

### Actions Prioritaires

1. **Résolution des Bugs Existants** : Corriger les bugs actuels pour améliorer l’expérience utilisateur.
2. **Amélioration des Réponses au Support** : Centraliser les demandes sous forme d’issues sur GitHub.
3. **Augmentation de la Couverture de Code** : Renforcer les tests backend et frontend pour atteindre les seuils de couverture recommandés.
4. **Engagement des Utilisateurs** : Ajouter des fonctionnalités nouvelles comme la suggestion de blagues et le post de vidéos.
5. **Tests en Production** : Mettre en place des procédures de test pour garantir la stabilité de la version déployée.

En appliquant ces actions, nous améliorerons la qualité et la fiabilité de BobApp tout en répondant aux attentes des utilisateurs.