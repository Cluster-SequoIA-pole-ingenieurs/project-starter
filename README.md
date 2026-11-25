# Socle Python SequoIA

Ce dépôt fournit un point de départ pour un projet d’ingénierie Python sur
GitHub ou GitLab. Le code se trouve dans `src/` et les tests dans `tests/`.

## Créer et personnaliser le dépôt

Créer le dépôt à partir du modèle sur GitHub ou GitLab lorsqu’il est disponible.
Pour un dépôt existant, intégrer les fichiers utiles dans une branche et adapter
les contrôles à l’environnement scientifique en place.

- Renseigner le nom, la description et les dépendances dans `pyproject.toml`.
- Si `src/sequoia_project/` est renommé, adapter le test d’import, la cible de
  couverture et, le cas échéant, l’import final du workflow de construction.
- Vérifier la compatibilité de Python avec les bibliothèques scientifiques.
  Aligner `.python-version`, `requires-python`, Ruff, Mypy et la version utilisée en CI.
- Adapter ce README avec l’objectif du projet, un exemple exécutable et les
  modalités de maintenance.

## Développement

Prérequis : Git et [uv](https://docs.astral.sh/uv/getting-started/installation/).
Python 3.12 est le choix initial, à adapter à l'environnement scientifique visé.

```bash
uv sync
uv run --locked mypy
uv run --locked pytest -m 'not gpu and not integration and not slow' --cov
```

Versionner `uv.lock`, créé par `uv sync`. Les vérifications suivantes et la CI
utilisent `--locked` pour détecter une divergence entre manifeste et verrou.
La configuration `.pre-commit-config.yaml` permet d’installer le hook et de vérifier les
fichiers suivis par Git avec les commandes suivantes.

```bash
uv run --locked pre-commit install
uv run --locked pre-commit run --all-files
```

Si un hook corrige un fichier, relire et indexer la correction, puis relancer
la commande.

Ajouter le premier test de comportement dès le premier incrément fonctionnel.
Ne pas interpréter une CI verte sur le squelette comme une validation du projet.

Pytest trouve le code dans `src/` grâce à `pythonpath` dans `pyproject.toml`.
Pour lancer un module du projet depuis la racine avec Bash, utiliser par exemple
`PYTHONPATH=src uv run --locked python -m sequoia_project.mon_module` après avoir
créé `mon_module.py`.

## Tests spécifiques

Les marqueurs `gpu`, `integration` et `slow` sont déclarés mais exclus de la
commande CPU ci-dessus. Les marquer n'empêche pas leur exécution avec un simple
`pytest` ; utiliser explicitement la sélection appropriée.

Décrire ici les prérequis et commandes des tests scientifiques ou d'intégration
lorsqu'ils existent, et leurs conditions d'exécution. Les tests ordinaires ne
doivent télécharger ni modèles ni jeux de données, ni soumettre de jobs.

## Activer la CI et rendre les contrôles obligatoires

### Sur GitHub

Créer le dépôt distant avec la visibilité convenue, ajouter son URL comme
remote, puis pousser le premier commit selon la procédure de l'équipe.
Le [workflow GitHub Actions](.github/workflows/ci.yml) est inclus dans le modèle.
Vérifier que GitHub Actions est autorisé et que le premier workflow passe. Le workflow
doit fournir les contrôles `Quality` et `CPU tests`.

Dans **Settings → Rules → Rulesets** (ou **Branches**, selon l'interface),
protéger la branche par défaut :

- Exiger une pull request et au moins une approbation humaine.
- Exiger les contrôles `Quality` et `CPU tests`, après leur premier run.
- Bloquer les force-push et la suppression de la branche ; limiter les bypass.

Vérifier les noms de contrôles réellement affichés par GitHub avant de les
sélectionner. Ces réglages ne sont pas activés par la simple copie des fichiers.
Leur disponibilité dépend de l'offre GitHub et de la visibilité du dépôt ;
si une option manque, voir avec l'administrateur et documenter la règle de revue.

Références : [verrouillage uv](https://docs.astral.sh/uv/concepts/projects/sync/),
[protection de branche GitHub](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches).

### Sur GitLab

Copier le [modèle GitLab CI](extras/gitlab-ci.yml) à la racine du dépôt
cible sous le nom `.gitlab-ci.yml`. Utiliser un runner capable d'exécuter des
jobs dans une image conteneur. Aligner `image: python:3.12` sur la version Python retenue pour le projet.
Le job `quality` utilise les hooks définis dans `.pre-commit-config.yaml`.

Les jobs `quality` et `cpu-tests` s'exécutent en parallèle pour les merge
requests, les pushes sur la branche par défaut et les pipelines lancés depuis
l'interface sur une branche. Le rapport JUnit affiche les résultats des tests
dans GitLab. Le typage est exécuté par `quality`, après pre-commit.

Protéger la branche par défaut et bloquer les pushes directs. Dans
**Settings → Merge requests → Merge checks**, activer **Pipelines must succeed**.
Configurer une approbation humaine si l'offre GitLab le permet. Pour réutiliser
le formulaire de PR, copier son contenu dans
`.gitlab/merge_request_templates/Default.md`.

[Déclenchement des pipelines](https://docs.gitlab.com/ci/yaml/workflow/) ·
[Contrôles avant fusion](https://docs.gitlab.com/user/project/merge_requests/auto_merge/#require-a-successful-pipeline-for-merge).

## Construire un paquet si le projet doit être installé

Le socle exécute les contrôles directement depuis le dépôt. Pour une bibliothèque
ou un outil destiné à être installé, ajouter ces tables au `pyproject.toml`.
Placer `packaging` dans la table `[dependency-groups]` existante à côté de `dev`.

```toml
[build-system]
requires = ["hatchling>=1.27,<2"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/sequoia_project"]

[dependency-groups]
packaging = ["build>=1,<2", "hatchling>=1.27,<2", "twine>=7,<8"]
```

Adapter le nom du paquet et retirer `pythonpath = ["src"]` de la configuration
Pytest pour que les tests utilisent l'installation réalisée par `uv sync`.
Régénérer et versionner `uv.lock`. Copier le
[workflow de construction](extras/package.yml) dans
`.github/workflows/package.yml` du projet et adapter son import final. Il valide
les distributions et installe la wheel dans un environnement vierge. Si la
distribution fait partie des livrables, rendre `Package` obligatoire après son
premier passage.

## Références à compléter

- Cadrage validé, critères d'acceptation et politique IA : référence approuvée.
- Données et modèles : modalités d'accès, licences et versions de référence.
- Responsable de maintenance et procédure de transfert.
- Licence du code : à convenir avant diffusion.

Créer l'AGENTS.md une fois le cadrage et la politique IA validés.
