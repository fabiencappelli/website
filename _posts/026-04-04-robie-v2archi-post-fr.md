---
title: "Architecture de la v2"
date: 2026-04-04
lang: fr
project: robie
image:
categories: [robie]
summary: "Tentative de visualisation de ce à quoi la v2 pourrait ressembler"
---

En essayant de réfléchir au comportement que je veux vraiment avoir, je comprends que mon approche naïve était incomplète. Il va falloir que Robie arrive en fait à écouter en même temps qu'il "lit". Parce qu'on va vouloir l'interrompre :

- pour qu'il change d'histoire
- pour qu'il change d'action
- pour régler le volume, même, car pour l'instant il n'y a rien pour pouvoir contrôler le volume de Robie

Pour amorcer la réflexion, rien de tel qu'un petit schéma

## Flow

<div class="mermaid">

flowchart TB
Start([Start]) --> Idle[Idle: waiting for wake word]

    Idle --> Wake[/Wake word said/]
    Wake --> Listen[/Record or listen live/]
    Listen --> DetectIntent[Process intent]
    DetectIntent --> ConfirmIntent[/Confirm intent/]
    ConfirmIntent --> IsConfirmIntent{Intent confirmed?}

    IsConfirmIntent -- No --> Listen
    IsConfirmIntent -- Yes --> IsIntent{Which intent?}

    IsIntent -- Read a story --> ReadingInit[Enter reading mode]
    IsIntent -- Take a note --> NotePrompt[/Please record the note/]
    IsIntent -- Other request --> HandleOther[Handle other intent]

    NotePrompt --> NoteListen[/Record note/]
    NoteListen --> NoteSave[Save note]
    NoteSave --> Idle

    HandleOther --> Idle

    subgraph ReadingMode [Reading mode]
        ReadingInit --> StartPlayback[Start audio playback]
        StartPlayback --> ReadingLoop[Reading active]

        ReadingLoop --> CheckCommand{Command detected?}
        ReadingLoop --> CheckTime{Is it midnight?}
        ReadingLoop --> EndOfStory{Story finished?}

        CheckCommand -- No --> ReadingLoop
        CheckCommand -- Stop --> StopPlayback[Stop playback]
        CheckCommand -- Volume up --> VolumeUp[Increase volume]
        CheckCommand -- Volume down --> VolumeDown[Decrease volume]

        VolumeUp --> ReadingLoop
        VolumeDown --> ReadingLoop

        CheckTime -- No --> ReadingLoop
        CheckTime -- Yes --> Shutdown[Shutdown device]

        EndOfStory -- No --> ReadingLoop
        EndOfStory -- Yes --> ExitReading[Exit reading mode]
    end

    StopPlayback --> Idle
    ExitReading --> Idle

</div>

## Conséquences

Le point central, c’est que **Reading n’est pas une action ponctuelle**.
C’est un **mode actif long**, pendant lequel plusieurs choses doivent exister en même temps :

- la lecture audio continue
- l’écoute de commandes de contrôle
- la surveillance de l’heure
- la capacité à interrompre proprement la lecture

Autrement dit, mon système ne peut plus être pensé comme une simple chaîne linéaire du type :

```text
wake → écoute → STT → action → fin
```

Il doit devenir un système avec **activité persistante + événements concurrents**.

### Première contrainte : Concurrence

Comme je ne veux pas découper la lecture en petits bouts, la lecture doit pouvoir continuer **pendant qu’autre chose se passe**.

Ça implique une forme de concurrence, typiquement :

- multithread
- ou processus séparés
- ou boucle événementielle plus élaborée

Mais dans tous les cas, on quitte la logique “une seule boucle qui fait tout dans l’ordre”.

Concrètement, il faudra probablement au minimum distinguer :

- un composant qui gère la lecture
- un composant qui écoute le micro
- un composant qui traite les commandes
- un composant qui surveille l’heure
- un orchestrateur qui décide quoi faire

## Deuxième contrainte : Communication inter-composants propre

Dès qu'on a plusieurs activités en parallèle, on doit définir comment elles communiquent.

Par exemple :

- le module micro détecte “stop”
- il doit prévenir le module lecture
- le module horloge détecte minuit
- il doit déclencher un arrêt global
- le module lecture termine le fichier
- il doit notifier le système pour revenir à `Idle`

Donc je ne peux plus me contenter de fonctions qui s’appellent les unes les autres de manière simple.
Il faut une logique de type :

- événements
- files de messages
- drapeaux d’état
- objets de synchronisation

Sinon je vais très vite entrer dans du spaghetti.

## Troisième contrainte : Vrai modèle d’état

Mon schéma dit implicitement qu’on n’est plus seulement dans “faire une action”, mais dans “être dans un état”.

Par exemple :

- `Idle`
- `Listening`
- `Reading`
- `Note recording`
- peut-être plus tard `Thinking`
- peut-être `Shutting down`

Et pendant `Reading`, certaines commandes sont autorisées :

- stop
- volume up
- volume down

alors que d’autres ne le sont peut-être pas, ou pas de la même manière.

Donc il faut modéliser explicitement :

- l’état courant
- les transitions autorisées
- ce qui se passe quand un événement arrive dans tel ou tel état

Sinon j'aurai des comportements flous du genre :
“que fait Robie si on lui parle pendant qu’il lit ?”
“qu’arrive-t-il si minuit tombe pendant une commande volume ?”

## Quatrième contrainte : Interruption propre

Une lecture audio continue, ça veut dire qu’il faudra savoir :

- arrêter immédiatement
- mettre en pause éventuellement
- changer le volume à chaud
- sortir sans laisser le système audio dans un état bancal

Donc le lecteur audio ne pourra pas être une simple commande bloquante lancée sans contrôle.
Il faudra un composant pilotable, avec des commandes du type :

- start
- stop
- pause
- set_volume

Et ces commandes devront être sûres même si elles arrivent à n’importe quel moment.

## Cinquième contrainte : la reconnaissance vocale ne peut plus être pensée comme avant

Dans la boucle conversationnelle classique, on fait :

- on enregistre
- puis on transcrit
- puis on agit

Mais ici, pendant la lecture, il faut détecter **en continu** des commandes très courtes.

Donc je ne fais plus de la STT “classique” seulement.
Je fais plutôt une écoute de contrôle en continu, probablement avec :

- vocabulaire réduit
- logique de commandes limitées
- détection robuste et rapide

Donc le problème n’est plus :
“transcrire une demande ouverte”

mais plutôt :
“repérer vite et proprement quelques ordres critiques”

C’est un autre type de besoin.

## Sixième contrainte : Risque que Robie s’entende lui-même

C’est sans doute l’un des gros défis cachés du mode lecture.

Si Robie lit à voix haute pendant qu’il écoute, alors le micro risque de capter :

- sa propre lecture
- de la réverbération
- des voix d’enfants
- du bruit ambiant

Donc je devrai penser à des garde-fous :

- commandes courtes très spécifiques
- seuils / détection adaptée
- peut-être wake word secondaire en mode lecture
- ou réglages micro/volume/placement physique

Le diagramme n’en parle pas, mais la formalisation de `Reading` comme mode interactif implique directement ce problème.

## Septième contrainte : Logique de priorités

Tous les événements n’ont pas le même poids.

Par exemple :

- `Shutdown at midnight` doit sans doute être prioritaire
- `Stop playback` est très prioritaire
- `Volume up` est moins critique
- `Story finished` est un événement normal

Donc il faudra définir une politique :

- qui interrompt quoi
- qui gagne en cas de collision
- dans quel ordre on traite les événements

Sans ça, risques de comportements bizarres.

## Huitième contrainte : Séparer comportement et implémentation

Le schéma est excellent parce qu’il formalise le comportement attendu.
Mais il force aussi une distinction importante :

- **niveau fonctionnel** : ce que Robie doit faire
- **niveau technique** : comment on le réalise

Dans ce cas, la formalisation dit déjà qu’il faudra probablement :

- un lecteur audio contrôlable
- une écoute en parallèle
- une surveillance horaire autonome
- un système d’événements
- une gestion d’état explicite

Même si on n’a pas encore choisi :

- thread
- callback
- queue
- process

## En résumé

La formalisation entraîne ceci :

Robie ne peut plus être développé comme un simple pipeline séquentiel.
Le mode lecture impose une architecture **concurrente, pilotée par événements, avec gestion d’état explicite**.

Plus concrètement, ça veut dire :

- plusieurs activités doivent vivre en même temps
- elles doivent communiquer proprement
- le système doit connaître son état courant
- la lecture doit être interrompable à tout moment
- la détection vocale pendant lecture devient un problème spécifique
- il faut gérer les priorités et les transitions proprement
