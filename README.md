# Boutique Diayma – TD .NET

- **Étudiant :** Papa Ousmane Mané
- **GitHub :** Ousmaaane
- **École :** École Supérieure Polytechnique (ESP), UCAD – Département Génie Informatique
- **TD :** .NET, séance du 03/11/2026 (année 2026/2027)
- **Chargé de cours :** E. H. Ousmane Diallo
- **Objectif :** prendre en main l'EDI (saisir, exécuter et déboguer du code)

Code de départ : https://github.com/devehod/BoutiqueDiayma2025 (branche `master`).

---

## 1. Récupérer et exécuter le code

J'ai cloné le dépôt du prof puis lancé `dotnet build Diayma.sln`. La première restauration NuGet a pris environ 6 min 30, parce que le projet tire tout le méta-package `Microsoft.AspNetCore.All 2.0.6`. La compilation passe avec **0 erreur et 8 avertissements** :

- `NETSDK1138` : la cible `netcoreapp2.0` n'est plus prise en charge ;
- `NU1902` / `NU1903` : packages avec des vulnérabilités connues (`Microsoft.AspNetCore.All 2.0.6`, `Microsoft.NETCore.App 2.0.0`). C'est l'avertissement que l'énoncé dit d'ignorer.

Par contre, au premier lancement l'application ne démarre pas :

```
You must install or update .NET to run this application.
Framework: 'Microsoft.NETCore.App', version '2.0.0' (x64)
The following frameworks were found:
  10.0.14 at [C:\Program Files\dotnet\shared\Microsoft.NETCore.App]
```

Sur ma machine je n'ai que .NET 10. Pour la lancer sans installer un vieux runtime, j'ai autorisé le passage à la version majeure suivante avec la variable `DOTNET_ROLL_FORWARD` :

```bash
cd P2FixAnAppDotNetCode
DOTNET_ROLL_FORWARD=Major ASPNETCORE_ENVIRONMENT=Development dotnet bin/Debug/netcoreapp2.0/Diayma.dll
```

(sous PowerShell : `$env:DOTNET_ROLL_FORWARD="Major"` avant la commande)

Avec ça, le site démarre (`Now listening on...`) et la page d'accueil renvoie bien la liste des 9 produits. J'ai mis la même variable dans `.vscode/launch.json` pour pouvoir déboguer avec F5.

Remarque : le dossier `wwwroot/lib` (Bootstrap, Font Awesome, jQuery) n'est pas dans le dépôt. Il devait être rempli par Bower (`bower.json`, `.bowerrc`). Le site marche quand même, mais sans mise en forme et sans validation côté client.

## 2. Les projets de la solution

`Diayma.sln` ne contient qu'**un seul projet** :

| Projet | Fichier | Rôle |
|---|---|---|
| Diayma | `P2FixAnAppDotNetCode/Diayma.csproj` | Application web ASP.NET Core MVC (SDK `Microsoft.NET.Sdk.Web`) : la boutique en ligne |

Il n'y a pas de projet de tests ni de bibliothèque à part. Tout est dans ce projet, rangé par dossiers :

- `Program.cs`, `Startup.cs` : démarrage de l'hôte web, injection de dépendances, pipeline HTTP ;
- `Controllers/` : `ProductController` (liste des produits, page d'accueil), `CartController` (panier), `OrderController` (formulaire de commande), `LanguageController` (changement de langue) ;
- `Components/` : deux view components appelés depuis le layout, `CartSummary` (résumé du panier en haut) et `LanguageSelector` (choix de la langue en bas) ;
- `Models/` : `Product`, `Cart`/`CartLine`, `Order`, plus `Repositories/` (données en mémoire, pas de base de données) et `Services/` (logique métier) ;
- `Views/` : vues Razor et `_Layout.cshtml` ;
- `Resources/` : fichiers `.resx` pour la traduction (en, fr, et wo que j'ai ajouté).

Il n'y a pas de migrations : les données sont générées en dur dans `ProductRepository.GenerateProductData()`. Le dossier `Views/Home` est exclu de la compilation dans le `.csproj`. Sa vue utilise d'ailleurs `p.Quantity`, une propriété qui n'existe pas dans `Product`.

## 3. Version du SDK .NET utilisée

Dans `P2FixAnAppDotNetCode/Diayma.csproj`, ligne 4 :

```xml
<TargetFramework>netcoreapp2.0</TargetFramework>
```

Le projet vise donc **.NET Core 2.0**, avec les packages :

- `Microsoft.AspNetCore.All` 2.0.6 ;
- `Microsoft.Extensions.Localization` 2.1.1 ;
- l'outil `Microsoft.VisualStudio.Web.CodeGeneration.Tools` 2.0.3.

Il n'y a **pas de `global.json`**, donc aucune version de SDK n'est imposée et c'est le SDK le plus récent installé qui est utilisé. À l'origine, ce projet a été fait avec un SDK .NET Core 2.x. Le `.sln` indique Visual Studio 17.13.

## 4. Installer le SDK

`dotnet --list-sdks` sur ma machine :

```
10.0.403 [C:\Program Files\dotnet\sdk]
```

et `dotnet --list-runtimes` ne montre que les runtimes 10.0.14.

Ce que j'ai vérifié en pratique :

- **compiler** : le SDK 10.0.403 suffit, il télécharge le package `Microsoft.NETCore.App 2.0.0` depuis NuGet ;
- **publier en win-x64 self-contained** : marche aussi avec le SDK 10 (voir tâche 9) ;
- **lancer en mode Debug** : il faut normalement le runtime .NET Core 2.0, que je n'ai pas. J'ai utilisé `DOTNET_ROLL_FORWARD=Major` (tâche 1) et l'application tourne sur le runtime 10.0.14.

Je n'ai donc rien installé. Si on veut le faire tourner sur son vrai runtime, il faut installer le runtime .NET Core 2.0 ou 2.1 depuis les archives de Microsoft (https://dotnet.microsoft.com/download/dotnet/2.1). Une appli `netcoreapp2.0` passe automatiquement sur la 2.1, qui est la version mineure suivante. Ces versions ne sont plus supportées et ont des failles connues, c'est pour ça que j'ai préféré éviter.

## 5. Stocker le code dans mon dépôt GitHub

```bash
git clone https://github.com/devehod/BoutiqueDiayma2025
cd BoutiqueDiayma2025
git remote remove origin
git remote add origin https://github.com/Ousmaaane/exodotnet.git
git push -u origin master
```

J'ai gardé l'historique du prof (2 commits) et j'ai ajouté mes commits par-dessus. Le `.gitignore` exclut bien les dossiers générés : `[Bb]in/` et `[Oo]bj/` (lignes 21-22), `.vs/` (ligne 26) et `publish/` (ligne 139). Le dossier `publish` de la tâche 9 n'est donc pas envoyé sur GitHub.

Dépôt : https://github.com/Ousmaaane/exodotnet

## 6. Exploration de l'application : 2 bugs

J'ai testé l'accueil, l'ajout au panier, la suppression, le formulaire de commande vide puis rempli, et le changement de langue. Il n'y a ni pagination ni filtre par catégorie dans cette application (`Product` n'a pas de catégorie), donc je n'ai rien pu tester de ce côté.

### Bug 1 : le total et la moyenne du panier ne tiennent pas compte de la quantité

**Reproduction :**
1. Sur l'accueil, cliquer deux fois sur « Ajouter au panier » pour *XIAOMI Redmi 13C* (88 000).
2. Cliquer une fois sur « Ajouter au panier » pour *Clé USB - 64GB* (1 000).
3. Ouvrir le panier.

**Obtenu :** les sous-totaux sont bons (176 000 et 1 000), mais **Total = 89 000** et **Moyenne = 44 500**. Le résumé en haut affiche aussi « 3 item(s) 89 000 ».

**Attendu :** Total = 2 × 88 000 + 1 000 = **177 000**, et une moyenne par article de 177 000 / 3 = **59 000**.

**Cause :** `P2FixAnAppDotNetCode/Models/Cart.cs`
- ligne 66 : `return GetCartLineList().Sum(x => x.Product.Price);` additionne le prix une seule fois par ligne. Il faudrait `x.Product.Price * x.Quantity`.
- ligne 77 : `GetCartLineList().Average(x => x.Product.Price)` fait la moyenne des lignes et non des articles.

### Bug 2 : le stock ne diminue jamais après une commande

**Reproduction :**
1. Mettre 2 × *XIAOMI Redmi 13C* et 1 × *Clé USB* dans le panier.
2. « Passer la commande », remplir Nom, Adresse, Ville, Pays, puis « Terminer la commande ». La page « Merci ! » s'affiche.
3. Revenir sur l'accueil.

**Obtenu :** le stock affiché est toujours 10 pour le XIAOMI et 20 pour la clé USB.

**Attendu :** 8 et 19.

**Cause :** dans `Startup.cs` ligne 32, le dépôt est enregistré en `AddTransient<IProductRepository, ProductRepository>()`, donc il est recréé à chaque fois qu'on en a besoin. Or le constructeur (`Models/Repositories/ProductRepository.cs` lignes 15-16) refait `_products = new List<Product>()` puis `GenerateProductData()` sur une liste **statique**. La mise à jour faite par `UpdateProductStocks` (ligne 53) est donc écrasée à la requête suivante. En traçant les appels (voir tâche 8), j'ai bien vu `ProductRepository..ctor` s'exécuter à chaque requête, même sur `/Cart` et `/Order`.

Je n'ai pas corrigé ces deux bugs, c'était facultatif.

### Autres choses remarquées

- **Messages de validation non traduits.** En envoyant le formulaire de commande vide, on voit les clés brutes `ErrorMissingName`, `ErrorMissingAddress`, `ErrorMissingCity`, `ErrorMissingCountry` et `CartEmpty` au lieu des phrases. Les vues, elles, sont bien traduites. J'ai regardé les ressources compilées : elles s'appellent `Diayma.Resources.Controllers.OrderController.fr.resources`, parce que l'assembly s'appelle `Diayma`. Mais les classes sont dans le namespace `P2FixAnAppDotNetCode`, donc le localiseur ne trouve pas le bon nom. En plus, `Order.fr.resx` est rangé dans `Models/ViewModels` alors que la classe `Order` est dans `Models`.
- **Prix avec « ¤ ».** En français, les prix s'affichent `88 000,00 ¤`. La culture `fr` est neutre, elle n'a pas de devise, alors que les prix sont en francs CFA.
- **Espagnol.** Choisir « Espagnol » met un cookie `es`, mais `es` n'est pas dans les cultures supportées et il n'y a pas de `.resx` espagnol. La page reste donc dans la langue du navigateur.
- **Panier partagé.** `Cart` est enregistré en `AddSingleton` (`Startup.cs` ligne 29), donc tous les visiteurs du site partagent le même panier. Je l'ai vu dans le code, je ne l'ai pas testé avec deux navigateurs.
- **`FindProductInCartLines`** (`Cart.cs` ligne 89) planterait avec une `NullReferenceException` si le produit n'est pas dans le panier, à cause de `.FirstOrDefault().Product`. Mais cette méthode n'est appelée nulle part dans l'application, donc je n'ai pas pu le reproduire dans le site.

## 7. Points d'arrêt

Les 5 lignes demandées existent bien. Ce sont toutes des affectations dans un **constructeur** : le framework les exécute quand il crée l'objet par injection de dépendances.

| | Fichier : ligne | Code | Quand on y arrive |
|---|---|---|---|
| a | `Components/CartSummaryViewComponent.cs:12` | `_cart = cart as Cart;` | À chaque page affichée avec le layout : `_Layout.cshtml` ligne 22 appelle `Component.InvokeAsync("CartSummary")` et MVC crée le composant. Je l'ai vu sur `/`, `/Cart` et `/Order`. |
| b | `Controllers/ProductController.cs:15` | `_productService = productService;` | Sur la page d'accueil `/` (route par défaut Product/Index), après que la DI a construit `ProductService` et `ProductRepository`. |
| c | `Controllers/OrderController.cs:17` | `_cart = pCart;` | Quand on va sur `/Order` (bouton « Passer la commande »), à l'envoi du formulaire et sur `/Order/Completed`. |
| d | `Controllers/CartController.cs:15` | `_cart = pCart;` | Sur `/Cart`, et quand on clique « Ajouter au panier » ou « Supprimer ». |
| e | `Startup.cs:20` | `Configuration = configuration;` | Une seule fois, au démarrage : pendant `Build()` dans `Program.BuildWebHost`, avant même que le serveur écoute. |

Pour les poser, il faut ouvrir le fichier, placer le curseur sur la ligne et appuyer sur **F9** (ou cliquer dans la marge à gauche). Ensuite on lance avec F5 (profil « Diayma (Debug) » dans `.vscode/launch.json`). Dans VS Code, les points d'arrêt ne s'écrivent pas dans `launch.json`, il faut les poser à la main.

## 8. Chemin d'exécution jusqu'à l'affichage des produits

Pour ne pas deviner l'ordre, je l'ai vérifié. Dans une copie du projet, j'ai ajouté sur les lignes concernées un appel qui écrit la pile d'appels (`System.Diagnostics.StackTrace`), sur la même ligne pour ne pas décaler les numéros. Ensuite j'ai lancé le site et appelé `/`. L'ordre et les numéros de ligne du tableau viennent de ces traces. Par contre, le choix F10 / F11 / Maj+F11 est mon raisonnement sur ce qu'on veut voir à chaque étape : je ne l'ai pas enregistré pas à pas dans Visual Studio. Le comportement suppose l'option « Uniquement mon code » activée, ce qui est le réglage par défaut.

| # | Namespace | Classe | Méthode | Fichier : ligne | Ce qui se passe | Pas à pas et pourquoi |
|---|---|---|---|---|---|---|
| 1 | `P2FixAnAppDotNetCode` | `Program` | `Main` | `Program.cs:10` | Point d'entrée. La ligne appelle `BuildWebHost(args)` puis `.Run()`. | **F11** : je veux entrer dans `BuildWebHost`, c'est mon code. |
| 2 | `P2FixAnAppDotNetCode` | `Program` | `BuildWebHost` | `Program.cs:14-16` | `WebHost.CreateDefaultBuilder` (Kestrel, appsettings, logs), `UseStartup<Startup>()` puis `Build()`. C'est dans `Build()` que tout Startup s'exécute. | **F11** sur `Build()` : le code du framework est sauté et on s'arrête dans le constructeur de `Startup`. Si on tombe dans du code Microsoft, **Maj+F11** pour en sortir. |
| 3 | `Microsoft.AspNetCore.Hosting.Internal` | `WebHost` | `Initialize` → `BuildApplication` | (framework) | Le framework crée `Startup` par DI (`StartupLoader.LoadMethods`), puis appelle `ConfigureServices` et `Configure`. | Rien à voir ici : **Maj+F11** si le débogueur y entre. |
| 4 | `P2FixAnAppDotNetCode` | `Startup` | constructeur | `Startup.cs:20` (point d'arrêt e) | Stocke l'`IConfiguration` reçue. | **F10** : simple affectation. |
| 5 | `P2FixAnAppDotNetCode` | `Startup` | `ConfigureServices` | `Startup.cs:28-59` | Enregistre les services : localisation, `Cart` et `LanguageService` en singleton, services et dépôts en transient, cache, session, MVC avec localisation des vues, et cultures supportées. | **F10** ligne par ligne : ce sont des méthodes d'extension du framework, ça ne sert à rien d'y entrer. |
| 6 | `P2FixAnAppDotNetCode` | `Startup` | `Configure` | `Startup.cs:65-69` | Construit le pipeline : `UseStaticFiles`, `UseRequestLocalization`, `UseSession`, `UseMvc`. | **F10**, sauf à la ligne 69 où **F11** permet d'entrer dans la lambda des routes, qui est mon code. |
| 7 | `P2FixAnAppDotNetCode` | `Startup` (lambda) | `<Configure>b__5_0` | `Startup.cs:71-73` | Déclare la route `{controller=Product}/{action=Index}/{id?}`. C'est elle qui envoie `/` vers `ProductController.Index`. Elle est exécutée tout de suite par `UseMvc`. | **F10** puis **Maj+F11** pour revenir dans `Configure`. |
| 8 | `P2FixAnAppDotNetCode` | `Program` | `Main` | `Program.cs:10` (`.Run()`) | Le serveur Kestrel démarre et attend (`Now listening on...`). | **F10** : `Run()` bloque. La suite se passe à l'arrivée de la requête, sur un autre thread. Il faut **F5** jusqu'au prochain point d'arrêt. |
| 9 | `Microsoft.AspNetCore.*` | `HostingApplication`, `StaticFileMiddleware`, `RequestLocalizationMiddleware`, `SessionMiddleware`, `RouterMiddleware`, `MvcRouteHandler`, `ControllerActionInvoker` | `Invoke` / `RouteAsync` | (framework) | La requête `GET /` traverse les middlewares dans l'ordre de `Configure`. `/` n'est pas un fichier statique, la culture est choisie (cookie ou navigateur), la session est chargée, puis le routage choisit Product/Index. On voit tout ça dans la pile d'appels. | Pas de code à moi : on ne fait pas de pas à pas, **F5**. Si on y tombe, **Maj+F11**. |
| 10 | `P2FixAnAppDotNetCode.Models.Repositories` | `ProductRepository` | constructeur | `ProductRepository.cs:15-16` | Pour créer le contrôleur, la DI construit d'abord `ProductService`, donc `ProductRepository` (qui recrée les 9 produits) et `OrderRepository`. Ça arrive **avant** le constructeur du contrôleur. | **F10**. On peut faire **F11** sur `GenerateProductData()` pour voir les données se créer. |
| 11 | `P2FixAnAppDotNetCode.Controllers` | `ProductController` | constructeur | `ProductController.cs:15` (point d'arrêt b) | Récupère `IProductService` et `ILanguageService`. | **F10** : affectations. |
| 12 | `P2FixAnAppDotNetCode.Controllers` | `ProductController` | `Index` | `ProductController.cs:21` | Appelle `_productService.GetAllProducts()`. | **F11** : pour suivre où viennent les produits, je rentre dans mon service. |
| 13 | `P2FixAnAppDotNetCode.Models.Services` | `ProductService` | `GetAllProducts` | `ProductService.cs:29` | Appelle le dépôt et convertit le tableau en `List<Product>`. | **F11** pour entrer dans le dépôt. |
| 14 | `P2FixAnAppDotNetCode.Models.Repositories` | `ProductRepository` | `GetAllProducts` | `ProductRepository.cs:43-44` | Filtre `Stock > 0`, trie par nom et renvoie un tableau. | **F10**, puis **Maj+F11** pour remonter dans le service puis le contrôleur. Pas besoin d'entrer dans LINQ. |
| 15 | `P2FixAnAppDotNetCode.Controllers` | `ProductController` | `Index` | `ProductController.cs:22` | `return View(products);` crée un `ViewResult`. Le rendu est fait ensuite par le framework (`ViewResultExecutor` puis `RazorView.RenderAsync`). | **F10**. Ensuite on est dans le framework, donc **F5** vers un point d'arrêt dans la vue, ou **Maj+F11**. |
| 16 | `AspNetCore` (classe générée) | `_Views_Product_Index_cshtml` | `ExecuteAsync` | `Views/Product/Index.cshtml:4` et `27-43` | Titre « Liste des produits », puis boucle `foreach` sur les produits (nom, description, prix formaté avec `ToString("c")`, stock, bouton Ajouter). La vue est exécutée **avant** le layout. | **F10** dans la vue (on peut poser un point d'arrêt dans un `.cshtml`). |
| 17 | `AspNetCore` (classe générée) | `_Views_Shared__Layout_cshtml` | `ExecuteAsync` | `Views/Shared/_Layout.cshtml:22` | Le layout entoure la vue (`RenderBody()`) et appelle `Component.InvokeAsync("CartSummary")`. | **F11** sur la ligne 22 : on arrive dans le composant du panier. |
| 18 | `P2FixAnAppDotNetCode.Components` | `CartSummaryViewComponent` | constructeur, puis `Invoke` | `CartSummaryViewComponent.cs:12` (point d'arrêt a), puis ligne 17 | Récupère le panier (singleton) et renvoie la vue `Shared/Components/CartSummary/Default.cshtml`. | **F10**, puis **Maj+F11** pour revenir au layout. |
| 19 | `P2FixAnAppDotNetCode.Components` | `LanguageSelectorViewComponent` | `Invoke` | `LanguageSelectorViewComponent.cs:10` (appelé depuis `_Layout.cshtml:33`) | Affiche le formulaire de choix de langue dans le pied de page. Ensuite le HTML est envoyé au navigateur et les produits s'affichent. | **F10** puis **F5** pour finir la requête. |

Ce que je retiens : les 5 points d'arrêt sont dans des constructeurs, donc on ne les atteint pas en appelant une méthode depuis mon code. C'est le framework qui crée les objets, au démarrage pour `Startup`, à chaque requête pour les contrôleurs et les composants. Le plus pratique est de les poser puis de faire **F5** de l'un à l'autre, et de garder F11 pour mes propres méthodes (contrôleur → service → dépôt).

## 9. Déploiement en exécutable Windows

Dans `P2FixAnAppDotNetCode` :

```bash
dotnet publish -c Release -r win-x64 --self-contained true -o publish
```

Résultat : le dossier `publish/` contient **`Diayma.exe`** avec tout le runtime .NET Core 2.0 embarqué (`coreclr.dll`, `hostfxr.dll`...), soit 1 004 fichiers et environ 100 Mo. J'ai lancé `Diayma.exe` sans aucune variable d'environnement et sans runtime 2.0 installé : il affiche `Hosting environment: Production` et `Now listening on: http://localhost:5000`, et la page d'accueil répond (code 200). J'ai aussi vérifié que la version wolof marche dans l'exe.

Pour l'utiliser : double-cliquer sur `Diayma.exe` puis ouvrir http://localhost:5000 dans le navigateur.

Dans `Properties/PublishProfiles/FolderProfile.pubxml`, le `PublishUrl` pointait vers `F:\Diayma` (un disque qui n'existe pas chez moi). Je l'ai remplacé par `publish\`. Mais j'ai constaté qu'avec `dotnet publish -p:PublishProfile=FolderProfile` en ligne de commande, `PublishUrl` n'est pas utilisé : la sortie va dans `bin\Release\netcoreapp2.0\win-x64\publish\`. C'est pour ça que je passe par `-o publish`. Le `PublishUrl` sert quand on publie depuis Visual Studio.

## 10. Lien de l'exécutable

J'ai compressé le contenu de `publish/` dans `Diayma-win-x64.zip` (41,3 Mo).

Lien de téléchargement : LIEN_ICI

## 11. Partie optionnelle

### a) Langue wolof

J'ai ajouté le wolof comme troisième langue en gardant les cultures du français (`fr-FR`, `fr`) :

- `Startup.cs` : ajout de `wo-SN` et `wo` dans les cultures supportées. `UseRequestLocalization` était déjà en place et lit le cookie de culture ;
- `LanguageService.SetCulture` : l'option « Wolof » donne la culture `wo-SN` ;
- sélecteur de langue : nouvelle option « Wolof » (libellé ajouté dans les `.resx` en et fr) ;
- un fichier `.wo.resx` à côté de chaque `.fr.resx` (vues Produit, Panier, Commande, composants CartSummary et LanguageSelector, Order, OrderController).

Les vues utilisent `IViewLocalizer` (injecté dans `_ViewImports.cshtml`), qui est la version pour les vues d'`IStringLocalizer`. Je n'ai pas eu à toucher aux vues, sauf pour l'option du sélecteur.

Résultat : en choisissant Wolof, le cookie devient `c=wo-SN|uic=wo-SN`. L'accueil affiche « Limu marsandiis yi », « Tur », « Njëg », « Yokk ci pañe bi », le panier « Sa pañe », « Njëg lépp », etc. Comme `wo-SN` a le franc CFA comme monnaie, les prix s'affichent en **F CFA** (par exemple `F CFA 88.000`). Pour l'instant c'est la seule langue où le prix est dans la bonne monnaie.

Limite : les messages de validation et le message « panier vide » ont aussi leur `.wo.resx`, mais ils restent affichés comme des clés, à cause du même problème de namespace qu'en français (voir tâche 6). Mes traductions mériteraient aussi d'être relues par un locuteur wolof.

### b) Commits

- `Ajout de la config de débogage VS Code` : `launch.json` et `tasks.json` pour déboguer avec F5 ;
- `Ajout de la langue wolof (wo-SN)` ;
- `Profil de publication : dossier publish au lieu de F:\Diayma` ;
- `Ajout du README avec les réponses du TD`.

### c) Google Classroom

Le lien du dépôt est déposé sur Google Classroom : https://github.com/Ousmaaane/exodotnet
