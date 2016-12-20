### Install Vault

#### https://github.com/spring-cloud/spring-cloud-vault-config/tree/master/src/test/bash

```
  $ src/test/bash/install_vault.sh
```

### Create SSL certificates for Vault

```
  $ src/test/bash/create_certificates.sh
```

NOTE: `create_certificates.sh` creates certificates in `work/ca` and a JKS truststore `work/keystore.jks`. If you want to run Spring Cloud Vault using this quickstart guide you need to configure the truststore the `spring.cloud.vault.ssl.trust-store` property to `file:work/keystore.jks`.

### Start Vault server

```
  $ src/test/bash/local_run_vault.sh
```

Vault is started listening on `0.0.0.0:8200` using the `inmem` storage and
`https`.
Vault is sealed and not initialized when starting up
so you need to initialize it first.

```
  $ export VAULT_ADDR="https://localhost:8200"
  $ export VAULT_SKIP_VERIFY=true # Don't do this for production
  $ vault init
```

You should see something like:

```
Key 1: 7149c6a2e16b8833f6eb1e76df03e47f6113a3288b3093faf5033d44f0e70fe701
Key 2: 901c534c7988c18c20435a85213c683bdcf0efcd82e38e2893779f152978c18c02
Key 3: 03ff3948575b1165a20c20ee7c3e6edf04f4cdbe0e82dbff5be49c63f98bc03a03
Key 4: 216ae5cc3ddaf93ceb8e1d15bb9fc3176653f5b738f5f3d1ee00cd7dccbe926e04
Key 5: b2898fc8130929d569c1677ee69dc5f3be57d7c4b494a6062693ce0b1c4d93d805
Initial Root Token: 19aefa97-cccc-bbbb-aaaa-225940e63d76

Vault initialized with 5 keys and a key threshold of 3. Please
securely distribute the above keys. When the Vault is re-sealed,
restarted, or stopped, you must provide at least 3 of these keys
to unseal it again.

Vault does not store the master key. Without at least 3 keys,
your Vault will remain permanently sealed.
```

Vault will initialize and return a set of unsealing keys and the root token.
Pick 3 keys and unseal Vault. Store the Vault token in the `VAULT_TOKEN`
 environment variable.

```
  $ vault unseal (Key 1)
  $ vault unseal (Key 2)
  $ vault unseal (Key 3)
  $ export VAULT_TOKEN=(Root token)
```

### Client Side Usage

To use these features in an application, just build it as a Spring
Boot application that depends on `spring-cloud-vault-config` (e.g. see
the test cases). Example Maven configuration:

### .pom.xml

[source,xml,indent=0,subs="verbatim,quotes,attributes"]
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.1.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-vault-starter-config</artifactId>
        <version>{spring-cloud-version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>

<!-- repositories also needed for snapshots and milestones -->
```

Then you can create a standard Spring Boot application, like this simple HTTP server:

```
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

   @Value("${password: nopwd}")
    public String password;

    @PostConstruct
    private void postConstruct() {
        System.out.println("My password is: " + password);
    }
}
```
hen it runs it will pick up the external configuration from the
default local Vault server on port `8200` if it is running. To modify
the startup behavior you can change the location of the Vault server
using `bootstrap.properties` (like `application.properties` but for
the bootstrap phase of an application context), e.g.

.bootstrap.yml

```
spring:
    application:
        name: my-application
    cloud:
        vault:
            token: 3d5dd20f-160e-e52c-890c-54377e713db1
            scheme: https
            ssl:
              trust-store: file:../../work/keystore.jks
```
### Run spring boot app

#### Before running application write test data to vault
```
  $ vault write secret/my-application password=H@rdT0Gu3ss
```
#### Run application
```
  mvn spring-boot:run
```
#### Optput
```
  My password is: H@rdT0Gu3ss
```
