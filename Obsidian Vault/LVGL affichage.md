On déclare la **mémoire vidéo** (le Framebuffer). On va la découper en trois morceaux pour bien la comprendre.
#### La base : `uint16_t fb[800 * 480];`

C'est la création de la "toile" sur laquelle ton image et ton animation vont être dessinées.

- **`800 * 480`** : C'est la résolution exacte de ton écran (384 000 pixels au total).
    
- **`uint16_t`** : Cela veut dire que chaque pixel prend 16 bits (2 octets) en mémoire. C'est le fameux format couleur **RGB565** (Rouge, Vert, Bleu).
    
- **Le poids total** : 384 000 pixels × 2 octets = **768 Kilo-octets**. C'est un tableau gigantesque pour un microcontrôleur !
    

####  Le rangement : `__attribute__((section(".bss")))`

C'est un ordre direct donné au compilateur (GCC) pour lui dire _où_ ranger ce tableau géant.

- En langage C, la section **`.bss`** est la zone de la RAM réservée aux variables globales qui ne sont pas initialisées (elles valent zéro au démarrage).
    
-  Si tu déclarais ce tableau normalement à l'intérieur de ta fonction `main()`, il irait dans la mémoire "Stack" (la pile). La pile est toute petite (quelques Ko). Ton tableau de 768 Ko la ferait exploser instantanément (c'est le fameux _Stack Overflow).
		// Pour après
- On peut aussi configurer une mémoire externe en allant dans l'ioc Tools>memory management>External Memory> Choisir le nom de la mémoire>Code Generation Compilation> Cochez les 2 >Ctrl+s
- Allez dans Pinout et dans connectivity et mettre le hspi1 en octo spi. Ensuite mettre dans clock prescaler 4.
####  L'optimisation matérielle : `__attribute__((aligned(32)))`

C'est le secret pour que l'écran (le matériel LTDC) puisse lire la mémoire à la vitesse de l'éclair.

- Cette commande force l'adresse de départ du tableau à être un **multiple de 32 octets** (par exemple, l'adresse mémoire finira par des zéros).
    
- **L'explication technique :** Le processeur, le DMA2D et l'écran ne lisent pas les pixels un par un. Ils les aspirent par gros blocs de 32 octets (ce qu'on appelle une "ligne de cache" ou un "burst" sur le bus de données).
- Exemple
__attribute__((section(".bss"))) __attribute__((aligned(32)))
uint16_t fb[800 * 480];//

## Affichage Image
- **`LV_IMG_DECLARE(nom_de_l_image);`** :Placée en haut du fichier, elle prévient le compilateur que l'image existe quelque part dans le projet 
    
- **`lv_image_create(lv_screen_active());`** : Crée un conteneur vide  et l'accroche à l'écran principal en cours.
    
- **`lv_image_set_src(mon_image, &nom_de_l_image);`** : Insère l'image dans le cadre vide 
    
- **`lv_obj_center(mon_image);`** : Pour centrer parfaitement l'objet au milieu de l'écran (On peut aussi utiliser **`lv_obj_align`** ou **`lv_obj_set_pos`** pour des coordonnées précises).
- Code affichage image

LV_IMG_DECLARE(iut);
lv_init();
lv_port_disp_init();
lv_obj_set_style_bg_color(lv_scr_act(), lv_color_black(), 0);// Ecran noir
lv_obj_set_style_bg_opa(lv_scr_act(), LV_OPA_COVER, 0);// Ecran noir 
 lv_obj_t * img1 = lv_image_create(lv_screen_active());
lv_image_set_src(img1, &ballon);
lv_obj_center(img1);

## Pour une animation 

- **Création du Moteur (Callback) :** Vous devez écrire une fonction du type `void anim_cb(void * var, int32_t v)`. À l'intérieur, vous utilisez une fonction de modification :
    
    - **`lv_obj_set_x()`** ou **`lv_obj_set_y()`** pour bouger en horizontale ou en verticale.
        
    - **`lv_image_set_scale()`** pour zoomer.
        
- **`lv_anim_init(&a);`** : Initialise la structure mathématique de l'animation pour la vider de ses déchets mémoire.
    
- **`lv_anim_set_var(&a, mon_image);`** : Indique à LVGL _quel objet_ va subir l'animation.
    
- **`lv_anim_set_exec_cb(&a, anim_cb);`** : Branche le moteur créé au début à cette animation.
    
- **`lv_anim_set_values(&a, debut, fin);`** : Donne les limites (par exemple, de 128 à 400 pour un zoom).
    
- **`lv_anim_set_duration(&a, temps_en_ms);`** : Définit la durée de l'animation. C'est ici que l'on gère la vitesse.
- ### `lv_anim_set_playback_duration(&a, 3000);`

**C'est l'effet aller-retour**
L'animation classique (définie par `lv_anim_set_duration`) s'occupe de l'aller : par exemple, passer de la taille 128 à 400.
    Le "playback", c'est le **trajet retour**. Cette fonction dit à LVGL : _"Une fois que tu as fini de grossir, mets 3000 millisecondes (3 secondes) pour rapetisser et revenir à ta taille de départ"_.
    **Si on ne met pas cette ligne :** L'image mettrait 3 secondes à grossir, puis redeviendrait minuscule d'un seul coup sec, en une fraction de seconde, avant de recommencer.

- `lv_anim_set_repeat_count(&a, LV_ANIM_REPEAT_INFINITE);`
**C'est le mode "Boucle".**
Par défaut, une animation dans LVGL se joue exactement une seule fois, puis elle est détruite de la mémoire.
    Ici, on modifie le compteur de répétitions. La constante `LV_ANIM_REPEAT_INFINITE` est un mot-clé de LVGL qui veut dire "Ne t'arrête jamais". C'est grâce à cela que le zoom et le dézoom s'enchaînent tant que la carte est allumée.
    
- `lv_anim_set_path_cb(&a, lv_anim_path_ease_in_out);`

**La fluidité.**

- C'est l'une des fonctions les plus puissantes pour le design. Par défaut, une animation est "linéaire" : elle avance à une vitesse constante, comme un robot. C'est souvent très laid visuellement.
    
- `ease_in_out` (qui se traduit par "entrée et sortie douces") modifie la courbe de vitesse : le zoom va **démarrer tout doucement**, accélérer au milieu, puis **freiner en douceur** avant de s'arrêter pour faire le chemin inverse. Cela donne un mouvement très organique et naturel.
    
- Il existe d'autres effets très sympas dans LVGL, comme `lv_anim_path_bounce` qui fait faire des rebonds comme une balle, ou `lv_anim_path_overshoot` qui va un peu trop loin puis revient en arrière avec un effet élastique)._
    
- **`lv_anim_start(&a);`** : Lance l'animation.
Code pour l'animation:

lv_obj_t * img1 = lv_image_create(lv_screen_active());
lv_image_set_src(img1, &iut);
// IMPORTANT : On centre l'image pour que le zoom se fasse depuis le milieu
lv_obj_center(img1);

// --- 2. Animation ---
lv_anim_t a;
lv_anim_init(&a);
lv_anim_set_var(&a, img1);

// --- CHANGEMENT 1 : On utilise la fonction ZOOM ---
lv_anim_set_exec_cb(&a, anim_zoom_cb);

// --- CHANGEMENT 2 : Les valeurs de ZOOM ---
// Départ : 128 (Petit, 50% de la taille)
// Arrivée : 400 (Gros, env. 150% de la taille. Max conseillé 512)
lv_anim_set_values(&a, 128, 400);

// Vitesse (un peu plus lente pour bien voir le zoom)
lv_anim_set_duration(&a, 3000); // 3 secondes pour grossir

// Le reste pour faire l'aller-retour infini
lv_anim_set_playback_duration(&a, 1000); // 3 secondes pour rapetisser
lv_anim_set_repeat_count(&a, LV_ANIM_REPEAT_INFINITE);

// 'ease_in_out' rend le début et la fin du zoom plus doux
lv_anim_set_path_cb(&a, lv_anim_path_ease_in_out);
lv_anim_start(&a);

## Amélioration de l'animation 

- **L'optimisation du Compilateur (`-Ofast`) :** C'est un réglage de l'IDE, pour que le CPU fasse les calculs mathématiques assez vite. On fait un clic droit sur le projet >Properties>c/c++ build>settings>MCU/MPU Gcc Compiler>Optimization>Optimization for speed 0fast.
    
- **Le passage au Double Buffering (Tampon Double) :** Au lieu de dessiner directement sous les yeux de l'utilisateur, on dessine sur une toile cachée (fb2), puis on échange les toiles.
    
    - **`lv_display_set_buffers(disp, fb1, fb2, taille, LV_DISPLAY_RENDER_MODE_DIRECT);`** : C'est LA fonction qui change tout. En mode DIRECT avec deux tampons, LVGL ne découpe plus l'image. Il calcule tout l'écran d'un coup.
        
- **Le Flush sans copie (`my_disp_flush`) :** On arrête d'utiliser des boucles `for` ou des `memcpy` qui sont lents. On utilise directement le matériel (le contrôleur LTDC) :
    
    - **`HAL_LTDC_SetAddress_NoReload(&hltdc, (uint32_t)px_map, 0);`** : Pointeur magique
    - . Dit à l'écran d'aller lire la RAM à cet endroit précis.
        
    - **`HAL_LTDC_Reload(&hltdc, LTDC_RELOAD_VERTICAL_BLANKING);`** : La VSYNC (Synchronisation Verticale). Elle empêche l'effet de déchirure de l'écran en attendant que le balayage physique de la dalle soit terminé avant de changer l'image.
        
- **Le battement de cœur (`lv_tick_inc`) :** L'animation ne vivra jamais si le temps ne s'écoule pas. L'appel régulier de **`lv_tick_inc(1);`** dans l'interruption d'un Timer matériel (comme le TIM3) est le chef d'orchestre absolu de la fluidité.