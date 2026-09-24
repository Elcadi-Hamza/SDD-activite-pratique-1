# Rapport de l'Activité Pratique N°1 - Implémentation d'un microservice avec Spring Boot

`Référence :` https://www.youtube.com/watch?v=2-qIoZcvhAw \
`Réalisé par :` HAMZA ELCADI - GLSID3

---

## 1 - Création du projet :

Nous avons utilisé : https://start.spring.io/ \
![Création du projet](screenShots/1.png) \
avec les dépendances :
- *SPRING DATA JPA*
- *H2 DATABASE*
- *SPRING WEB*
- *LOMBOK*
- *SPRING FOR GRAPHQL*

## 2 - Création des entités JPA, des énumérations et des repositories :

Nous avons créé l'entité `BankAccount` (avec la méthode Builder), l'énumération `AccountType` (`SAVING_ACCOUNT`, `CURRENT_ACCOUNT`), ainsi que le repository `BankAccountRepository`. \
Après, nous avons ajouté des comptes bancaires à l'aide de `CommandLineRunner`.

```

@Bean
CommandLineRunner start(BankAccountRepository bankAccountRepository) {
    return args -> {
        for (int i = 0 ; i < 10 ; i++){
            BankAccount bankAccount = BankAccount.builder()
            .id(UUID.randomUUID().toString())
            .type(Math.random()>0.5? AccountType.CURRENT_ACCOUNT: AccountType.SAVING_ACCOUNT)
            .balance(1000+Math.random()*90000)
            .createdAt(new Date())
            .currency("MAD")
            .build();
            bankAccountRepository.save(bankAccount);
        }
    };
}

```

Après, nous avons mis à jour le fichier `application.properties` :

```

spring.datasource.url=jdbc:h2:mem:account-db
spring.h2.console.enabled=true
server.port=8081

````

Après, nous nous sommes connectés à H2 : \
![Écran de connexion H2](screenShots/2.png) \
![Création du projet](screenShots/3.png) \

## 3 - Création du service REST permettant de gérer des comptes

Nous avons créé le package `web` avec la classe `AccountRestController` et nous avons défini les mappings ainsi que les méthodes permettant d'obtenir la liste des comptes bancaires et un seul compte bancaire par son identifiant. \
Puis, nous avons créé deux méthodes pour ajouter, modifier et supprimer des comptes.

```java
@RestController
public class AccountRestController {
    private BankAccountRepository bankAccountRepository;

    public AccountRestController(BankAccountRepository bankAccountRepository) {
        this.bankAccountRepository = bankAccountRepository;
    }

    @GetMapping("/bankAccounts")
    public List <BankAccount> bankAccounts() {
        return  bankAccountRepository.findAll();
    }

    @GetMapping("/bankAccounts/{id}")
    public BankAccount bankAccount(@PathVariable String id) {
        return  bankAccountRepository.findById(id)
                .orElseThrow(() -> new RuntimeException(String.format("account %s not found",id)));
    }

    @PostMapping("/bankAccounts")
    public BankAccount save (@RequestBody BankAccount bankAccount) {
        if(bankAccount.getId() == null) bankAccount.setId(UUID.randomUUID().toString());
        if(bankAccount.getCreatedAt() == null) bankAccount.setCreatedAt(new Date());
        return bankAccountRepository.save(bankAccount);
    }
    @PutMapping("/bankAccounts/{id}")
    public BankAccount update (@PathVariable String id, @RequestBody BankAccount bankAccount) {
        BankAccount account = bankAccountRepository.findById(id).orElseThrow();
        if(bankAccount.getBalance() != null) account.setBalance(bankAccount.getBalance());
        if(bankAccount.getCurrency() != null) account.setCurrency(bankAccount.getCurrency());
        if(bankAccount.getType() != null) account.setType(bankAccount.getType());
        if(bankAccount.getCreatedAt() != null) account.setCreatedAt(new Date());
        return bankAccountRepository.save(account);
    }

    @DeleteMapping("/bankAccounts/{id}")
    public void delete (@PathVariable String id) {
        bankAccountRepository.deleteById(id);
    }
}
````

Nous avons effectué les tests :
![Liste des comptes bancaires](screenShots/4.png)
![Informations d'un compte bancaire par identifiant](screenShots/5.png)
Et si un compte bancaire n'existe pas :
![Compte bancaire introuvable - RuntimeException](screenShots/6.png) \

## 4 - Test du microservice web en utilisant un client REST comme Postman

Nous allons tester avec `Postman`.

![Liste des comptes bancaires avec Postman](screenShots/7.png)
![Compte bancaire par identifiant avec Postman](screenShots/8.png)
![Ajout d'un compte bancaire avec Postman](screenShots/9.png)
![Modification d'un compte bancaire avec Postman](screenShots/10.png)
![Suppression d'un compte bancaire avec Postman](screenShots/11.png) \

## 5. Génération et test de la documentation Swagger des API REST du service web

Tout d'abord, nous ajoutons la dépendance de Swagger.

Nous allons sur le site https://mvnrepository.com/artifact/org.springdoc/springdoc-openapi-ui/1.6.11 et nous copions la dépendance dans le fichier Maven `pom.xml`.

![Site Maven Repository](screenShots/12.png) 

Note : cette méthode ne fonctionne plus, nous utilisons donc cette dépendance à la place :

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>3.1.1</version>
</dependency>
```

Nous obtenons la documentation à l'adresse http://localhost:8081/swagger-ui/index.html

![Documentation Swagger](screenShots/13.png)
![Exemple de test et d'exécution avec Swagger](screenShots/14.png) \

Nous pouvons exporter Swagger vers d'autres outils comme Postman en utilisant simplement le lien.

![Importation de Swagger dans Postman](screenShots/15.png)
![Récupération de toutes les requêtes dans Postman depuis Swagger](screenShots/16.png) \

