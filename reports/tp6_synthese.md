# Orchestrateur ou intégration continue ? (TP06)

| Critère | Prefect (orchestrateur) | GitHub Actions (CI) |
|-----------------------------------|-------------------------|---------------------|
| Ce qui déclenche l'exécution | Une planification (cron chaque lundi à 6 h), un Quick run depuis l'interface, ou un appel manuel | Un événement Git (`git push`) ou le bouton « Run workflow » (`workflow_dispatch`) |
| Où s'exécute le traitement | Sur notre machine, par un processus qui reste ouvert (`serve`) ou par un worker rattaché à un work pool | Sur une machine Linux (ubuntu-latest) fournie par GitHub |
| Qui surveille, et où | L'équipe MLOps, dans le tableau de bord Prefect (Runs, Deployments, Work Pools) | Les développeurs, dans l'onglet Actions de GitHub et la croix rouge ou coche verte sur le commit |
| Ce qui se passe en cas d'échec | Réessais automatiques sur `rejouer_pipeline` ; si `valider` échoue, le flow passe à Failed et `notifier` ne s'exécute pas | L'étape « Verifier la regle de qualite » sort avec le code 1, le run passe au rouge et le commit est marqué en échec |
| Type de tâche adapté | Traitements récurrents et longs (réentraînement, validation du modèle, notification) | Vérification du code à chaque modification (installation, `dvc repro`, règle de qualité) |
| Ce qui est versionné dans Git | `flow_mlops.py`, `planifier.py`, `prefect.yaml` (déploiement) | `.github/workflows/ci.yml`, `verifier_qualite.py`, `requirements-ci.txt` |

## Conclusion (5 lignes)

La chaîne d'intégration continue (GitHub Actions) valide le code à chaque modification, avant qu'elle n'atteigne le pipeline de production.
L'orchestrateur (Prefect) exécute et surveille les traitements récurrents, ici le réentraînement planifié chaque lundi.
Les deux appliquent la même règle de qualité (F1 ≥ seuil_f1 de `params.yaml`) mais à des moments différents : au push pour la CI, à l'exécution planifiée pour Prefect.
Lors des tests, le modèle dégradé (`max_depth: 2`, F1 = 0.25) a fait échouer le flow Prefect et le run GitHub Actions, ce qui confirme que la règle bloque bien un mauvais modèle.
Les deux outils sont complémentaires : aucun ne remplace l'autre.