CREATE DATABASE foody;
USE foody;

CREATE TABLE client (
CodeCli VARCHAR(5) NOT NULL,
Societe VARCHAR(45) NOT NULL,
Contact VARCHAR(45) NOT NULL,
Fonction VARCHAR(45),
Adresse VARCHAR(45),
Ville VARCHAR(25),
Region VARCHAR(25),
CodePostal VARCHAR(10),
Pays VARCHAR(25),
Tel VARCHAR(25),
Fax VARCHAR(25),
PRIMARY KEY (CodeCli)
)
ENGINE=INNODB;

CREATE TABLE commande (
NoCom INT NOT NULL AUTO_INCREMENT,
CodeCli VARCHAR(10),
NoEmp INT,
DateCom DATETIME,
ALivAvant DATETIME,
DateEnv DATETIME,
NoMess INT,
Port DECIMAL(10,4) DEFAULT 0,
Destinataire VARCHAR(50),
AdrLiv VARCHAR(50),
VilleLiv VARCHAR(50),
RegionLiv VARCHAR(50),
CodepostalLiv VARCHAR(20),
PaysLiv VARCHAR(25),
PRIMARY KEY (NoCom)
)
ENGINE=INNODB;

CREATE TABLE messager (
NoMess INT NOT NULL AUTO_INCREMENT,
NomMess VARCHAR(50) NOT NULL,
Tel VARCHAR(20),
PRIMARY KEY (NoMess)
)
ENGINE=INNODB;

CREATE TABLE detailcommande (
NoCom INT NOT NULL,
Refprod INT NOT NULL,
PrixUnit DECIMAL(10,4) NOT NULL DEFAULT 0,
Qte INT NOT NULL DEFAULT 1,
Remise Double NOT NULL DEFAULT 0,
PRIMARY KEY (NoCom, RefProd)
)
ENGINE=INNODB;

CREATE TABLE employe (
NoEmp INT NOT NULL AUTO_INCREMENT,
Nom VARCHAR(50) NOT NULL,
Prenom VARCHAR(50) NOT NULL,
Fonction VARCHAR(50),
TitreCourtoisie VARCHAR(50),
DateNaissance DATETIME,
DateEmbauche DATETIME,
Adresse VARCHAR(60),
Ville VARCHAR(50),
Region VARCHAR(50),
CodePostal VARCHAR(50),
Pays VARCHAR(50),
TelDom VARCHAR(20),
Extension VARCHAR(50),
RendCompteA INT,
PRIMARY KEY (NoEmp)
)
ENGINE=INNODB;

CREATE TABLE produit (
RefProd INT NOT NULL AUTO_INCREMENT,
NomProd VARCHAR(50) NOT NULL,
NoFour INT,
CodeCateg INT,
QteParUnit VARCHAR(20),
PrixUnit DECIMAL(10,4) DEFAULT 0,
UnitesStock SMALLINT DEFAULT 0,
UnitesCom SMALLINT DEFAULT 0,
NiveauReap SMALLINT DEFAULT 0,
Indisponible BIT NOT NULL DEFAULT 0,
PRIMARY KEY (RefProd)
)
ENGINE=INNODB;

CREATE TABLE categorie (
CodeCateg INT NOT NULL AUTO_INCREMENT,
NomCateg VARCHAR(15) NOT NULL,
Description VARCHAR(255),
PRIMARY KEY (CodeCateg)
)
ENGINE=INNODB;

CREATE TABLE fournisseur (
NoFour INT NOT NULL AUTO_INCREMENT,
Societe VARCHAR(45) NOT NULL,
Contact VARCHAR(45),
Fonction VARCHAR(45),
Adresse VARCHAR(255),
Ville VARCHAR(45),
Region VARCHAR(45),
CodePostal VARCHAR(10),
Pays VARCHAR(45),
Tel VARCHAR(20),
Fax VARCHAR(20),
PageAccueil MEDIUMTEXT,
PRIMARY KEY (NoFour)
)
ENGINE=INNODB;

ALTER TABLE commande ADD FOREIGN KEY (CodeCli) REFERENCES client(CodeCli);
ALTER TABLE commande ADD FOREIGN KEY (NoMess) REFERENCES messager(NoMess);
ALTER TABLE commande ADD FOREIGN KEY (NoEmp) REFERENCES employe(NoEmp);
ALTER TABLE detailcommande ADD FOREIGN KEY (NoCom) REFERENCES commande(NoCom);
ALTER TABLE detailcommande ADD FOREIGN KEY (RefProd) REFERENCES produit(RefProd);
ALTER TABLE employe ADD FOREIGN KEY (RendCompteA) REFERENCES employe(NoEmp);
ALTER TABLE produit ADD FOREIGN KEY (NoFour) REFERENCES fournisseur(NoFour);
ALTER TABLE produit ADD FOREIGN KEY (CodeCateg) REFERENCES categorie(CodeCateg);

-----------------------
I - Requêtage simple :

I.1 - Requêtage simple

Exercices :

#1. Afficher les 10 premiers éléments de la table Produit triés par leur prix unitaire 
SELECT * FROM produit ORDER BY PrixUnit ASC LIMIT 10

#2. Afficher les trois produits les plus chers
SELECT * FROM produit ORDER BY PrixUnit DESC LIMIT 3

I.2 - Restriction

Exercices :

#1. Lister les clients français installés à Paris dont le numéro de fax n'est pas renseigné
SELECT * FROM client WHERE Ville = 'Paris' AND FAX IS NULL

#2. Lister les clients français, allemands et canadiens
SELECT * FROM client WHERE Pays = 'France' OR Pays = 'Canada' OR Pays = 'Germany'

#3. Lister les clients dont le nom de société contient "restaurant"
SELECT * FROM client WHERE Societe LIKE '%restaurant%'
//SELECT * FROM client wHERE INSTR(Societe, 'restaurant')!=0

I.3 - Projection

Exercices :

#1. Lister les descriptions des catégories de produits (table Categorie)
SELECT DISTINCT Description, NomCateg FROM categorie

#2. Lister les différents pays et villes des clients, le tout trié par ordre alphabétique croissant du pays et décroissant de la ville
SELECT DISTINCT Pays, Ville FROM client ORDER BY Pays ASC, Ville DESC
//SELECT DISTINCT Pays, Ville FROM client ORDER BY 1, 2

#3. Lister les fournisseurs français, en affichant uniquement le nom, le contact et la ville, triés par ville
SELECT DISTINCT Societe, Contact, Ville FROM `fournisseur` WHERE Pays = 'France' ORDER BY Ville

#4. Lister les produits (nom en majuscule et référence) du fournisseur n° 8 dont le prix unitaire est entre 10 et 100 euros, en renommant les attributs pour que ça soit explicite
SELECT UPPER(NomProd) AS nomDuProduit, RefProd AS referenceDuProduit FROM produit WHERE NoFour = 8 AND PrixUnit BETWEEN 10 AND 100

II - Calculs et Fonctions :

II.1 - Calculs arithmétiques

Exercices :

La table DetailsCommande contient l'ensemble des lignes d'achat de chaque commande. Calculer, pour la commande numéro 10251, pour chaque produit acheté dans celle-ci,
le montant de la ligne d'achat en incluant la remise (stockée en proportion dans la table). Afficher donc (dans une même requête) :
SELECT PrixUnit, Qte, Remise, (PrixUnit * Remise) AS 'MontantRemise', PrixUnit * Qte - (PrixUnit * Remise) * Qte AS 'MontantPaye' FROM detailcommande WHERE NoCom = 10251

II.2 - Traitement conditionnel

A partir de la table Produit, afficher "Produit non disponible" lorsque l'attribut Indisponible vaut 1, et "Produit disponible" sinon.
SELECT CASE WHEN Indisponible = 1 THEN 'Produit non disponible' ELSE 'Produit disponible' END AS 'Disponibilite' FROM produit

II.3 - Fonctions sur chaînes de caractères

Exercices :

#Concaténer les champs Adresse, Ville, CodePostal et Pays dans un nouveau champ nommé Adresse_complète, pour avoir : Adresse, CodePostal, Ville, Pays
SELECT CONCAT (Adresse, ", ", CodePostal, ", ", Ville, ", ", Pays) AS 'Adresse_complete' FROM client
#Extraire les deux derniers caractères des codes clients
SELECT SUBSTR(CodeCli, -2) FROM client
//SELECT SUBSTR(CodeCli, 4, 2) FROM client
#Mettre en minuscule le nom des sociétés
SELECT LOWER(Societe) AS nomSociete FROM client
#Remplacer le terme "Owner" par "Freelance" dans Fonction
SELECT REPLACE(Fonction, 'Owner', 'Freelance') FROM client
#Indiquer la présence du terme "Manager" dans Fonction
SELECT * FROM client WHERE Fonction LIKE '%manager%'

Requete complete = 

SELECT Fonction,
REPLACE(Fonction, 'Owner', 'Freelance') AS Fonctions,
LOWER(Societe) AS Societe, 

SUBSTR(Codecli,4,5) AS CodeCli ,
SUBSTR(Codecli,-2)
RIGHT(Codecli,2)
SUBSTR(Codecli,length(Codecli)-1,2)

CONCAT(Adresse ," ",Ville," ",Codepostal," ",Pays) AS Adresse_Complete
FROM `client` WHERE  `fonction` LIKE '%Manager%'

II.4 - Fonctions sur les dates

Exercices :

#1. Afficher le jour de la semaine en lettre pour toutes les dates de commande, afficher "week-end" pour les samedi et dimanche.
SELECT CASE 
      WHEN DATE_FORMAT(DateCom,"%W")='Saturday' THEN 'week-end' 
      WHEN DATE_FORMAT(DateCom,"%W")='Sunday' THEN 'week-end' 
      ELSE DATE_FORMAT(DateCom,"%W") END 
      FROM commande

#2. Calculer le nombre de jours entre la date de la commande (DateCom) et la date butoir de livraison (ALivAvant), pour chaque commande, 
On souhaite aussi contacter les clients 1 mois après leur commande. ajouter la date correspondante pour chaque commande
SELECT *, DATEDIFF(ALivAvant,DateCom), DATE_ADD(DateCom, INTERVAL 30 DAY) FROM commande

III - Aggrégats :

III.1 - Dénombrements

Exercices :

#1. Calculer le nombre d'employés qui sont "Sales Manager"
SELECT COUNT(*) FROM employe WHERE Fonction ='Sales Manager'

#2. Calculer le nombre de produits de catégorie 1, des fournisseurs 1 et 18
SELECT COUNT(*) FROM produit WHERE CodeCateg = 1 AND NoFour IN (1, 18)

#3. Calculer le nombre de pays différents de livraison
SELECT COUNT(DISTINCT PaysLiv) FROM commande

#4. Calculer le nombre de commandes réalisées en Aout 2006.
SELECT COUNT(*) FROM commande WHERE DateCom LIKE '2006-08%'

III.2 - Calculs statistiques simples

Exercices :

#1. Calculer le coût du port minimum et maximum des commandes , ainsi que le coût
moyen du port pour les commandes du client dont le code est "QUICK"(attribut CodeCli)
SELECT MIN(Port) AS MinPort, MAX(Port) AS MaxPort, AVG(Port) AS AvgPort FROM commande WHERE CodeCli = 'QUICK'

#2. Pour chaque messager (par leur numéro : 1, 2 et 3), donner le montant total des frais de port leur correspondant
SELECT NoMess, SUM(Port) FROM commande GROUP BY NoMess

III.3 - Agrégats selon attribut(s)

Exercices :

#1. Donner le nombre d'employés par fonction
SELECT Fonction, COUNT(*) AS NbrEmploye FROM employe GROUP BY Fonction

#2. Donner le nombre de catégories de produits fournis par chaque fournisseur
SELECT NoFour, COUNT(DISTINCT CodeCateg) AS CatFournisseur FROM produit GROUP BY NoFour

#3. Donner le prix moyen des produits pour chaque fournisseur et chaque catégorie de produits fournis par celui-ci
SELECT NoFour, CodeCateg, AVG(PrixUnit) AS PrixMoyen FROM produit GROUP BY NoFour, CodeCateg

III.4 - Restriction sur agrégats

Exercices :

#1. Lister les fournisseurs ne fournissant qu'un seul produit
SELECT NoFour, COUNT(RefProd) FROM produit GROUP BY NoFour HAVING COUNT(RefProd) = 1

#2.Lister les fournisseurs ne fournissant qu'une seule catégorie de produits
SELECT COUNT(DISTINCT CodeCateg) AS Nb, CodeCateg, NoFour FROM produit GROUP BY NoFour HAVING Nb = 1

#3. Lister le Products le plus cher pour chaque fournisseur, pour les Products de plus de 50 euro
SELECT NoFour, PrixUnit, MAX(PrixUnit) FROM produit GROUP BY NoFour HAVING MAX(PrixUnit) > 50

IV - Jointures :

IV.1 - Jointures naturelles

Exercices :

#1. Récupérer les informations des fournisseurs pour chaque produit
SELECT * FROM produit NATURAL JOIN fournisseur

#2. Afficher les informations des commandes du client "Lazy K Kountry Store"
SELECT * FROM client NATURAL JOIN commande WHERE Societe = "Lazy K Kountry Strore"

#3. Afficher le nombre de commande pour chaque messager (en indiquant son nom)
SELECT NomMess, COUNT(*) AS "Nombre Commandes" FROM Messager NATURAL JOIN Commande GROUP BY NomMess;

IV.2 - Jointures internes

Exercices :

#1. Récupérer les informations des fournisseurs pour chaque produit, avec une jointure interne
SELECT * FROM produit INNER JOIN fournisseur ON produit.NoFour = fournisseur.NoFour

#2. Afficher les informations des commandes du client "Lazy K Kountry Store", avec une jointure interne
SELECT * FROM commande INNER JOIN client ON commande.CodeCli = client.Codecli WHERE Societe = "Lazy K Kountry Store"

#3. Afficher le nombre de commande pour chaque messager (en indiquant son nom), avec une jointure interne
SELECT NomMess, COUNT(*) FROM commande INNER JOIN messager ON commande.NoMess = messager.NoMess GROUP BY NomMess

IV.3 - Jointures externes

Exercices :

#1. Compter pour chaque produit, le nombre de commandes où il apparaît, même pour ceux dans aucune commande
SELECT NomProd, COUNT(DISTINCT NoCom) FROM produit LEFT OUTER JOIN detailcommande ON produit.RefProd = detailcommande.RefProd GROUP BY NomProd

#2. Lister les produits n'apparaissant dans aucune commande
SELECT NomProd FROM produit LEFT OUTER JOIN detailcommande ON produit.RefProd = detailcommande.RefProd GROUP BY NomProd HAVING COUNT(DISTINCT NoCom) = 0;

#3. Existe-t'il un employé n'ayant enregistré aucune commande ?
SELECT Nom, Prenom FROM employe LEFT OUTER JOIN commande ON employe.NoEmp = commande.NoEmp GROUP BY Nom, Prenom HAVING COUNT(DISTINCT NoCom) = 0;

IV.4 - Jointures à la main

Exercices :

#1. Récupérer les informations des fournisseurs pour chaque produit, avec jointure à la main
SELECT fournisseur.NoFour, fournisseur.Societe, produit.NomProd FROM fournisseur, produit WHERE fournisseur.NoFour = produit.NoFour

#2. Afficher les informations des commandes du client "Lazy K Kountry Store", avec jointure à la main
SELECT * FROM commande, client WHERE commande.CodeCli = client.Codecli AND Societe = "Lazy K Kountry Store"

#3. Afficher le nombre de commande pour chaque messager (en indiquant son nom), avec jointure à la main
SELECT NomMess, COUNT(*) FROM commande, messager WHERE commande.NoMess = messager.NoMess GROUP BY NomMess

V - Sous-requêtes :

V.1 - Sous-requêtes

Exercices :

#1. Lister les employés n'ayant jamais effectué une commande, via une sous-requête
SELECT Nom, Prenom FROM employe WHERE NoEmp NOT IN (SELECT NoEmp FROM commande)

#2. Nombre de produits proposés par la société fournisseur "Ma Maison", via une sous-requête
SELECT COUNT(*) FROM produit WHERE NoFour = (SELECT NoFour FROM fournisseur WHERE Societe = "Ma Maison")
//SELECT COUNT(*) FROM produit NATURAL JOIN (SELECT NoFour FROM fournisseur WHERE Societe = "Ma Maison")

#3. Nombre de commandes passées par des employés sous la responsabilité de "Buchanan Steven"
SELECT COUNT(commande.NoCom) AS NbCom
    FROM commande
    WHERE commande.NoEmp 
    IN 
    (SELECT employe.NoEmp 
    FROM employe WHERE employe.RendCompteA 
    = 
    (SELECT employe.NoEmp 
    FROM employe 
    WHERE employe.Nom = "Buchanan" 
    AND employe.Prenom = "Steven"))

V.2 - Opérateur EXISTS

Exercices :

#1. Lister les produits n'ayant jamais été commandés, à l'aide de l'opérateur EXISTS
SELECT produit.RefProd, produit.NomProd
    FROM produit
    WHERE NOT EXISTS (SELECT detailcommande.RefProd FROM detailscommande WHERE produit.RefProd = detailcommande.RefProd)

#2. Lister les fournisseurs dont au moins un produit a été livré en France
SELECT fournisseur.Societe
        FROM fournisseur
        WHERE EXISTS (SELECT * 
                    FROM produit, detailcommande, commande
                    WHERE produit.RefProd = detailcommande.RefProd
                    AND detailcommande.NoCom = commande.NoCom
                    AND PaysLiv = "France"
                    AND NoFour = fournisseur.NoFour)

#3. Liste des fournisseurs qui ne proposent que des boissons (drinks)
SELECT fournisseur.Societe
    FROM fournisseur
    WHERE EXISTS 
        (SELECT * FROM produit, categorie WHERE produit.NoFour = fournisseur.NoFour AND produit.CodeCateg = categorie.CodeCateg AND categorie.NomCateg = "drinks")
    AND NOT EXISTS 
        (SELECT * FROM produit, categorie WHERE produit.NoFour = fournisseur.NoFour AND produit.CodeCateg = categorie.CodeCateg AND categorie.NomCateg <> "drinks")

VI - Opérations Ensemblistes :

VI.1 - Union

Exercices :

#1. Lister les employés (nom et prénom) étant "Representative" ou étant basé au Royaume-Uni (UK)
SELECT Nom, Prenom
    FROM employe
    WHERE Fonction = "Representantive"
UNION  
SELECT Nom, Prenom
    FROM employe
    WHERE Pays = "UK"

#2. Lister les clients (société et pays) ayant commandés via un employé situé à Londres ("London" pour rappel) ou ayant été livré par "Speedy Express"
SELECT Societe, cl.Pays
    FROM client cl, commande co, employe e
    WHERE cl.CodeCli = co.CodeCli
    AND co.NoEmp = e.NoEmp
    AND e.Ville = "London"
UNION
SELECT Societe, cl.Pays
    FROM client cl, commande co, messager m
    WHERE cl.CodeCli = co.CodeCli
    AND co.NoMess = m.NoMess
    AND NomMess = "Speedy Express"

VI.2 - Intersection

Exercices :

#1. Lister les employés (nom et prénom) étant "Representative" et étant basé au Royaume-Uni (UK)
SELECT Nom, Prenom
    FROM employe
    WHERE Fonction = "Representantive"
INTERSECT
SELECT Nom, Prenom
    FROM employe
    WHERE Pays = "UK"

#2. Lister les clients (société et pays) ayant commandés via un employé basé à "Seattle" et ayant commandé des "Desserts"
SELECT Societe, cl.Pays
    FROM client Cl, commande co, Employe e
    WHERE cl.CodeCli = co.CodeCli
    AND co.NoEmp = e.NoEmp
    AND e.Ville = "Seattle"
INTERSECT
SELECT Societe, cl.Pays
    FROM client cl, commande co, detailcommande dc,
         produit p, categorie ca
    WHERE cl.CodeCli = co.CodeCli
    AND co.NoCom = dc.NoCom
    AND dc.RefProd = p.RefProd
    AND p.CodeCateg = ca.CodeCateg
    AND NomCateg = "Desserts"

VI.3 - Différence

Exercices :

#1. Lister les employés (nom et prénom) étant "Representative" mais n'étant pas basé au Royaume-Uni (UK)
SELECT Nom, Prenom
    FROM employe
    WHERE Fonction = "Representantive
EXCEPT
SELECT Nom, Prenom
    FROM employe
    WHERE Pays = "UK"

#2. Lister les clients (société et pays) ayant commandés via un employé situé à Londres ("London" pour rappel) et n'ayant jamais été livré par "United Package"
SELECT Societe, Cl.Pays
    FROM Client Cl, Commande Co, Employe E
    WHERE Cl.CodeCli = Co.CodeCli
    AND Co.NoEmp = E.NoEmp
    AND E.Ville = "London"
EXCEPT
SELECT Societe, cl.Pays
    FROM client cl, commande co, messager m
    WHERE cl.CodeCli = co.CodeCli
    AND co.NoMess = m.NoMess
    AND NomMess = "United Package"
