# Guide des procédures de merge Git

## 1. Merger `develop` dans `release`

### Contexte
Quand tu es en **detached HEAD** sur `origin/release` et que tu veux merger les changements de `develop` dans `release`.

### Procédure

#### 1️⃣ Mettre à jour les deux branches
```bash
git fetch origin
```

#### 2️⃣ Basculer sur la branche `release` locale

**Si tu n'as pas encore de branche `release` locale :**
```bash
git checkout -b release origin/release
```

**Sinon :**
```bash
git checkout release
git pull origin release
```

#### 3️⃣ Merger `develop` dans `release`
```bash
git merge origin/develop
```

⚠️ **Attention :** Ça peut te demander de résoudre des conflits si les deux branches ont modifié les mêmes parties du code.

#### 4️⃣ Pousser la branche `release` mise à jour vers le remote
```bash
git push origin release
```

### Alternative avec rebase (historique linéaire)
💡 Si tu veux éviter les merges compliqués et garder un historique linéaire :

```bash
git checkout release
git pull origin release
git rebase origin/develop
git push origin release
```

---

## 2. Merger `features/fix_onboarding` dans `develop`

### Procédure

#### 1️⃣ Mettre à jour tes branches locales
```bash
git fetch origin
```

#### 2️⃣ Aller sur `develop`
```bash
git checkout develop
```

#### 3️⃣ S'assurer que `develop` est à jour
```bash
git pull origin develop
```

#### 4️⃣ Merger `features/fix_onboarding` dans `develop`
```bash
git merge origin/features/fix_onboarding
```

**S'il y a des conflits, résous-les puis :**
```bash
git add .
git commit
```

#### 5️⃣ Pousser la branche `develop` mise à jour
```bash
git push origin develop
```

### Alternative avec rebase (historique linéaire)
💡 Si tu veux éviter un commit de merge et garder un historique linéaire :

```bash
git checkout develop
git pull origin develop
git rebase origin/features/fix_onboarding
git push origin develop
```

---

## 3. Gérer l'éditeur Vim lors des merges

### Problème
Quand tu es dans l'éditeur Vim que Git ouvre pour confirmer le message du merge.

### Solution rapide
1. Tape simplement : `:wq` (pour **write** et **quit**)
2. Appuie sur **Entrée**

Après ça, Git terminera le merge et tu pourras pousser vers le remote.

---

## Commandes de référence rapide

### Merge develop → release
```bash
git fetch origin && git checkout release && git pull origin release && git merge origin/develop && git push origin release
```

### Merge feature → develop
```bash
git fetch origin && git checkout develop && git pull origin develop && git merge origin/features/fix_onboarding && git push origin develop
```

### Rebase develop → release
```bash
git fetch origin && git checkout release && git pull origin release && git rebase origin/develop && git push origin release
```

### Rebase feature → develop
```bash
git fetch origin && git checkout develop && git pull origin develop && git rebase origin/features/fix_onboarding && git push origin develop
```