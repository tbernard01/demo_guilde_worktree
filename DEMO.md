# 🎬 Script de démo — Git Worktrees

## Situation de départ

Tu es sur `feature/add-cross-validation`, en train d'ajouter de la cross-validation
à ton script d'entraînement. Les modifs ne sont pas encore commitées.

```bash
git status
# → modified: train.py
```

**Dis à la salle** : *"Je suis en plein milieu de mon travail. Un collègue me suggère
d'essayer XGBoost. Sans worktree, je dois stash, changer de branche, etc.
Avec une worktree, je crée un espace isolé en une commande."*

---

## Étape 1 — Créer la worktree

```bash
git worktree add -b experiment/xgboost ../worktree-demo-xgboost main
```

**Montre** :
```bash
git worktree list
# worktree-demo-repo         [feature/add-cross-validation]  ← modifs intactes
# worktree-demo-xgboost      [experiment/xgboost]            ← copie propre de main
```

---

## Étape 2 — Passer sur la worktree XGBoost

```bash
cd ../worktree-demo-xgboost
```

Remplace le contenu de `train.py` :

```bash
cat > train.py << 'EOF'
from sklearn.datasets import load_breast_cancer
from xgboost import XGBClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

X, y = load_breast_cancer(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

model = XGBClassifier(n_estimators=100, random_state=42, eval_metric='logloss')
model.fit(X_train, y_train)

acc = accuracy_score(y_test, model.predict(X_test))
print(f"XGBoost       →  accuracy: {acc:.4f}")
EOF
```

---

## Étape 3 — Lancer les deux en parallèle

**Ouvre deux terminaux côte-à-côte et lance simultanément :**

Terminal 1 (RandomForest) :
```bash
cd worktree-demo-repo && python train.py
# RandomForest  →  accuracy: 0.9649
```

Terminal 2 (XGBoost) :
```bash
cd worktree-demo-xgboost && python train.py
# XGBoost       →  accuracy: 0.9737
```

**Dis à la salle** : *"Les deux tournent en même temps, sur des branches différentes,
dans des dossiers différents — un seul `.git`."*

---

## Étape 4 — Vérifier que rien n'a bougé

```bash
cd ../worktree-demo-repo
git status
# → modified: train.py   ← mes modifs de cross-val sont toujours là
```

---

## Étape 5a — XGBoost gagne → on ramène le commit

```bash
cd ../worktree-demo-xgboost
git add train.py
git commit -m "experiment: XGBoost approach"

cd ../worktree-demo-repo
git cherry-pick experiment/xgboost
```

## Étape 5b — Pas concluant → suppression propre

```bash
git worktree remove ../worktree-demo-xgboost
git branch -D experiment/xgboost
# Zéro trace, zéro branche parasite
```

---

## Message de conclusion

> *"Ce que font Conductor, Codex et VS Code Copilot sous le capot,
> c'est exactement ça : un `git worktree add` par agent.
> Vous venez d'apprendre l'infrastructure cachée derrière ces outils."*
