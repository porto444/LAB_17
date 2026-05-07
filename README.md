# Writeup : OWASP UnCrackable Mobile Dashboard - Level 3

Ce dépôt documente la résolution du challenge de reverse engineering Android **UnCrackable Level 3** de l'OWASP. L'objectif était de contourner plusieurs couches de sécurité (anti-root, anti-tampering, anti-debug) pour extraire un code secret caché dans une librairie native.

## 🛠 Outils utilisés
*   **Jadx-GUI** : Analyse statique du code Java.
*   **Apktool** : Décompilation et reconstruction de l'APK (Smali).
*   **Ghidra** : Reverse engineering de la librairie native (`libfoo.so`).
*   **Android Build Tools (`apksigner`)** : Signature de l'APK modifié.
*   **ADB** : Installation et tests sur émulateur.
*   **Python** : Script de décodage final (XOR).

---

## 📈 Méthodologie étape par étape

### 1. Analyse Statique et Identification des Protections
L'analyse initiale sous **Jadx-GUI** a révélé plusieurs mécanismes de défense dans la classe `MainActivity` :
*   **Détection Root** : Utilisation de `RootDetection.checkRoot()`.
*   **Vérification d'Intégrité** : Un calcul de CRC sur les fichiers `.dex` et `.so` via la méthode `verifyLibs()`.
*   **Code Natif** : La logique de vérification du mot de passe est déléguée à une librairie nommée `libfoo.so` via l'appel `System.loadLibrary("foo")`.
<img width="1101" height="322" alt="Capture d&#39;écran 2026-05-07 180635" src="https://github.com/user-attachments/assets/38a55d0a-2687-4b01-b94c-ebce9bbef178" />
<img width="587" height="188" alt="Capture d&#39;écran 2026-05-07 180724" src="https://github.com/user-attachments/assets/14847e78-fe53-4471-80a5-7534199a1acb" />


### 2. Contournement des protections (Patch Smali)
L'application se fermait immédiatement suite à la détection de l'émulateur. Pour neutraliser cette sécurité :
1.  **Décompilation** : 
    <img width="1165" height="276" alt="Capture d&#39;écran 2026-05-07 181837" src="https://github.com/user-attachments/assets/e2ac8489-fe16-4898-bc1e-bf13b3f13697" />

2.  **Modification du code Smali** : Dans `smali/sg/vantagepoint/uncrackable3/MainActivity.smali`, j'ai patché la méthode `showDialog` pour qu'elle ne fasse rien et n'arrête pas l'exécution :
    ```smali
    .method private showDialog(Ljava/lang/String;)V
        .locals 3
        return-void  # Neutralisation de la fermeture de l'application
    .end method
    ```
    <img width="1211" height="420" alt="Capture d&#39;écran 2026-05-07 182209" src="https://github.com/user-attachments/assets/a8affb7c-1fe4-441a-833e-7adb89fda754" />

3.  **Reconstruction** :
    <img width="1262" height="231" alt="Capture d&#39;écran 2026-05-07 182459" src="https://github.com/user-attachments/assets/f6ba6f0d-7408-4b5a-9585-bd6a30e430d0" />


### 3. Signature et Installation
Android nécessite un certificat valide. J'ai utilisé la clé de debug d'Android Studio :
<img width="1457" height="120" alt="Capture d&#39;écran 2026-05-07 182933" src="https://github.com/user-attachments/assets/1a81ecbe-0da3-4609-ac02-57e7f35dc39c" />

### 4. Analyse de la librairie native (Ghidra)
Le cœur du challenge résidait dans libfoo.so. En analysant la fonction JNI Java_sg_vantagepoint_uncrackable3_CodeCheck_bar, j'ai identifié une fonction fortement obfusquée contenant une suite de constantes à la fin de son exécution.
En isolant la fin de la fonction, j'ai extrait 24 octets encodés :
.0x1549170f1311081d
.0x15131d5a1903000d
.0x14130817005a0e08
<img width="472" height="280" alt="Capture d&#39;écran 2026-05-07 184444" src="https://github.com/user-attachments/assets/8954f1cf-4b02-4d61-b510-694c025b8d3a" />
<img width="740" height="280" alt="Capture d&#39;écran 2026-05-07 184830" src="https://github.com/user-attachments/assets/ca753f99-c6c3-4de5-a602-79f9d751c125" />

### 5. Extraction du Secret
La clé de chiffrement XOR identifiée dans le code Java était "pizzapizzapizzapizzapizza". J'ai utilisé le script Python suivant pour déchiffrer la suite hexadécimale :
<img width="1635" height="377" alt="Capture d&#39;écran 2026-05-07 185114" src="https://github.com/user-attachments/assets/425fd876-60fb-4902-82ba-a07b23936ef1" />

### Résultat Final
Le code secret permettant de valider le challenge est :
making owasp great again
<img width="366" height="781" alt="Capture d&#39;écran 2026-05-07 185104" src="https://github.com/user-attachments/assets/ae749602-7c89-4ead-bf51-721a8daef227" />


