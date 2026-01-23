
## OctoSPI : c’est quoi ?

L’OctoSPI est une interface série à 8 lignes de données qui permet au STM32 de communiquer avec une mémoire externe comme la MX66. Elle peut fonctionner en Single, Quad ou Octo-SPI.



## Signaux principaux

Les principaux signaux sont :

NCS : sélectionne la mémoire (Chip Select)

CLK : horloge

DQS : sert à aligner les données, surtout en mode rapide

IO0..7 : lignes de données


Ces signaux permettent de synchroniser et d’envoyer correctement les données entre le MCU et la mémoire.



## Protocole Regular-command (MX66)

La communication se fait en plusieurs phases :

Instruction : indique à la mémoire ce qu’elle doit faire (lecture, écriture, effacement)

Address : précise où dans la mémoire agir

Dummy cycles : temps d’attente avant lecture

Data : phase où les données sont vraiment lues ou écrites


Il peut y avoir aussi des octets alternatifs, mais c’est optionnel. Chaque phase peut utiliser différents modes SPI et SDR/DTR.



## Mode DTR

Le mode DTR double la vitesse en envoyant les données sur les fronts montant et descendant de l’horloge. Le signal DQS permet de synchroniser parfaitement les données.



## Modes OctoSPI

L’OctoSPI peut fonctionner de trois façons :

Indirect : transferts via registres OctoSPI, CPU ou DMA

Automatic status-polling : le matériel lit automatiquement le registre d’état

Memory-mapped : la mémoire externe est accessible comme une mémoire interne et peut exécuter du code directement (XIP)


En Memory-mapped, une instance peut gérer jusqu’à 256 Mo.



## Utilisation de la MX66

La MX66 communique via Regular-command et peut être utilisée dans les trois modes. En Memory-mapped, on peut lire et écrire directement par adresse, comme si c’était de la mémoire interne.



## Initialisation de la MX66

Pour l’initialiser :

Configurer les GPIO et l’horloge OctoSPI

Choisir le mode SPI/Quad/Octo et SDR/DTR

Envoyer les commandes d’initialisation en mode Indirect

Configurer les paramètres de lecture : instruction, dummy cycles

 Activer éventuellement le Memory-mapped mode


Après ça, la mémoire est prête à être utilisée.



## Utilisation après init

En Indirect mode, on accède aux données via les registres.

En Memory-mapped, on lit et écrit directement par adresse.  

Toutes les transactions sont gérées automatiquement par l’OctoSPI.


## Écriture et lecture de données dans la flash MX66

La MX66 est une mémoire flash NOR, ce qui impose certaines contraintes :

Une zone mémoire doit être effacée avant d’être écrite

L’écriture se fait par pages (typiquement 256 octets)

L’effacement se fait par secteurs (4 Ko, 64 Ko, etc.)

## Écriture de données dans la flash

L’écriture se fait généralement en mode Indirect et suit les étapes suivantes :

Activation de l’écriture (Write Enable)
Une commande est envoyée pour autoriser les opérations d’écriture ou d’effacement.

Effacement du secteur cible
Un secteur contenant l’adresse à écrire est effacé.

Attente de la fin de l’effacement
Le bit BUSY du registre d’état est surveillé, souvent via le mode Automatic status-polling.

Programmation de la page (Page Program)
Les données sont envoyées à la mémoire à l’adresse choisie.

Attente de la fin de l’écriture
La mémoire indique qu’elle est de nouveau disponible.

Les données à écrire sont stockées dans un tableau en RAM, puis envoyées vers la flash.

## Stockage des données dans un tableau

Avant l’écriture, les données sont préparées dans un tableau en mémoire interne :

uint8_t txBuffer[256] = { 0x10, 0x20, 0x30, 0x40 };


Ce tableau est ensuite transmis à la mémoire MX66 lors de la phase Data de la commande Page Program.

## Lecture des données
Lecture en mode Indirect

Les données lues depuis la flash sont stockées dans un tableau RAM :

uint8_t rxBuffer[256];


Après la lecture, les données sont accessibles directement dans ce tableau.

## Lecture en mode Memory-mapped

En mode Memory-mapped, la mémoire flash est vue comme une zone mémoire classique.
Il est possible de lire les données directement par adresse ou de les copier dans un tableau :

memcpy(rxBuffer, (uint8_t*)OSPI_FLASH_ADDRESS, 256);


Aucune commande SPI n’est nécessaire : l’OctoSPI gère automatiquement les accès.

## Organisation des données dans la flash

Pour une utilisation fiable, la mémoire est généralement organisée en zones :

paramètres de configuration

données applicatives

stockage temporaire ou logs

Cette organisation permet de limiter l’usure de la flash et de structurer les accès mémoire.
