

**1. Mémoire "MX66" (Octo-SPI) **


 La référence "MX66" désigne une mémoire de la marque **Macronix**. Sur les cartes STM32U5, on va l'utiliser sur la carte STM32U5G9ZJ.

**La technologie (Octo-SPI) :**

- **SPI classique :** 1 seule route pour les données (1 bit à la fois). C'est lent.

- **Octo-SPI :** 8 routes en parallèle (8 bits à la fois) + on envoie des données à la montée et à la descente de l'horloge.

- **La Taille :**

- Le "1G" dans le nom signifie **1 Gigabit**, Donc : 128Mo

### 2. Comment l'initialiser ? 

La mémoire MX66 démarre en mode SPI (1 fil). Le STM32 doit la configurer pour qu'elle passe en mode "Ferrari" (Octal Double rate)

Voici le scénario précis que votre code doit exécuter :

1. **Réveil (Reset) :** J'envoie une commande pour remettre la mémoire à zéro.
    
2. **Autorisation d'écriture :** On envoie la commande 0x06.
    
3. **Changement de mode  :**

    - Le STM32 envoie une commande à la mémoire (en mode lent SPI) pour écrire dans son **Registre de Configuration **.

    - On y écrit la valeur qui dit : _"Active le mode Octal (8 bits) et le mode DTR (Double Vitesse)"_.


4. **Synchronisation  :**

- On configure le **STM32** pour qu'il sache qu'il doit désormais parler en Octal.

- On règle les "Dummy Cycles" (cycles d'attente). C'est le temps de latence pour aller chercher une information en mémoire..
### 3. Comment l'utiliser ? (Le "Memory Mapped Mode")

Une fois l'initialisation finie, on va utiliser la mémoire externe 

**Le Mappage Mémoire :**

C'est une fonction du STM32 (`HAL_OSPI_MemoryMapped`) qui crée un pont direct.

1. Le STM32 réserve une plage d'adresses pour cette mémoire, 0x90000000.

2. **L'accès :**
    
    - Si on le 1er octet de la mémoire externe, on lit l'adresse `0x90000000`.
    
    - Si on veut lire le 100ème octet, on lit `0x90000064`.
    
    - Le contrôleur Octo-SPI du STM32 fera tout le travail de communication en arrière-plan .


