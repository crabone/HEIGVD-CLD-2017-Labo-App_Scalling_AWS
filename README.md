# LABORATOIRE 2 - ÉVOLUTIVITÉ D'UNE APPLICATION WEB SUR AWS

Lors du [précédent laboratoire](https://github.com/crabone/HEIGVD-CLD-2017-Labo-IaaS),
nous avions dû déployer une application web (Drupal 7) sur l'infrastructure de
cloud computing d'Amazon (AWS). Ici, nous allons rendre notre application web
évolutive (scalable) par l'utilisation du service web d'Amazon, RDS. De plus,
nous allons nous initier au principe de redondance par l'utilisation de
plusieurs instances EC2 et d'un repartisseur de charge (Load Balancer).

## ÉTUDIANTS

* FRANCHINI Fabien
* DONGMO NGOUMNAÏ Annie Sandra

## TÂCHE 1: CRÉATION D'UNE BASE DE DONNÉE RDS

Dans ce chapitre, nous allons préparer le terrain pour utiliser le service de
base de données relationnelles proposé par Amazon (RDS). Pour ce faire nous
allons créer une instance spécifique à l'utilisation de base de données.

[[[[[[[[Détaillé les avantages de cette approche.]]]]]]]]

Premièrement, nous nous rendons sur le tableau de bord permettant de gérer les
instances RDS. Pour ce faire, dans la page d'accueil, il faut cliquer sur le
menu "RDS" de la catégorie "Database".

![RDS creation step-by-step](assets/images/01-rds_creation_step_1.png)

Avant de créer l'instance, nous souhaitons créer un "Security Group" adapté à
son utilisation, à savoir permettre les connexions entrantes par le port 3306.
Nous remarquons que le service permettant la création de groupes spécifiques
aux bases de données, n'est pas disponible pour notre région.

![RDS creation step-by-step](assets/images/01-rds_creation_step_2.png)

De ce fait, nous créons un "Security Group" traditionnel, nommé
"franchini-Drupal-DB". La procédure n'est pas détaillée dans ce chapitre (cf.
[Laboratoire n°1, tâche 2](https://github.com/crabone/HEIGVD-CLD-2017-Labo-IaaS#t%C3%82che-2-cr%C3%89ation-dune-instance-amazon-ec2)).

![RDS creation step-by-step](assets/images/01-rds_creation_step_3.png)

Maintenant que le "Security Group" a été créé, nous allons créer une nouvelle
instance RDS. Pour ce faire il faut retourner dans le tableau de bord prévu à
cet effet (cf. plus haut), ce rendre dans le sous-menu "Instance" et cliquer
sur "Launch DB Instance".

![RDS creation step-by-step](assets/images/01-rds_creation_step_4.png)

Nous sommes invités à choisir le type de base de données à employer. Ici nous
souhaitons utiliser une instance basée sur `MySQL`.

![RDS creation step-by-step](assets/images/01-rds_creation_step_5.png)

Comme cette procédure est expérimentale, nous ne choisissons pas de mettre en
production cette instance et nous utilisons à la place l'offre "Dev/Test".

![RDS creation step-by-step](assets/images/01-rds_creation_step_6.png)

Ensuite, nous sommes invité à saisir les détails de l'instance, à savoir: Son
type, son identifiant et les crédentials liés à `MySQL`.

![RDS creation step-by-step](assets/images/01-rds_creation_step_7.png)

À cet instant on choisis son "Security Group" et désactivons les sauvegardes
automatiques.

![RDS creation step-by-step](assets/images/01-rds_creation_step_8.png)

Pour finir, nous confirmons la création de l'instance et prennons compte des
détails de configuration.

![RDS creation step-by-step](assets/images/01-rds_creation_step_9.png)

![RDS creation step-by-step](assets/images/01-rds_creation_step_10.png)

Nous apprenons que l'endpoint est identifié par le nom de domaine publique
`franchini-drupal.cm4237cqo8tv.eu-central-1.rds.amazonaws.com`.

Pour tester le bon déroulement de la manipulation nous tentons d'accèder à
cette nouvelle base de données, depuis l'instance contenant `drupal7` via le
client `mysql`.

```
ubuntu@ip-172-31-3-176:~$ mysql --host=franchini-drupal.cm4237cqo8tv.eu-central-1.rds.amazonaws.com --user=root --password=<SECRET>
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 19
Server version: 5.6.27 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> quit
Bye
```

Le client ne nous retourne pas d'erreur, tout est ok.

[[[[[[[[[[[[[[[Répondre aux questions posés]]]]]]]]]]]]]]]

## TÂCHE 2: CONFIGURATION DE DRUPAL POUR L'UTILISATION D'UNE BASE DE DONNÉE RDS

Dans ce chapitre, nous allons reconfigurer `drupal7` pour l'utilisation d'une
base de données distante et migrer son contenu dans cette dernière.

### CHANGEMENT DE LA CONFIGURATION DE DRUPAL

Avant toute manipulation, nous devons stopper le serveur `apache2` afin d'éviter
tout désagrément.

```
ubuntu@ip-172-31-3-176:~$ sudo /etc/init.d/apache2 stop
 * Stopping web server apache2  
 *
```

Pour reconfigurer `drupal7` nous utilisons l'utilitaire `dpkg-reconfigure`.
`dpkg` est un système de gestion de paquets de bas niveau, utilisé dans les
distributions dérivant de Debian. Par analogie, nous citons `rpm`, utilitaire
similaire pour les distributions dérivant de Red Hat.

```
ubuntu@ip-172-31-3-176:~$ sudo dpkg-reconfigure drupal7
```

À présent, nous sommes invité à reconfigurer `drupal7` à l'aide d'une interface
graphique minimale.

Premièrement, on nous demande si nous voulons réinstallé la base de donnée. Nous
confirmons ce choix, étant donné que c'est notre intention première.

![Drupal 7 database reconfiguration step-by-step](assets/images/02-drupal7_reconfigure_step_1.png)

Nous choisissons le type de base de données, à savoir `MySQL`.

![Drupal 7 database reconfiguration step-by-step](assets/images/02-drupal7_reconfigure_step_2.png)

Nous sommes invités à saisir la méthode d'accès à la base de données distante.
Nous choisissons d'utiliser TCP/IP comme méthode de communication.

![Drupal 7 database reconfiguration step-by-step](assets/images/02-drupal7_reconfigure_step_3.png)

Ici, nous sommes invités à saisir le nom d'hôte du serveur distant, à savoir
l'endpoint obtenu dans la tâche précédente.

![Drupal 7 database reconfiguration step-by-step](assets/images/02-drupal7_reconfigure_step_4.png)

Nous spécifions le port d'accès, à savoir le port 3306.

![Drupal 7 database reconfiguration step-by-step](assets/images/02-drupal7_reconfigure_step_5.png)

À cet instant, nous devons saisir les credentials de l'administrateur de `MySQL`
(cf. [Tâche 1]()).

![Drupal 7 database reconfiguration step-by-step](assets/images/02-drupal7_reconfigure_step_6.png)

![Drupal 7 database reconfiguration step-by-step](assets/images/02-drupal7_reconfigure_step_7.png)

Maintenant, nous créons un nouvel utilisateur qui va gérer les tables liés à
`drupal7`.

![Drupal 7 database reconfiguration step-by-step](assets/images/02-drupal7_reconfigure_step_8.png)

Pour terminer nous saisissons le nom de la base de données à créer pour
`drupal7`.

![Drupal 7 database reconfiguration step-by-step](assets/images/02-drupal7_reconfigure_step_9.png)

[[[[[[[[[[[[[Détaillé cette partie après avoir posé quelques questions au prof]]]]]]]]]]]]]

```
ubuntu@ip-172-31-8-194:~$ sudo cat /etc/drupal/7/sites/default/dbconfig.php
<?php
#
# database access settings in php format
# automatically generated from /etc/dbconfig-common/drupal7.conf
# by /usr/sbin/dbconfig-generate-include
#
# by default this file is managed via ucf, so you shouldn't have to
# worry about manual changes being silently discarded.  *however*,
# you'll probably also want to edit the configuration file mentioned
# above too.

$databases['default']['default'] = array(
	'driver' => 'mysql',
	'database' => 'drupal7',
	'username' => 'drupal7',
	'password' => '<SECRET_DRUPAL7>',
	'host' => 'franchini-drupal.cm4237cqo8tv.eu-central-1.rds.amazonaws.com',
	'port' => '',
	'prefix' => ''
);

?>
```

```
ubuntu@ip-172-31-3-176:~$ mysql --host=franchini-drupal.cm4237cqo8tv.eu-central-1.rds.amazonaws.com --user=root --password=<SECRET>
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 19
Server version: 5.6.27 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE USER 'drupal7'@'%' IDENTIFIED BY '<SECRET_DRUPAL7>';
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES ON drupal7.* TO 'drupal7'@'%' IDENTIFIED BY '<SECRET_DRUPAL7>';
Query OK, 0 rows affected (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> quit
Bye
```

### MIGRATION DE LA BASE DE DONNÉES

```
ubuntu@ip-172-31-3-176:~$ mysqldump --add-drop-table --user=drupal7 --password=<SECRET_DRUPAL7> drupal7 |
  mysql --host=franchini-drupal.cm4237cqo8tv.eu-central-1.rds.amazonaws.com --user=drupal7 --password=<SECRET_DRUPAL7> drupal7
```

## TÂCHE 3: CRÉATION D'UNE IMAGE VIRTUELLE PERSONNALISÉE

Dans ce chapitre, nous allons créer une image virtuelle personnalisée basée sur
l'instance contenant `drupal7`. Cette étape est nécessaire pour la redondance
de l'application. Ainsi nous pourrons déployer plusieurs instances de
l'application, dans différentes régions proposées par Amazon.

Premièrement, il faut se rendre dans le tableau de bord permettant la gestion
des instances EC2. Après avoir sélectionné l'instance contenant l'application
web, nous déroulons le menu "Actions" et sélectionnons l'option permettant de
créer une image à partir de l'instance.

![Custom AMI creation step-by-step](assets/images/03-image_creation_step_1.png)

Ensuite, nous sommes invités à saisir les détails de l'image. Nous saisissons
son nom et sa description tel demandé dans l'énoncé du laboratoire.

![Custom AMI creation step-by-step](assets/images/03-image_creation_step_2.png)

Nous reçevons la confirmation que le processus de création de l'image est lancé.
À partir de ça, nous devons attendre quelques minutes pour sa création.

![Custom AMI creation step-by-step](assets/images/03-image_creation_step_3.png)

Pour voir si l'image a bien été créée, nous nous rendons dans le tableau de bord
approprié. Pour ce faire il faut se rendre dans le sous-menu "AMIs" de la
catégorie "Images".

![Custom AMI creation step-by-step](assets/images/03-image_creation_step_4.png)

Nous avons la confirmation que l'image a bien été créée. Notons tout de même que
l'image est stocké dans la régions où elle a été créée.

## TÂCHE 4: CRÉATION D'UN LOAD BALANCER

Dans ce chapitre, nous allons créer un répartisseur de charge proposé par
Amazon (ELB). [[[[[Détail flemme]]]]]

![Load balancer creation step-by-step](assets/images/04-load_balancer_creation_step_1.png)

![Load balancer creation step-by-step](assets/images/04-load_balancer_creation_step_2.png)

Ici, nous spécifions le nom de notre load balancer, à savoir "franchini-Drupal".

![Load balancer creation step-by-step](assets/images/04-load_balancer_creation_step_3.png)

![Load balancer creation step-by-step](assets/images/04-load_balancer_creation_step_4.png)

![Load balancer creation step-by-step](assets/images/04-load_balancer_creation_step_5.png)

![Load balancer creation step-by-step](assets/images/04-load_balancer_creation_step_6.png)

![Load balancer creation step-by-step](assets/images/04-load_balancer_creation_step_7.png)

http://franchini-drupal-1520229052.eu-central-1.elb.amazonaws.com/drupal7/

```
172.31.25.19 - - [07/Mar/2017:22:36:34 +0000] "GET /drupal7/ HTTP/1.1" 200 9189 "-" "ELB-HealthChecker/1.0"
172.31.5.164 - - [07/Mar/2017:22:36:36 +0000] "GET /drupal7/ HTTP/1.1" 200 9189 "-" "ELB-HealthChecker/1.0"
172.31.25.19 - - [07/Mar/2017:22:36:43 +0000] "GET /drupal7/ HTTP/1.1" 200 9189 "-" "ELB-HealthChecker/1.0"
172.31.5.164 - - [07/Mar/2017:22:36:46 +0000] "GET /drupal7/ HTTP/1.1" 200 9189 "-" "ELB-HealthChecker/1.0"
172.31.25.19 - - [07/Mar/2017:22:36:53 +0000] "GET /drupal7/ HTTP/1.1" 200 9189 "-" "ELB-HealthChecker/1.0"
172.31.5.164 - - [07/Mar/2017:22:36:56 +0000] "GET /drupal7/ HTTP/1.1" 200 9189 "-" "ELB-HealthChecker/1.0"
172.31.25.19 - - [07/Mar/2017:22:37:03 +0000] "GET /drupal7/ HTTP/1.1" 200 9189 "-" "ELB-HealthChecker/1.0"
172.31.5.164 - - [07/Mar/2017:22:37:06 +0000] "GET /drupal7/ HTTP/1.1" 200 9189 "-" "ELB-HealthChecker/1.0"
172.31.25.19 - - [07/Mar/2017:22:37:13 +0000] "GET /drupal7/ HTTP/1.1" 200 9189 "-" "ELB-HealthChecker/1.0"
172.31.5.164 - - [07/Mar/2017:22:37:16 +0000] "GET /drupal7/ HTTP/1.1" 200 9189 "-" "ELB-HealthChecker/1.0"
172.31.25.19 - - [07/Mar/2017:22:37:23 +0000] "GET /drupal7/ HTTP/1.1" 200 9189 "-" "ELB-HealthChecker/1.0"
```

## TÂCHE 5: LANCEMENT D'UNE SECONDE INSTANCE DEPUIS LA NOUVELLE IMAGE

Dans ce chapitre, nous allons générer une seconde instance de l'application,
dans une zone différente, à partir de l'image créée précédement.

Premièrement nous constatons que nous pouvons pas, de base, générer une instance
EC2 dans une région différente de celle où est situé notre AMI. Nous devons, de
ce fait, copier l'image dans la région où nous souhaitons générer notre deuxième
instance.

![AMI copy step-by-step](assets/images/05-ami_move_step_0.png)

Pour ce faire, il faut retourner dans le tableau de bord des AMIs, dans leur
région d'origine. Ensuite, après avoir sélectionné l'image, dans le menu
déroulant "Actions", il faut cliquer sur "Copy AMI".

![AMI copy step-by-step](assets/images/05-ami_move_step_1.png)

Alors s'ouvre une fenêtre pop-up nous demandant de spécifier la région de
destination et ses caractéristiques. Nous choisisons de copier l'image dans
la région de Tokyo.

![AMI copy step-by-step](assets/images/05-ami_move_step_2.png)

À présent, nous devons attendre quelques minutes pour ensuite constater la
présence de la nouvelle image dans la région désirée.

![AMI copy step-by-step](assets/images/05-ami_move_step_3.png)

![AMI copy step-by-step](assets/images/05-ami_move_step_4.png)

![Second instance creation step-by-step](assets/images/05-create_second_ec2_step_.png)

![Second instance creation step-by-step](assets/images/05-create_second_ec2_step_.png)

![Second instance creation step-by-step](assets/images/05-create_second_ec2_step_.png)

![Second instance creation step-by-step](assets/images/05-create_second_ec2_step_.png)

![Second instance creation step-by-step](assets/images/05-create_second_ec2_step_.png)

## TÂCHE 6: TEST DE L'APPLICATION DISTRIBUÉE

## TÂCHE 7: LIBÉRATION DES RESSOURCES

## CONCLUSION
