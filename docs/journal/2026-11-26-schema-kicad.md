---
layout: default
title: "2026-11-26 : schéma de la carte dans KiCad (Eeschema)"
parent: Journal de bord
nav_order: 5
---

# 2026-11-26 : schéma de la carte dans KiCad (Eeschema)

**Séance :** planifiée · **Durée :** 1 h 30
**Présents :** Timothé Froumentin, Victor Fonte Nove Lamotte
**Tutoriels suivis :** [Concevoir la carte de la console](https://doc.makerspace-amiens.fr/workshops/microcontroleur/tutorials/conception-carte-kicad/) (étape 2 et début de l'étape 3), [KiCad : bases d'Eeschema](https://doc.makerspace-amiens.fr/docs/tutorials/software/kicad/kicad-eeschema-bases/) (fait par Victor sur un circuit d'entraînement)

## Fait

- **Tutoriel « bases d'Eeschema » (Victor)** : circuit d'entraînement LED + résistance, avec symboles d'alimentation, PWR_FLAG et ERC.
- **Schéma complet de la carte relié (Timothé)** : XIAO, écran, joystick, deux boutons, connecteur I2C, LED témoin et condensateur de découplage.
- **ERC lancé : 0 erreur**, 2 avertissements (empreintes par défaut de la XIAO et du joystick introuvables).
- **Empreinte de la XIAO** ajoutée au projet, depuis la bibliothèque officielle Seeed Studio (`Seeed-XIAO.pretty`).

## Comment ça marche

### Les outils d'Eeschema utilisés

| Outil | Touche | Rôle |
|---|---|---|
| Étiquette (label) | **L** | donne un nom à un fil : deux broches avec la même étiquette sont reliées, sans fil entre elles |
| Symbole d'alimentation | **P** | `+3V3` et `GND` : tous les symboles du même nom sont reliés entre eux |
| Fil | **W** | relie deux points ; utile quand les broches sont trop serrées pour y poser directement un symbole |
| Croix de non-connexion | **Q** | indique qu'une broche est laissée libre volontairement |
| Déplacer | **M** | déplace un élément (ou seulement son texte) |
| ERC | menu Inspecter | vérifie automatiquement le schéma (broches oubliées, conflits, alimentation) |

Les étiquettes sont **sensibles à la casse** : `joy_x` et `JOY_X` ne sont pas reliées. On a tout écrit en majuscules.

Une étiquette n'est reliée que si son point d'accroche touche **exactement le bout de la broche** : le petit rond au bout de la broche disparaît alors.

### Une seule tension : 3,3 V

Le 5 V de l'USB-C reste à l'intérieur de la XIAO, dont le régulateur fournit le **3,3 V** sur la broche `3V3_OUT`. Tous les modules sont alimentés par ce `+3V3`, car les entrées de l'ESP32-S3 ne supportent pas plus de 3,3 V. La broche `VBUS` (5 V) n'est donc pas utilisée.

## Comment on a fait

### 0. Le tutoriel d'entraînement (Victor)

Avant de toucher au schéma de la carte, Victor a suivi le tutoriel *bases d'Eeschema* dans un projet séparé. Le circuit :

```
+5V ── R1 (220 Ω) ── anode ▷| cathode ── GND
 │                    (LED rouge)           │
PWR_FLAG                                PWR_FLAG
```

- **Symboles d'alimentation** `+5V` et `GND`, reliés aux composants par des fils (touche **W**).
- **Valeurs** : R1 = 220 Ω, LED rouge. Courant : (5 V − 2 V) / 220 Ω ≈ **14 mA**, la valeur typique d'une LED rouge classique.
- **Sens de la LED** : l'anode du côté de la résistance (+5V), la cathode (la barre) du côté de GND.
- **PWR_FLAG** sur le fil +5V et sur le fil GND : sans eux, l'ERC signale « broche d'entrée d'alimentation non alimentée », car les symboles `+5V` et `GND` sont des **entrées** d'alimentation et rien dans le schéma ne les fournit (dans la réalité, c'est une alimentation externe). Le PWR_FLAG dit à l'ERC : « ce fil est alimenté de l'extérieur ».
- **ERC** lancé pour vérifier le circuit.

Ce tutoriel a permis d'apprendre les gestes de base (placer, relier, nommer, vérifier) avant de les appliquer à la carte.

### A. Les signaux de la XIAO (étiquettes)

| Broche XIAO | Étiquette | Relié à |
|---|---|---|
| D0 / D1 / D2 | `JOY_X` / `JOY_Y` / `JOY_SEL` | joystick : Horizontal / Vertical / Select |
| D3 / D4 | `SDA` / `SCL` | J2 broches 3 / 4 + résistances R1 / R2 |
| D5 / D6 | `SW1` / `SW2` | boutons SW1 (gauche) / SW2 (droite), autre patte à GND |
| D7 / D8 / D9 / D10 | `TFT_RESET` / `TFT_DC` / `TFT_SCK` / `TFT_MOSI` | écran J1 broches 4 / 5 / 7 / 6 |

Sur le symbole, les broches ne sont pas dans l'ordre (D1, D2, D0, D3, D4, D6, D5…) : il faut lire le nom de chaque broche.

### B. Alimentation et broches libres

- `+3V3` : XIAO 3V3_OUT, joystick VCC, écran VCC et LED, J2 broche 2, R1, R2, R3, C1.
- `GND` : les trois GND de la XIAO, les deux GND du joystick, écran GND et CS, J2 broche 1, boutons, LED, C1.
- Croix de non-connexion : VBUS, EN, MTDI, MTDO, MTMS, MTCK, USB_DN, USB_DP, VBAT, PAD, et les broches 9 à 14 de l'écran (SDO et tactile).

### C. L'écran (J1, 1 × 14), vérifié avec la sérigraphie du module

| J1 | Module | Schéma |
|---|---|---|
| 1 | VCC | +3V3 |
| 2 | GND | GND |
| 3 | CS | GND (écran toujours sélectionné) |
| 4 | RESET | `TFT_RESET` |
| 5 | DC | `TFT_DC` |
| 6 | SDI (MOSI) | `TFT_MOSI` |
| 7 | SCK | `TFT_SCK` |
| 8 | LED | +3V3 (rétroéclairage toujours allumé) |
| 9 à 14 | SDO, T_CLK, T_CS, T_DIN, T_DO, T_IRQ | non connectées |

### D. Les composants de conception

| Réf. | Valeur | Branchement |
|---|---|---|
| R1, R2 | 4,7 kΩ | entre +3V3 et SDA / SCL (tirage I2C) |
| R3 + D1 | 1 kΩ + LED | +3V3 → R3 → anode → cathode → GND (témoin d'alimentation) |
| C1 | 100 nF | entre +3V3 et GND, à côté de la XIAO (découplage) |
| J2 | 1 × 4 | 1 GND, 2 +3V3, 3 SDA, 4 SCL |

### E. Vérification

1. Après chaque étape : vérification broche par broche que chaque signal relie bien les bons composants, sans court-circuit entre deux signaux.
2. **ERC : 0 erreur.** Les 2 avertissements restants : les symboles de la XIAO et du joystick pointent vers des bibliothèques d'empreintes (`xiao`, `sparkfun-switch`) absentes. Ils disparaîtront à l'attribution des empreintes.
3. Bibliothèque `Seeed-XIAO` (empreintes `XIAO-ESP32-S3-DIP` et `XIAO-ESP32-S3-SMD`, issues de [OPL_Kicad_Library](https://github.com/Seeed-Studio/OPL_Kicad_Library)) ajoutée au projet.

## Pourquoi

- **Étiquettes plutôt que fils** : le schéma reste lisible, chaque signal porte un nom qu'on retrouve dans le code (`PIN_TFT_DC`…).
- **Petits fils sur J1 et J2** : les broches sont espacées d'un seul cran, les symboles se chevauchaient ; avec un fil, chaque symbole a sa place.
- **Tirages I2C de 4,7 kΩ** : le bus I2C ne fait que tirer les lignes vers 0 ; il faut une résistance vers +3V3 pour les remonter. 4,7 kΩ est la valeur classique en 3,3 V : fronts assez rapides, courant faible (3,3 V / 4,7 kΩ ≈ 0,7 mA).
- **LED avec 1 kΩ** : (3,3 V − 2 V) / 1 kΩ ≈ 1,3 mA, suffisant pour voir la LED sans gaspiller de courant.
- **Condensateur de 100 nF près de la XIAO** : petite réserve d'énergie qui absorbe les appels de courant brusques (mise à jour de l'écran) et filtre les parasites. Il devra être placé au plus près de la broche 3V3 sur le PCB.
- **Joystick en 3,3 V** : alimenté en 5 V, il enverrait jusqu'à 5 V sur D0 et D1, au-delà de ce que supportent les entrées de l'ESP32-S3.
- **Boutons sans résistance** : le code utilise `INPUT_PULLUP`, la résistance de tirage interne de l'ESP32-S3 ; appuyer relie la broche à GND (lecture `LOW`, comme sur la breadboard).
- **D6 pour SW2** : D6 est aussi la sortie TX de l'UART0, qui envoie des messages au démarrage. Le bouton ne relie D6 à GND que pendant un appui, ce qui ne gêne pas le fonctionnement normal ; il faut seulement éviter d'appuyer sur SW2 pendant le démarrage. *(Choix à confirmer avec l'enseignant.)*
- **CS de l'écran à GND et rétroéclairage à +3V3** : l'écran est le seul périphérique SPI, il peut rester sélectionné en permanence ; cela libère deux broches de la XIAO.
- **Ordre de J2 (GND, +3V3, SDA, SCL)** : c'est l'ordre des connecteurs I2C Qwiic / STEMMA QT, compatible avec des modules du commerce.
- **PWR_FLAG dans le tutoriel, pas sur la carte** : dans le circuit d'entraînement, le +5V vient d'une alimentation extérieure au schéma, il faut donc un PWR_FLAG. Sur notre carte, toutes les broches des symboles (XIAO comprise) sont « passives » : l'ERC ne signale aucun problème d'alimentation et le PWR_FLAG n'est pas nécessaire.

## Reste à faire

- [ ] **Attribuer les empreintes** (outil *Assigner les empreintes*) : XIAO (`Seeed-XIAO`, version DIP ou SMD à décider avec l'enseignant), joystick, boutons, résistances et condensateur en 1206, LED 1206, connecteurs au pas de 2,54 mm.
- [ ] Relancer l'ERC (objectif : 0 erreur, 0 avertissement non justifié).
- [ ] Remettre le texte « 4.7k » de R1 à côté de son symbole.
- [ ] Remplir le cartouche (titre, date, révision, auteurs).
- [ ] Exporter le schéma en PDF, le mettre dans le dépôt et présenter la **validation 1**.
