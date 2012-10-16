#TinyImage

La classe tinyImage permet de simplement gérer l'upload d'une image ainsi que son redimensionnement.
Elle a été conçu pour avoir une API la plus simple possible mais offrant une grande souplesse dans son utilisation.

Elle est basé sur la classe upload de Colin Verot : http://www.verot.net et est sous licence GNU en version 2.


##Initialisation

L'initialisation de la classe `TinyImage` se fait en modifiant si nécessaire le chemin d'inclusion de la classe upload à la ligne 4 :
```php
<?php
//load dependency
include ('path/to/upload.php');

```

Il reste à spécifier dans le constructeur (ligne 38 et 39) le chemin où vont être sauvegardés les images, et l'URL vers ce repertoire :
```php
<?php 
 	/*
     * CONFIG :
     * Indicate root path to move uploaded pictures and the associate URL.
     */
   
   $this->imageRootPath = DS . 'upload' . DS . 'images' . DS;
   $this->imageRootURI = '/upload/images/';
```

Vous êtes à présent en situation pour commencer à utiliser la classe TinyImage

##Utilisation

###Téléversement d'une image
La librairie enicImage dispose de deux méthodes pour ajouter des images dans son système  : 
* `add()` 
* `upload()` 

Elles s'utilisent de la façon suivante :
```php
<?php

  //load enicImage class
  enic::to_load('image');
  
  //Instanciate class
  $imageObject = new enicImage();
  
  //image upload
  $imageName = $imageObject->upload($_FILE['myImage']);
    
  //image add from another location
  $imageName = $imageObject->add('/path/to/image/image.jpg');
  
  //you MUST BE save the return of add or upload methods, the imageName is required to get image's URI

```
**ATTENTION** : vous **DEVEZ** sauvegarder le retour des méthodes `add()` ou `upload()`, elles retournent un identifiant unique pour l'image précédemment chargée.
Cet identifiant est requis pour pouvoir procéder aux traitements sur l'image en question.

------

###Traitement et affichage d'une image
Une fois l'image chargée, elle est stockée au sein d'école numérique, il est alors possible de redimensionner l'image selon différentes stratégies.

####Stratégies de redimensionnement
EnicImage redimensionne les images selon trois grandes stratégies (les redimensionnements effectués au sein d'enicImage conservent systématiquement les proportions) :

* Stratégie par défaut : L'image est redimensionnée en fonction du plus grand coté par rapport au paramètres de redimensionnement, le coté restant est calculé en fonction des proportions.
* Stratégie crop : L'image est redimensionnée pour respecter la hauteur et la largeur spécifiée, les parties de l'image qui sont hors du cadre sont supprimées.
* Stratégie fill : L'image est redimensionnée pour respecter la hauteur et la largeur spécifiée, l'image est redimensionnée pour entrer dans le cadre et du blanc est ajouté pour respecter les dimensions demandés.

**Note :** Les stratégies Crop et Fill peuvent être combinées.

------

####Exemples d'utilisation

```php
<?php

  //load enicImage class
  enic::to_load('image');
  
  //Instanciate class
  $imageObject = new enicImage();
  
  /**
   * USAGES
   */
  //get image with default size (250x250) and default strategy
  $imageURI = $imageObject->get('imageUniqueId');
  
  //get image with size of 450x230 and default strategy
  $imageURI = $imageObject->get('imageUniqueId', 450, 230);

  //get image with size of 170x230 and crop strategy
  $imageURI = $imageObject->get('imageUniqueId', 170, 230, 'crop');

  //get image with size of 150x150 and fill strategy
  $imageURI = $imageObject->get('imageUniqueId', 150, 150, 'fill');
   
  //get image with size of 260x230 and combined crop & fill strategy
  $imageURI = $imageObject->get('imageUniqueId', 260, 230, array('crop', 'fill'));
  
  /**
   * DISPLAY RESIZED IMAGE
   */
  
  echo '<img src="'.$imageURI.'" alt="some randome image" />';

```

**Note :** 
* Les traitements effectués sur une image sont réalisés au moment du premier appel à la méthode `get()` puis placés en cache.
* Un appel à la méthode `get()` sans spécifier de hauteur ou de largeur va redimensionner l'image aux valeurs par défaut (250x250) avec la stratégie par défaut.

-----

###Récupérer l'image d'origine
Il est aussi possible de récuperer l'image d'origine (sans aucun traitement) via la méthode `getOriginal()` :
```php
<?php

  //load enicImage class
  enic::to_load('image');
  
  //Instanciate class
  $imageObject = new enicImage();
  
  //get original image
  $imageURI = $imageObject->getOriginal('imageUniqueId');

  //display original image
  echo '<img src="'.$imageURI.'" alt="some randome image" />';

```

-----

###Suppression d'une image
EnicImage propose une méthode pour supprimer une image : `delete()`.

**ATTENTION** : La suppression d'une image entraine la suppression de l'ensemble des images redimensionnées liées.

Exemple d'utilisation :
```php
<?php

  //load enicImage class
  enic::to_load('image');
  
  //Instanciate class
  $imageObject = new enicImage();
  
  //delete image
  $imageObject->delete('imageUniqueId');

```

##Attributs
```php
  <?php
  
  class enicImage
  {
  
    /**
     * @var string indicate the images saved base path (default value: www/static/images)
     */
    protected $imageRootPath;
    
    /**
     * @var string indicate the images base URI (default value: static/images)
     */
    protected $imageRootURI;
    
    /**
     * @var int image's default height in pixel (default value: 250)
     */
    private $height = 250;
    
    /**
     * @var int image's default width in pixel (default value: 250)
     */
    private $width = 250;
    
    /**
     * @var bool activate crop resize strategy (default value: false)
     */
    private $crop = false;
    
    /**
     * @var bool activate fill resize strategy (default value: false)
     */
    private $fill = false;
  
  }
```


##Methodes
```php
  <?php
  
  class enicImage
  {
    
    /**
     * Save a new uploaded image, it's a proxy to add method
     * @param array $_FILES['input'] of the uploaded image
     * @return string image unique id
     */
    public function upload($pathToFile){}
    
    /**
     * Save an image in enicImage's files repository
     * @param string $pathToFile path to the image to save
     * @param boolean $keep_original_image keep the original image (default value: true)
     * @return string image unique id
     */
    public function add($pathToFile, $keep_original_image = true){}
    
    /**
     * get the URI of resized image
     * @param string $imageOriginalName the image's unique id of the wanted image
     * @param int $size_x width (in pixel) of the final resize image (default value: self::$height)
     * @param int $size_y height (in pixel) of the final resize image (default value: self::width)
     * @param array|string $options resize strategy : fill and/or crop, see bellow for usages (default value: null)
     * @return string URI of the resized image
     */
    public function get($imageOriginalName, $size_x = 0, $size_y = 0, $options = array()){}
    
    /**
     * getURI of the original image (not resized)
     * @param string image's unique id
     * @return string URI of the original image
     */
    public function getOriginal($imageOriginalName){}
    
    /**
     * delete an image, and all of the associated sizes
     * @param string image's unique id
     * @return boolean true
     */
    public function delete($imageOriginalName){}
    
  }
```
