# amazon-rds-connection-test
Sample spring-boot application which connect Amazon RDS for mySQL to test TLS encryption



### Pre-requsites

Create a Amazon RDS database for mySQL instance. Declare environment variables below

```
RDS_ENDPOINT=database-1.somerandomstring.ap-northeast-2.rds.amazonaws.com
ADMIN_USERNAME=admin
ADMIN_PASSWORD={enter-password-here}
```





### Build a Spring Boot project

1. Install Spring boot CLI

   

```
brew tap spring-io/tap
brew install spring-boot
```



2. Create a new Spring Boot Project

```
spring init --build=maven --java-version=17 --dependencies=web,data-jpa,mysql --groupId=com.example --artifactId=rds-demo --name=RDSDemo rds-demo

cd rds-demo
```



3. Create a database in Amazon RDS for mySQL

```
mysql -h $RDS_ENDPOINT -u admin -p
mysql> CREATE DATABASE myapp_db;
mysql> USE myapp_db;
Database changed
```



4. Add AWS MySQL JDBC driver dependency in 'pom.xml

As of writing, the latest AWS JDBC Drive for MySQL version is 1.1.15

```
<dependency>
    <groupId>software.aws.rds</groupId>
    <artifactId>aws-mysql-jdbc</artifactId>
    <version>1.1.15</version>
</dependency>
```



3. Create an 'application.properties'

```
cat << EOF > src/main/resources/application.properties
spring.application.name=demo
spring.datasource.url=jdbc:mysql://${RDS_ENDPOINT}:3306/myapp_db?useSSL=true&requireSSL=true
spring.datasource.username=${ADMIN_USERNAME}
spring.datasource.password=${ADMIN_PASSWORD}
spring.datasource.driver-class-name=software.aws.rds.jdbc.mysql.Driver
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
server.port = 8090
EOF

```



4. Add 'User.java'

```
cat << EOF > src/main/java/com/example/rds_demo/User.java
package com.example.demo;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String email;

    // Getters and setters
}
EOF
```



4. Add 'UserController.java'

```
cat << EOF > src/main/java/com/example/rds_demo/UserController.java
package com.example.demo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/users")
public class UserController {

    @Autowired
    private UserRepository userRepository;

    @GetMapping
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        return userRepository.save(user);
    }
}
EOF
```





4. Add 'UserRepository.java'

```
cat << EOF > src/main/java/com/example/rds_demo/UserRepository.java
package com.example.demo;

import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
}
EOF
```



5. Build & run 

```
mvn clean install
mvn spring-boot:run 
```



### Run with TLS handshake debug message

**RDS for SQL engine version : 5.7.44**

```
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Djavax.net.debug=ssl:handshake" 
...
"ClientHello": {
  "client version"      : "TLSv1.2",
  "random"              : "8DC0F02666F2349EA2DF17CB7096D3B6A187A8F2584DA30C1DDFEC8A1EE3F04F",
  "session id"          : "7A6E62857226BA350EC7AE826043C6648FFE8CA868219C074AA0872059646255",
  "compression methods" : "00",
  "extensions"          : [
    "server_name (0)": {
      type=host_name (0), value=database-1.cs7h2pruemmr.ap-northeast-2.rds.amazonaws.com
    },
    "status_request (5)": {
      "certificate status type": ocsp
      "OCSP status request": {
        "responder_id": <empty>
        "request extensions": {
          <empty>
        }
      }
    },
    "supported_groups (10)": {
      "named groups": [x25519, secp256r1, secp384r1, secp521r1, x448, ffdhe2048, ffdhe3072, ffdhe4096, ffdhe6144, ffdhe8192]
    },
    "ec_point_formats (11)": {
      "formats": [uncompressed]
    },
    "status_request_v2 (17)": {
      "cert status request": {
        "certificate status type": ocsp_multi
        "OCSP status request": {
          "responder_id": <empty>
          "request extensions": {
            <empty>
          }
        }
      }
    },
    "extended_master_secret (23)": {
      <empty>
    },
    "session_ticket (35)": {
      <empty>
    },
    "signature_algorithms (13)": {
      "signature schemes": [ecdsa_secp256r1_sha256, ecdsa_secp384r1_sha384, ecdsa_secp521r1_sha512, ed25519, ed448, rsa_pss_rsae_sha256, rsa_pss_rsae_sha384, rsa_pss_rsae_sha512, rsa_pss_pss_sha256, rsa_pss_pss_sha384, rsa_pss_pss_sha512, rsa_pkcs1_sha256, rsa_pkcs1_sha384, rsa_pkcs1_sha512, dsa_sha256, ecdsa_sha224, rsa_sha224, dsa_sha224, ecdsa_sha1, rsa_pkcs1_sha1, dsa_sha1]
    },
    "supported_versions (43)": {
      "versions": [TLSv1.3, TLSv1.2]
    },
    "psk_key_exchange_modes (45)": {
      "ke_modes": [psk_dhe_ke]
    },
    "signature_algorithms_cert (50)": {
      "signature schemes": [ecdsa_secp256r1_sha256, ecdsa_secp384r1_sha384, ecdsa_secp521r1_sha512, ed25519, ed448, rsa_pss_rsae_sha256, rsa_pss_rsae_sha384, rsa_pss_rsae_sha512, rsa_pss_pss_sha256, rsa_pss_pss_sha384, rsa_pss_pss_sha512, rsa_pkcs1_sha256, rsa_pkcs1_sha384, rsa_pkcs1_sha512, dsa_sha256, ecdsa_sha224, rsa_sha224, dsa_sha224, ecdsa_sha1, rsa_pkcs1_sha1, dsa_sha1]
    },
    "key_share (51)": {
      "client_shares": [
        {
          "named group": x25519
          "key_exchange": {
            0000: 94 D1 6A 99 2C 30 84 3B   DE 93 68 2E C5 BE F6 FC  ..j.,0.;..h.....
            0010: D4 E6 4D 4D DB 7B 25 BB   44 96 67 76 69 0F 9E 2B  ..MM..%.D.gvi..+
          }
        },
        {
          "named group": secp256r1
          "key_exchange": {
            0000: 04 72 B7 8A 23 AD F6 23   6B 4E CC AF D9 A3 09 D6  .r..#..#kN......
            0010: 24 D0 E9 E9 87 4C 2F C0   DA 16 25 A1 53 E3 21 B7  $....L/...%.S.!.
            0020: B2 E6 E9 8F 74 C1 72 0E   B2 2B F9 0D 67 67 D7 1D  ....t.r..+..gg..
            0030: A8 E8 FA 2E D9 4D 24 4C   5C D2 78 78 C5 EA 0C 8A  .....M$L\.xx....
            0040: CB
          }
        },
          }
        },
      ]
    },
    "renegotiation_info (65,281)": {
      "renegotiated connection": [<no renegotiated connection>]
    }
  ]
}
)
javax.net.ssl|DEBUG|10|main|2024-07-18 22:49:06.610 KST|ServerHello.java:883|Consuming ServerHello handshake message (
"ServerHello": {
  "server version"      : "TLSv1.2",
  "random"              : "D363F130CA341960D7D3689BA1326CCF3056C7EE81B5CDA12D3D2926E9686CBC",
  "session id"          : "",
  "cipher suite"        : "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384(0xC030)",
  "compression methods" : "00",
  "extensions"          : [
    "renegotiation_info (65,281)": {
      "renegotiated connection": [<no renegotiated connection>]
    },
    "ec_point_formats (11)": {
      "formats": [uncompressed, ansiX962_compressed_prime, ansiX962_compressed_char2]
    },
    "session_ticket (35)": {
      <empty>
    },
    "extended_master_secret (23)": {
      <empty>
    }
  ]
}
)
...
```

