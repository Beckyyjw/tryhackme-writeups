
# 🔐 Cryptosystem Challenge — RSA Fermat Attack

## 📌 Description

Dans ce challenge, nous interceptons une communication entre plusieurs acteurs (Rivest, Shamir, Adleman).
Un fichier Python est récupéré, contenant une implémentation RSA ainsi qu’un message chiffré.

Objectif : **retrouver la clé secrète et déchiffrer le flag.**

---

## 🧠 Analyse du code

Le code fourni :

```python
def primo(n):
    n += 2 if n & 1 else 1
    while not isPrime(n):
        n += 2
    return n

p = getPrime(1024)
q = primo(p)
n = p * q
e = 0x10001
d = inverse(e, (p-1) * (q-1))
c = pow(bytes_to_long(FLAG.encode()), e, n)
```

### 🔍 Problème identifié

La ligne critique est :

```python
q = primo(p)
```

👉 Cela signifie que `q` est **le premier nombre premier suivant `p`**.

Donc :

* `p ≈ q`
* les deux facteurs sont **très proches**

---

## ⚠️ Vulnérabilité

RSA repose sur la difficulté de factoriser `n = p * q`.

Cependant, si :

[
p \approx q
]

Alors une attaque appelée **factorisation de Fermat** devient efficace.

---

## 💥 Attaque utilisée : Fermat Factorization

On exploite l’identité :

[
n = a^2 - b^2 = (a - b)(a + b)
]

Avec :

* ( a \approx \sqrt{n} )
* ( b^2 = a^2 - n )

Quand ( p ) et ( q ) sont proches, cette méthode converge très rapidement.

---

## ⚙️ Exploitation

### Étapes :

1. Factoriser `n` avec Fermat
2. Retrouver `p` et `q`
3. Calculer :
   [
   \varphi(n) = (p-1)(q-1)
   ]
4. Calculer la clé privée :
   [
   d = e^{-1} \mod \varphi(n)
   ]
5. Déchiffrer :
   [
   m = c^d \mod n
   ]

---

## 🧪 Script d'exploitation

```python
from math import isqrt

n = ...
c = ...
e = 65537

def fermat_factor(n):
    a = isqrt(n)
    if a * a < n:
        a += 1

    while True:
        b2 = a*a - n
        b = isqrt(b2)
        if b*b == b2:
            return a - b, a + b
        a += 1

def long_to_bytes(x):
    return x.to_bytes((x.bit_length() + 7) // 8, 'big')

p, q = fermat_factor(n)

phi = (p - 1) * (q - 1)
d = pow(e, -1, phi)

m = pow(c, d, n)

flag = long_to_bytes(m)

print(flag.decode())
```

---

## 🎯 Résultat

Le flag est récupéré après déchiffrement :

```
FLAG{...}
```

---

## 📚 Leçon à retenir

❌ Mauvaise pratique :

* Générer `q` comme étant le prochain nombre premier après `p`

✅ Bonne pratique :

* Générer `p` et `q` **indépendamment et aléatoirement**

---

## 🚀 Compétences mobilisées

* Cryptographie RSA
* Analyse de vulnérabilité
* Factorisation (Fermat)
* Python scripting
* CTF problem solving

---

## 💡 Conclusion

Ce challenge illustre une erreur classique en cryptographie :
**des paramètres mal choisis peuvent compromettre complètement la sécurité d’un système pourtant robuste.**

---

✍️ *Write-up réalisé dans le cadre d’un entraînement en cybersécurité (CTF).*
