
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