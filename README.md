# -LAB-11-Bypass-de-la-D-tection-de-Root-Android-avec-Frida-Hooks-Java-Natif-
#  Rapport d’audit – Bypass Root Detection avec Frida

##  Informations générales

* **Lab** : Root Detection Bypass using Frida
* **Application** : Root Detection Test App
* **Package** : com.example.rootcheck
* **Date** : 2026-03-25
* **Auditeur** : ilyas
* **Environnement** : Android Emulator (API 29)
* **Outils** : Frida, ADB, Python

---

##  Résumé exécutif

Ce laboratoire avait pour objectif d’analyser et de contourner les mécanismes de détection de root dans une application Android en utilisant Frida.

L’analyse a permis de :

* Identifier plusieurs techniques de détection (Java et natives)
* Mettre en place des hooks pour neutraliser ces contrôles
* Vérifier le contournement avec succès

 **Risque global : ÉLEVÉ**
Une application mal protégée peut être facilement contournée par un attaquant maîtrisant l’instrumentation dynamique.

---

##  Méthodologie

1. Vérification de l’environnement Frida
2. Identification du package cible
3. Analyse des techniques de détection root
4. Implémentation de hooks Java
5. Ajout de hooks natifs
6. Validation du bypass
7. Analyse des logs et debugging

---

##  Techniques de détection identifiées

###  Niveau Java

| Technique    | Description                           |
| ------------ | ------------------------------------- |
| Build.TAGS   | Vérification de "test-keys"           |
| File.exists  | Recherche de fichiers `su`, `busybox` |
| Runtime.exec | Exécution de commandes root           |
| RootBeer     | Librairie de détection                |

---

###  Niveau natif (JNI)

| Technique     | Description                |
| ------------- | -------------------------- |
| open / access | Accès aux fichiers système |
| stat / lstat  | Vérification des chemins   |
| /proc/mounts  | Analyse des partitions     |
| Anti-Frida    | Détection d’outils         |

---

##  Découvertes principales

###  V1 – Détection via Build.TAGS

* Vérification de la présence de `test-keys`

###  V2 – Détection via fichiers système

* Recherche de `/system/xbin/su`, `/system/bin/su`

###  V3 – Détection via Runtime.exec

* Exécution de commandes système (`su`, `which su`)

### V4 – Détection via librairie RootBeer

* Méthodes `isRooted()`

###  V5 – Détection native

* Appels système (open, access)

---

##  Implémentation du bypass

###  Hook Java principal

```javascript id="rj0k3q"
Java.perform(function () {
  const Build = Java.use('android.os.Build');
  Object.defineProperty(Build, 'TAGS', {
    get: function () { return 'release-keys'; }
  });
});
```

---

###  Hook File.exists

```javascript id="3c9y7s"
File.exists.implementation = function () {
  return false;
};
```

---

###  Hook Runtime.exec

```javascript id="2kxlqv"
Runtime.exec.overload('java.lang.String').implementation = function(cmd) {
  return this.exec("echo");
};
```

---

###  Hook RootBeer

```javascript id="wz4fkm"
RootBeer.isRooted.implementation = function () {
  return false;
};
```

---

##  Bypass natif

###  Hook open/access

```javascript id="5x1b9n"
Interceptor.attach(Module.getExportByName(null, "open"), {
  onLeave: function(retval) {
    retval.replace(-1);
  }
});
```

---

##  Exécution

```bash id="6w2v8c"
frida -U -f com.example.rootcheck -l bypass_root.js --no-pause
```

---

##  Résultats

### Avant bypass

*  Root détecté
*  Accès bloqué

### Après bypass

*  Root non détecté
*  Application fonctionnelle

### Logs Frida

```id="w8g3ps"
[+] Hook Build.TAGS
[+] RootBeer.isRooted -> false
[+] Blocked access to /system/xbin/su
```

---

##  Analyse des risques

| Niveau    | Impact                            |
| --------- | --------------------------------- |
|  Élevé  | Contournement complet de sécurité |
|  Moyen  | Accès aux fonctions internes      |
|  Faible | Détection contournable            |

---

##  Mapping OWASP MASVS

| Vulnérabilité                | Référence         |
| ---------------------------- | ----------------- |
| Root detection insuffisante  | MSTG-RESILIENCE-1 |
| Absence d’anti-tampering     | MSTG-RESILIENCE-2 |
| Manque de protection runtime | MSTG-RESILIENCE-3 |

---

##  Recommandations

###  Renforcement des contrôles

* Implémenter plusieurs couches de détection
* Utiliser des checks côté serveur

---

###  Obfuscation

* Obfusquer les classes sensibles
* Utiliser ProGuard / R8

---

###  Anti-Frida

* Détecter instrumentation dynamique
* Scanner ports et processus

---

###  Validation runtime

* Vérifier intégrité de l’application
* Implémenter signature checks

---

##  Résultats globaux

| Étape          | Statut       |
| -------------- | ------------ |
| Détection root | ✅ Identifiée |
| Hook Java      | ✅ Réussi     |
| Hook natif     | ✅ Réussi     |
| Bypass complet | ✅ Réussi     |

---

##  Annexes

### Commandes utilisées

```bash id="8ev1y7"
frida --version
adb devices
frida-ps -Uai
frida -U -f com.example.rootcheck -l bypass_root.js
```

---

### Techniques utilisées

* Instrumentation dynamique
* Hook Java (Frida)
* Hook natif (Interceptor)

---

##  Conclusion

Ce laboratoire démontre que :

* Les mécanismes de détection root peuvent être contournés
* Frida permet une instrumentation puissante
* Une seule méthode de détection est insuffisante

 Une approche multicouche est essentielle pour sécuriser une application Android.

---

##  Déclaration

Je certifie que ce travail a été réalisé :

* Dans un cadre pédagogique autorisé
* Sans exploitation malveillante
* Conformément aux bonnes pratiques de sécurité


**Date** : 2026-03-25

---
