# C-EV3-API-ROBOT (CEVAR)

API en langage C pour programmer la brique LEGO Mindstorms EV3 sous Linux + EV3Dev,
avec des prototypes inspirés de ROBOTC (`setMotorSpeed(motorA, 50)`, `getMotorEncoder(motorA)`, ...).
Projet M2 ILIADE, sujet 11 (UBO).

## Organisation du dépôt

```
c-ev3-api-robot/
├── include/cevar/
│   ├── cevar.h           # en-tête unique à inclure : #include <cevar/cevar.h>
│   ├── cevar_motor.h     # module moteurs            (implémenté)
│   ├── cevar_sensor.h    # module capteurs           (TODO, étape B)
│   ├── cevar_task.h      # module tâches             (TODO, étape C)
│   ├── cevar_sync.h      # sémaphores                (TODO, étape C)
│   ├── cevar_time.h      # temps (nSysTime, delay)   (TODO, étape C)
│   └── cevar_error.h     # codes d'erreur
├── src/
│   ├── core/             # init, erreurs (cevar_internal.h : interne)
│   ├── hal/              # seul endroit qui connaît ev3dev-c (interne)
│   ├── motor/            # module moteurs            (implémenté)
│   ├── sensor/           # TODO
│   ├── task/             # TODO
│   ├── sync/             # TODO
│   └── time/             # TODO
├── tests/                # test_motor.c : validation sur la brique
├── examples/             # moteurs.c : programme d'exemple
├── docs/                 # module_moteur.md : référence du module moteurs
├── build/                # libcevar.a et exécutables (générés)
└── Makefile
```

Les couches ne dépendent que de celle du dessous : programme utilisateur → `include/cevar` → `src/<module>` → `src/hal` → ev3dev-c.

## État d'avancement

| Étape | Fonctions | État |
|---|---|---|
| A. Moteurs | `setMotorSpeed`, `getMotorEncoder`, `getMotorRPM`, `resetMotorEncoder` | Écrit. Logique vérifiée sur un moteur simulé (PC). **À valider sur la brique.** |
| B. Capteurs | `getTouchValue`, `getColorRGB`, `getUSDistance`, `getGyroDegrees`, `getGyroRate`, `resetGyro` | Prototypes + TODO |
| C. Tâches, sémaphores, temps | `startTask`, `stopTask`, `stopAllTask`, `abortTimeslice`, `releaseCPU`, `semaphore*`, `clearTimer`, `delay`, `sleep`, `nSysTime` | Prototypes + TODO |

## Compiler

Depuis la racine du dépôt (Docker + `evlic` / `evec` installés, voir `INSTALLATION.txt` de captoc) :

```
make            # bibliothèque + test + exemple
make lib        # build/libcevar.a
make tests      # build/test_motor
make examples   # build/moteurs
```

Équivalent sans `make` (mêmes commandes que celles déjà utilisées) :

```
mkdir -p build
evlic $(find src -name '*.c') -Iinclude -Wall -Wextra -o build/libcevar.a
evec tests/test_motor.c -Iinclude build/libcevar.a -pthread -o build/test_motor
evec examples/moteurs.c -Iinclude build/libcevar.a -pthread -o build/moteurs
```

`-pthread` est nécessaire à l'édition de liens (verrous des modules).

## Exécuter sur la brique

```
scp build/test_motor build/moteurs robot@ev3dev.local:~/
ssh robot@ev3dev.local ./test_motor
```

Moteur de test sur le port A (roue libre), port D vide. Pour `moteurs`, moteurs A et B.

## Conventions

- Un seul en-tête : `#include <cevar/cevar.h>`, puis `cevar_init()` au début et `cevar_exit()` à la fin.
- Les fonctions gardent la forme ROBOTC : valeur de retour directe, unités ROBOTC.
- En cas d'échec, une fonction qui renvoie une valeur renvoie 0 ; la cause se lit avec `cevar_getLastError()` (propre à chaque tâche).
- Ce qui est interne à un fichier est `static` ; les symboles internes partagés sont préfixés `cevar_`.

## Écarts avec l'ancienne organisation

Les anciens fichiers `cevar/include/cevar.h` et `cevar/source/cevar.c` sont remplacés par cette arborescence (`-Icevar/include` devient `-Iinclude`). Les anciennes fonctions `cevar_motor_set_speed`, `cevar_motor_run_timed` et `cevar_motor_stop` (vitesses en degrés/s) ne sont pas reprises : elles sont remplacées par `setMotorSpeed` (puissance en %).
