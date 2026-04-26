# 🔓 UnCrackable Level 2 — Android Reverse Engineering Writeup

> **Objectif :** Retrouver le secret caché dans l'application Android UnCrackable Level 2 en analysant son code natif.

---

## 🛠️ Outils utilisés

| Outil | Rôle |
|-------|------|
| **JADX-GUI** | Décompilation du code Java de l'APK |
| **Ghidra** | Analyse statique du binaire natif `.so` |
| **Python** | Décodage hexadécimal et inversion de chaîne |

---

## 📋 Étapes

### Étape 1 — Analyse du code Java (JADX)

On ouvre l'APK avec **JADX-GUI** et on localise la classe `CodeCheck`.

On découvre que la validation **n'est pas faite en Java**. La ligne :
```java
System.loadLibrary("foo")
```
indique qu'une bibliothèque native (C/C++) nommée **`libfoo.so`** gère la sécurité.

![Classe CodeCheck dans JADX](images/img_p1_1.png)

---

### Étape 2 — Extraction de la bibliothèque native

On extrait l'APK (comme un ZIP) et on va dans le dossier **`lib/x86/`**.

On récupère le fichier **`libfoo.so`** — c'est lui qui contient le secret et les protections anti-reverse.

![Extraction de libfoo.so](images/img_p1_2.png)

---

### Étape 3 — Analyse statique du binaire (Ghidra)

On charge `libfoo.so` dans **Ghidra** et on cherche la fonction :

```
Java_sg_vantagepoint_uncrackable2_CodeCheck_bar
```

Dans le code décompilé, on repère la fonction **`strncmp`** qui compare ce que l'utilisateur tape avec une valeur stockée à l'adresse `local_34`.

![Décompilation Ghidra avec strncmp](images/img_p2_1.png)

---

### Étape 4 — Récupération des données hexadécimales

En analysant le code Ghidra, on voit que la chaîne secrète est construite via `builtin_strncpy` avec la valeur `"Thanks for all the fish"` encodée en mémoire.

On extrait les octets hexadécimaux correspondants.

---

### Étape 5 — Décodage de l'hexadécimal (Python)

On utilise Python pour transformer les octets hex en texte lisible :

```python
hex_data = "687369660eht6120726f6620736b6e616854"
print(bytes.fromhex(hex_data).decode("ascii"))
```

Résultat :
```
hsif eht lla rof sknahT
```

![Décodage Python](images/img_p2_2.png)

---

### Étape 6 — Inversion de la chaîne (Python)

Le texte est stocké en **Little-Endian** (à l'envers). On inverse avec `[::-1]` :

```python
s = "hsif eht lla rof sknahT"
print(s[::-1])
```

Résultat :
```
Thanks for all the fish
```

![Inversion Python](images/img_p3_1.png)

---

### Étape 7 — Validation dans l'application ✅

On entre **`Thanks for all the fish`** dans l'application UnCrackable Level 2.

L'application affiche **"Success! This is the correct secret."** 🎉

![Succès dans l'application](images/img_p3_2.png)

---

## 🔑 Flag

```
Thanks for all the fish
```

---

## 📌 Résumé de la méthode

```
APK → JADX (Java) → libfoo.so → Ghidra (C natif) → strncmp → hex → Python → flag
```

Le secret était dans la bibliothèque native **C**, invisible depuis Java, stocké à l'envers en mémoire.
