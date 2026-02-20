![[Pasted image 20260123115040.png]]

Allez dans software packs puis manage softawre pack et choisir x-cube-freertos installer et ensuite dans le middleware chosir le core et le heat (heat 4) et ensuite dans la connectivity et system core s'assurer que tout soit en ordre . 

sauvegarder et générer le code tester la liaison série 

Problème de liaison uart rien ne s'affiche 
sur 2 cartes / 1 carte à un problème.
stack size à revérifier plus tard

Problème résolu  affichage sur le terminal avec uart1 et mise en place de 2 tâches pour allumer les leds avec un delay différent 