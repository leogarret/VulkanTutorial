﻿## À propos

Ce tutoriel vous enseignera les bases de l'utilisation de l'API [Vulkan](https://www.khronos.org/vulkan/) pour les graphismes
et le calcul. Vulkan est une nouvelle API créée par le [groupe Khronos](https://www.khronos.org/) (connu pour OpenGL) qui
fournit une bien meilleure abstraction des cartes graphiques modernes. Cette nouvelle interface vous permet de mieux
décrire ce que votre application souhaite faire, ce qui peut mener à de meilleures performances et à des comportements moins
surprenants comparé à des APIs existantes comme [OpenGL](https://en.wikipedia.org/wiki/OpenGL) et
[Direct3D](https://en.wikipedia.org/wiki/Direct3D). Les concepts derrière Vulkan sont similaires à ceux de
[Direct3D 12](https://en.wikipedia.org/wiki/Direct3D#Direct3D_12) et [Metal](https://en.wikipedia.org/wiki/Metal_(API)),
mais Vulkan a l'avantage d'être complètement cross-platform et vous permet ainsi de développer pour Windows, Linux et
Android en même temps.

Cependant, le prix à payer pour ces avantages est que vous devrez travailler avec une API beaucoup plus verbeuse. Chaque
détail lié à l'API graphique doit être créé à partir de rien par vous-même, dont la mise en place d'un
framebuffer initial et la gestion de la mémoire pour les objets tels que les buffers et les textures. Le travail du driver graphique serait
beaucoup plus limité, ce qui implique un plus grand travail de votre application pour assurer un comportement correct.

Le message à retenir ici est que Vulkan n'est pas fait pour tout le monde. Il cible les programmeurs passionnés par la
programmation graphique de haute performance et de bas niveau et qui sont prêts à y travailler sérieusement. Si vous êtes plus
intéressés par le développement de jeux vidéos, vous devriez plutôt
continuer d'utiliser OpenGL et DirectX, qui ne seront pas dépréciés pour encore un long moment. Une autre
alternative serait d'utiliser un moteur de jeu comme
[Unreal Engine](https://en.wikipedia.org/wiki/Unreal_Engine#Unreal_Engine_4) ou
[Unity](https://en.wikipedia.org/wiki/Unity_(game_engine)), qui pourront être capables d'utiliser Vulkan tout en
exposant une API de bien plus haut niveau.

Cela étant dit, présentons quelques prérequis pour ce tutoriel:

* Une carte graphique et un driver compatibles avec Vulkan ([NVIDIA](https://developer.nvidia.com/vulkan-driver), [AMD](http://www.amd.com/en-us/innovations/software-technologies/technologies-gaming vulkan), [Intel](https://software.intel.com/en-us/blogs/2016/03/14/new-intel-vulkan-beta-1540204404graphics-driver-for-windows-78110-1540))
* De l'expérience avec le C++ (familiarité avec RAII, listes d'initialisation...)
* Un compilateur compatible C++11 (Visual Studio 2013+, GCC 4.8+)
* De l'expérience dans le domaine de la programmation graphique

Ce tutoriel considère que vous n'avez jamais utilisé OpenGL et Direct3D, mais vous devez connaîtreet maîtriser
les bases de la 3D. Les concepts mathématiques utilisés dans ce tutoriel ne seront pas expliqués, donc renseignez vous avant de commencer ce tutoriel. Lisez [ce livre](http://opengl.datenwolf.net/gltut/html/index.html) pour une bonne introduction à la 3D. D'autres ressources pour le développement d'application graphiques sont :
* [Ray tracing en un week-end](https://github.com/petershirley/raytracinginoneweekend)
* [Livre sur le Physical Based Rendering](http://www.pbr-book.org/)
* Une application de Vulkan dans les moteurs graphiques open source de [Quake](https://github.com/Novum/vkQuake) et
[DOOM 3](https://github.com/DustinHLand/vkDOOM3)

Vous pouvez utiliser le C plutôt que le C++ si vous le souhaitez, mais vous devrez utiliser une autre bibliothèque d'algèbre
linéaire et vous serez seul à structurer votre code. Nous utiliserons des possibilités du C++ (RAII, classes) pour
organiser la logique et la durée de vie des ressources. Il existe aussi une [version alternative](https://github
.com/bwasty/vulkan-tutorial-rs) de ce tutoriel pour les développeurs en rust.

Pour faciliter la tâche des développeurs utilisant d'autres langages de programmation et pour acquérir de l'expérience
avec l'API de base, nous allons utiliser l'API C originelle pour travailler avec Vulkan. Cependant, si vous utilisez le C++, vous pourrez
préférer d'utiliser le binding [Vulkan-Hpp](https://github.com/KhronosGroup/Vulkan-Hpp) plus récent, qui permet de s'abstraire du sale boulot,
et permet d'éviter certains types d'erreurs.

## E-book

Si vous préférez lire ce tutoriel en E-book, vous pouvez en télécharger une version EPUB ou PDF ici (anglais uniquement):

* [EPUB](https://raw.githubusercontent.com/Overv/VulkanTutorial/master/ebook/Vulkan%20Tutorial.epub)
* [PDF](https://raw.githubusercontent.com/Overv/VulkanTutorial/master/ebook/Vulkan%20Tutorial.pdf)

## Structure du tutoriel

Nous allons commencer par une approche globale de la manière dont Vulkan fonctionne et le travail que nous aurons à faire pour afficher un
premier triangle à l'écran. Le but de chaque petite étape aura plus de sens quand vous aurez compris leur rôle dans le
fonctionnement global. Ensuite, nous préparerons l'environnement de développement, avec le [SDK Vulkan](https://lunarg.com/vulkan-sdk/), la
[bibliothèque GLM](http://glm.g-truc.net/) pour les opérations d'algèbre linéaire, et [GLFW](http://www.glfw.org/) pour la
création de fenêtre. Ce tutoriel couvrira leur mise en place sur Windows avec Visual Studio, et sur Linux Ubuntu avec
GCC.

Ensuite, nous implémenterons tous les éléments basiques d'un programme Vulkan nécessaires pour l'affichage de votre
premier triangle. Chaque chapitre suivra approximativement la structure suivante:

* Introduction d'un nouveau concept et son but
* Utilisation de tous les appels correspondants à l'API pour leur mise en place dans votre programme
* Abstraction d'une partie de ces appels pour une réutilisation future

Bien que chaque chapitre soit écrit dans un ordre bien précis, il est possible d'utiliser ce tutoriel comme référence pour un ou plusieurs concepts en particuliers.
Toutes les fonctions et les types Vulkan sont liés à leur spécification, vous pouvez donc cliquer dessus pour en
apprendre plus. Vulkan est une API très récente, il peut donc y avoir des erreurs dans la spécification. Vous
êtes encouragés à transmettre vos retours dans [ce dépôt Khronos](https://github.com/KhronosGroup/Vulkan-Docs).

Comme indiqué plus haut, Vulkan est une API assez prolixe, avec de nombreux paramètres, afin de vous fournir un
maximum de contrôle sur le hardware. Ansi, des opérations comme créer une texture sont composées de beaucoup d'étapes
qui doivent être répétées chaque fois. Nous créerons par conséquent notre propre bibliothèque de fonctions tout au
long de ce tutoriel.

Chaque chapitre se conclura avec un lien menant à la totalité du code du chapitre en question. Vous pourrez vous y référer
si vous avez un quelconque doute sur la structure du code, ou si vous rencontrez un bug et que voulez comparer votre version avec celle de ce tutoriel.
Tous les fichiers de code ont été testés sur des cartes graphiques de différents vendeurs afin d'assurer qu'ils fonctionnent.
Chaque chapitre possède également une section pour écrire vos commentaires en relation avec le sujet discuté. Veuillez y
indiquer votre plateforme, la version de votre driver, votre code source, le comportement attendu et celui obtenu pour
nous aider à vous aider.

Ce tutoriel est destiné à être un effort de communauté. Vulkan est encore une API très récente et les meilleures manières
d'arriver à un résultat parfait n'ont pas encore été déterminées. Si vous avez un quelconque retour sur le tutoriel, n'hésitez alors pas à créer une issue ou une pull request sur le [dépôt GitHub](https://github.com/Overv/VulkanTutorial).
Vous pouvez *watch* le dépôt afin d'être notifié des dernières mises à jour du tutoriel.

Après avoir accompli le rituel de l'affichage de votre premier triangle avec Vulkan, nous étendrons le programme pour y
inclure les transformations linéaires, les textures et les modèles 3D.

Si vous avez déjà utilisé une API graphique auparavant, vous devez savoir qu'il y a nombre d'étapes avant d'afficher une géométrie sur l'écran. Il y a beaucoup de ces étapes préliminaires avec Vulkan, mais vous verrez que chacune
d'entre elle est simple à comprendre et n'est pas redondante. Gardez aussi à l'esprit qu'une fois que vous savez
afficher un triangle - certes peu intéressant -, afficher un modèle 3D parfaitement texturé ne nécessite pas tant de
travail supplémentaire, et que chaque étape à partir de ce point est bien mieux récompensée.

Si vous rencontrez un problème en suivant ce tutoriel, vérifiez d'abord dans la FAQ que votre problème et sa solution
n'y sont pas déjà listés. Si vous êtes toujours coincé après cela, demandez de l'aide dans la section des commentaires
du chapitre en lien avec votre problème.

Prêt à vous lancer dans l'aventure de Vulkan ? [Allons-y!](!Overview)
