# ROOTMASTER — prototype hybride

Prototype gratuit pour diagnostic d'un Google Pixel 8 (`shiba`) et compagnon Windows.
Il ne contient aucun exploit, aucun binaire Magisk redistribué et ne prétend pas rooter
un appareil verrouillé. Le patch de l'image doit être réalisé sur le téléphone cible
avec l'application Magisk officielle.

## Prérequis

- Android Studio récent compatible Android SDK 37
- JDK 17
- Android SDK Platform 37 et Build Tools 37.x
- Python 3.10+
- Google Android SDK Platform-Tools (adb.exe, fastboot.exe)
- Câble USB de données

## Compiler l'APK

1. Décompresser le projet.
2. Ouvrir le dossier `ROOTMASTER` dans Android Studio.
3. Installer Android SDK Platform 37 via Tools > SDK Manager.
4. Attendre la synchronisation Gradle.
5. Choisir Build > Build Bundle(s) / APK(s) > Build APK(s).
6. APK debug : `app/build/outputs/apk/debug/app-debug.apk`.

Le SDK Android 17 correspond à API 37. Les versions exactes du plugin Gradle et de Compose
peuvent nécessiter une mise à jour proposée par Android Studio si les dépôts ont publié
une révision plus récente.

## Préparer Windows

1. Télécharger Android SDK Platform-Tools depuis:
   https://developer.android.com/tools/releases/platform-tools
2. Extraire par exemple dans `C:\Android\platform-tools`.
3. Ajouter ce dossier à la variable PATH de Windows.
4. Ouvrir PowerShell dans `desktop-companion`.

Commandes:

```powershell
python .\rootmaster.py doctor
adb devices
python .\rootmaster.py device
```

Sur le téléphone, activer Options pour les développeurs > Débogage USB et autoriser
l'ordinateur. `device` lit le codename, le build et la version Android.

## Déverrouillage OEM

Sur le Pixel: Paramètres > À propos du téléphone > appuyer 7 fois sur Numéro de build,
puis Paramètres > Système > Options pour les développeurs > Déverrouillage OEM.

Ne pas continuer si l'option est grisée. Vérifier la politique de l'appareil, son statut
de gestion et la connectivité. Le déverrouillage via l'écran du bootloader efface les
données. ROOTMASTER ne déclenche pas cette opération automatiquement.

## Procédure Magisk contrôlée

1. Sauvegarder les données.
2. Vérifier le build exact dans Paramètres > À propos du téléphone.
3. Télécharger l'image officielle Google qui correspond exactement au build installé:
   https://developers.google.com/android/images
4. Extraire `init_boot.img` de l'archive d'usine, si elle existe pour ce build.
5. Copier `init_boot.img` sur le Pixel.
6. Installer l'APK Magisk officiel depuis:
   https://github.com/topjohnwu/Magisk/releases
7. Dans Magisk > Installer > Sélectionner et patcher un fichier, choisir l'image stock.
8. Récupérer le fichier `magisk_patched-*.img` créé sur le téléphone.
9. Redémarrer en bootloader (`adb reboot bootloader`).
10. Vérifier le produit et le statut:

```powershell
python .\rootmaster.py fastboot-check
fastboot getvar current-slot
```

11. Calculer le SHA-256 de l'image patchée:

```powershell
Get-FileHash .\magisk_patched.img -Algorithm SHA256
```

12. Seulement après confirmation de la correspondance appareil/build/image:

```powershell
python .\rootmaster.py flash-init-boot --image .\magisk_patched.img --sha256 SHA256_HEX
```

La commande exige `shiba`, un bootloader explicitement déverrouillé, un SHA-256 correct
et la saisie manuelle `FLASH INIT_BOOT`. Elle ne déverrouille pas le bootloader.

## Limites connues du prototype

- L'application Android fait le diagnostic matériel local.
- Le compagnon PC gère ADB/Fastboot et un flash explicitement confirmé.
- Le compagnon ne télécharge pas automatiquement un firmware et ne choisit pas
  automatiquement une partition à partir d'un build non vérifié.
- La présence de `su` ou d'un fichier Magisk n'est pas une preuve suffisante de root.
- Android 17 / API 37 et les firmwares Pixel évoluent: tester sur le build exact.
- Une image bêta, QPR ou provenant d'un autre build ne doit pas être flashée par défaut.

## Sécurité et récupération

Ne jamais flasher une image fournie par un tiers ou patchée sur un autre téléphone.
Conserver l'archive officielle correspondant au build et lire les instructions Google
de restauration avant toute modification. Le déverrouillage du bootloader efface les
données et réduit la sécurité du téléphone.
