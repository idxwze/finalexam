# 2024-2025 - SEG2505 - Examen final - Préparation - 1 - Correction

## Hypothèses

- Les marchandises circulent en caisse lorsqu'elles sont transportées (à l'arrivée ou au départ du port) par voie terrestre.
- Les caisses sont mises en container dans l'aire de stockage des caisses de marchandises qui arrivent par voie terrestre.
- Les containers qui arrivent par voie maritime sont déchargés et vidés dans l'aire de stockage des caisses de marchandises qui arrivent par voie maritime.

## Diagramme de cas d'utilisation

Le système doit traiter les principaux cas d'utilisation suivants :

- pour le responsable d'aire de stockage :
  - Décharger les caisses depuis les trains ou les camions
  - Charger les caisses vers les trains ou les camions
  - Décharger les caisses depuis les containers
  - Charger les caisses vers les containers

- pour les contrôleurs (physiques, administratif et financiers) :
  - Effectuer un contrôle physique
  - Effectuer un contrôle administratif
  - Effectuer un contrôle financier
  - Exécuter la procédure de rejet pour non-conformité

- pour les dockers :
  -  Charger les containers sur un bâteau
  -  Décharger les containers d'un bâteau

```plantuml
@startuml
    actor "Contrôleur" as C
    actor "Contrôleur physique" as CP
    actor "Contrôleur administratif" as CA
    actor "Contrôleur financier" as CF

    C <|-- CP
    C <|-- CA
    C <|-- CF 

    usecase "Effectuer un contrôle physique" as ECP
    usecase "Effectuer un contrôle administratif" as ECA
    usecase "Effectuer un contrôle financier" as ECF
    usecase "Exécuter la procédure de rejet pour non-conformité" as EPRNC

    CP --> ECP
    CA --> ECA
    CF --> ECF
    C --> EPRNC

    actor "Responsable d'aire de stockage" as RAS
    actor "Responsable d'aire de stockage terrestre" as RAST
    actor "Responsable d'aire de stockage maritime" as RASM

    RAS <|-- RAST
    RAS <|-- RASM

    usecase "Décharger les caisses depuis les trains ou les camions" as DCTC
    usecase "Charger les caisses vers les containers" as CCC
    usecase "Décharger les caisses depuis les containers" as DCC
    usecase "Charger les caisses vers les trains ou les camions" as CTC

    RAST --> DCTC
    RAST --> CCC

    RASM --> DCC
    RASM --> CTC

    actor "Docker" as D

    usecase "Charger les containers sur un bâteau" as CCB
    usecase "Décharger les containers d'un bâteau" as DCB

    D --> CCB
    D --> DCB
@enduml
```

Le diagramme de cas d'utilisation suivant détaille le cas d'utilisation "Accueillir une marchandise arrivant par voie terrestre" (il s'agit en fait d'un sous-ensemble du diagramme précédent, avec utilisation des liens extensions et des inclusions de cas):

```plantuml
@startuml
    actor "Contrôleur" as C
    actor "Contrôleur physique" as CP
    actor "Contrôleur administratif" as CA
    actor "Contrôleur financier" as CF

    C <|-- CP
    C <|-- CA
    C <|-- CF 

    usecase "Effectuer un contrôle physique" as ECP
    usecase "Effectuer un contrôle administratif" as ECA
    usecase "Effectuer un contrôle financier" as ECF
    usecase "Exécuter la procédure de rejet pour non-conformité" as EPRNC

    CP --> ECP
    CA --> ECA
    CF --> ECF

    C --> EPRNC
    
    actor "Responsable d'aire de stockage terrestre" as RAST
    
    usecase "Décharger les caisses depuis les trains ou les camions" as DCTC
    
    RAST --> DCTC

    usecase "Accueillir une marchandise arrivant par voie terrestre" as AMAVT

    AMAVT ..> DCTC : includes
    AMAVT ..> ECP : includes
    AMAVT ..> ECA : includes
    AMAVT ..> ECF : includes
    EPRNC ..> ECP : extends
    EPRNC ..> ECA : extends
    EPRNC ..> ECF : extends
@enduml
```

## Diagramme de classes

Les principaux concepts utilisés par le système sont les suivants :

```plantuml
@startuml
package "OpenPort" #DDDDDD {
    class Marchandise {
    }

    Marchandise "*" -- "*" MoyenDeTransport : estTransportéPar >

    abstract class MoyenDeTransport {
        identifiantUnique: long
        nomCompagniePropriétaire: String
        capacitéMaximale: integer
    }

    note left of MoyenDeTransport::capacitéMaximale
        Caisses ou containers.
    end note 

    note right of MoyenDeTransport::capacitéMaximale
        context MoyenDeTransport
        inv: capacitéMaximale>0
    end note

    abstract class MoyenDeTransportTerrestre extends MoyenDeTransport {
    }

    abstract class MoyenDeTransportMaritime extends MoyenDeTransport {
        nomUnique: String
    }

    class Train extends MoyenDeTransportTerrestre {
    }

    class Camion extends MoyenDeTransportTerrestre {
    }

    class Bâteau extends MoyenDeTransportMaritime{
    }

    MoyenDeTransportTerrestre o-- Caisse: transporte >

    Caisse "1" o-- "*" Marchandise : contient >

    class Caisse {
        identifiantUnique: long
        typeDeContenu: TypeDeContenuDeCaisse 
    }

    Caisse "*" -- "1" Client: envoyéePar >
    Caisse "*" -- "1" Client: envoyéeA >

    class Client {
        nomUnique: String
    }

    Client "*" -- "1" AdressePostale: résideA >

    class AdressePostale {
        numéroDeRue: int
        nomDeRue: String
        ville: String
        codePostal: String
    }

    AdressePostale "*" -- "1" Pays : localiséeDans >

    class Pays {
        nomUnique: String
    
    enum TypeDeContenuDeCaisse {
        MATIERE_PREMIERE
        BIEN_DE_CONSOMMATION
        BIEN_INDUSTRIEL
        BIEN_PERSONNEL
        AUTRE
    }
    
    MoyenDeTransportMaritime o-- Container : transporte >
    Container "1" o-- "*" Caisse : contient >
    
    class Container {
        identifiantUnique: long
    }

    Container "*" -- "1" Port : provientDe >
    Container "*" -- "1" Port : destinéA >

    class Port {
        nomUnique: String
    }

    Port "*" -- "1" Pays : localiséDans >

    EspaceDeStockage "0..1" o-- "*" Caisse : contient >

    abstract class EspaceDeStockage {
        capacitéMaximale: int

        obtenirCapacitéDisponible(): int
    }

    note left of EspaceDeStockage::capacitéMaximale
        Caisses.
    end note

    note right of EspaceDeStockage::capacitéMaximale
        context EspaceDeStockage
        inv: capacitéMaximale>0
        inv: Caisse.size()<=capacitéMaximale
    end note

    class EspaceDeStockageTerrestre extends EspaceDeStockage {
    }

    class EspaceDeStockageMaritime extends EspaceDeStockage {
    }

    Bâteau "*" -- "2.. <<ordered>>" : dessert >

    Employé "*" -- Rôle: a >

    class Employé {
        nomUnique: String
    }

    abstract class Rôle {
        nomUnique: String
    }

    class ResponsableAireStockage extends Rôle {
    }

    class Contrôleur extends Rôle {
    }

    class Docker extends Rôle {
    }

    Employé "1..*" -- "1..*" Equipe : faitPartieDe >

    class Equipe {
    }

    Equipe "*" -- "*" QuartDeTravail : travaillePendant >

    class QuartDeTravail {
        début: DateAndTime
        fin: DateAndTime
    }
@enduml
```

Remarques :

- L'utilisation d'une classe "Marchandise" n'est pas franchement utile car le système de gestion du port ne manipule que des caisses et des containers. Ceci dit, la spécification fait explicitement référence à la notion de marchandise, donc il est naturel de l'inclure à ce niveau d'avancement du projet (ou en tout cas : en première lecture de la spécification). Plus tard, lorsque le projet aura mûri, cette classe "Marchandise" pourra disparaître pour simplifier le modèle, ceci sans perdre d'information essentielle.
- Les classes "Train" et "Camion" sont vides ; leur présence n'est maintenue que  parce la spécification y fait explicitement référence et on pense que, dans la suite du projet, on aura besoin de les spécialiser en leur associant des attributs et des comportements spécifiques.
- L'ajout de la classe "Client" n'est pas franchement requise : elle permet uniquement de distinguer des clients qui partagent la même adresse postale. Attention : cela pourrait être considéré comme de la "sur-spécification" parce qu'on suppose l'existence de quelque chose qu'on inclut dans le modèle alors que ce n'est pas absolument nécessaire pour la compréhension et le fonctionnement du système.
- Il serait possible d'ajouter des contraintes OCL pour exprimer que :
  - toutes les caisses contenues dans un container doivent avoir le même pays de destination que le container ;
  - chaque container chargé sur un bâteau doit avoir un port de destination qui est sur le trajet suivi par un bâteau ... à condition de prendre pour hypothèses que les containers ne peuvent pas être transférées d'une ligne maritime à l'autre dans un port.

## Diagrammes d'états

Le diagramme suivant décrit les états qu'une instance de marchandise arrivant par voie terrestre peut prendre, depuis son arrivée aux portes du port, jusqu’à son départ sur un bâteau.

```plantuml
@startuml
    [*] -->  EnCoursDeContrôlePhysique : entréeDansLePortDemandée 

    EnCoursDeContrôlePhysique --> EnCoursDeContrôleAdministratif : contrôlePhysiqueTerminé
    EnCoursDeContrôlePhysique --> [*] : rejetéPourNonConformité

    EnCoursDeContrôleAdministratif --> EnCoursDeContrôleFinancier : contrôleAdministratifTerminé
    EnCoursDeContrôleAdministratif --> [*] : rejetéPourNonConformité

    EnCoursDeContrôleFinancier --> EnAttenteDeDisponibilitéDeStockage : contrôleFinancierTerminé
    EnCoursDeContrôleFinancier --> [*] : rejetéPourNonConformité

    EnAttenteDeDisponibilitéDeStockage --> EnCoursDeDéchargementDesCaisses : obtenirCapacitéDisponible()>0

    EnCoursDeDéchargementDesCaisses --> EnAttenteDeChargementSurLeBâteau : déchargementDesCaissesTerminé

    EnAttenteDeChargementSurLeBâteau --> EnCoursDeMiseEnContainer : miseEnContainerDemandée

    EnCoursDeMiseEnContainer --> EnCoursDeChargementDesContainersSurLeBâteau : miseEnContainerTerminée

    EnCoursDeChargementDesContainersSurLeBâteau --> [*] : chargementSurLeBâteauTerminé
@enduml
```

## Diagrammes de séquences

Le diagramme suivant illustre la séquence d'actions impliquées dans l’accueil d’une marchandise arrivant par voie terrestre, depuis son arrivée aux portes du port, jusqu’à son départ sur un bâteau.

```plantuml
@startuml
    actor "Contrôleur physique" as CP
    actor "Contrôleur administratif" as CA
    actor "Contrôleur financier" as CF

    actor "Responsable d'aire de stockage terrestre" as RAST

    MoyenDeTransportTerrestre -> CP : demandeAutorisationEntrer(Contenu, List<DocumentAdministratif>, List<DocumentFinancier>)

    group "Réception de marchandises terrestres"
        group "Contrôles"
            CP -> CP : contrôle(MoyenDeTransportTerrestre)
            
            alt Contrôle physique conforme
                CP -> CA : nouveauMoyenDeTransportTerrestreContrôlé
                CA -> CA : contrôle(MoyenDeTransportTerrestre)
                
                alt Contrôle administratif conforme
                    CA -> CF : nouveauMoyenDeTransportTerrestreContrôlé
                    CF -> CF : contrôle(MoyenDeTransportTerrestre)
                    
                    alt Contrôle administratif conforme
                        CF -> RAST : demandeDeDéchargementDe((MoyenDeTransportTerrestre)
                    else Non-conformité détectée
                        CF -> CF : rejeterPourNonConformité()
                        CF -> MoyenDeTransportTerrestre : rejetéPourNonConformité
                    end
                else Non-conformité détectée
                    CA -> CA : rejeterPourNonConformité()
                    CA -> MoyenDeTransportTerrestre : rejetéPourNonConformité
                end
            else Non-conformité détectée
                CP -> CP : rejeterPourNonConformité()
                CP -> MoyenDeTransportTerrestre : rejetéPourNonConformité
            end
        end
        
        RAST -> RAST : déchargementDe((MoyenDeTransportTerrestre)
    end

    group "Chargement d'un bâteau"
        Bâteau -> RAST : demandeDeChargementDe(Bâteau)
        RAST --> Bâteau : chargementPlanifié

        RAST -> Docker : demandeDeChargementDe(Bâteau,List<Caisse>)
        Docker -> Docker : miseEnContainer(List<Caisse>)
        Docker --> RAST : miseEnContainerTerminée
        Docker -> Docker : chargementDe(Bâteau, List<Container>)
        Docker --> RAST : chargementSurLeBâteauTerminé
        Docker --> Bâteau : chargementSurLeBâteauTerminé
    end
@enduml
```

Remarques :

- Certaines des classes et des méthodes utilisées ne sont pas mentionnées dans le diagramme de classes ; elles peuvent être rajoutées après coup, au moment de l'établissement des nouveaux diagrammes et du raffinement de la spécification : c'est une démarche typique du processus itératif de modélisation.
- Les contrôleurs et le responsable d'aire de stockage s'informent en cascade de la progression du processus par des échanges synchrones plutôt que par des évènements / notifications (donc des messages asynchrones) : cela permet de se prémunir contre la perte d'un message qui pourrait interrompre le processus suivi par une marchandise au risque dde la laisser indéfiniment en attente à l'intérieur du port.
- Rappel :
  - les fragments combinés "alt" sont utilisés pour modéliser des conditions multiples ou le traitement d'une ou plusieurs exceptions (i.e interruption d'un processus par un évènement / une anomalie) ;
  - les fragments combinés "opt" sont utilisés pour modéliser une interaction (ou une suite d'interactions) qui s'exécutent si un unique condition est vérifiée.
