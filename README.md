# API documentation ZLZ API-Gateway
## Security architecture:

## Authorized user
The authorized users are onboarded in the ZLZ identity management system, which serves as the central hub for user authentication and access control.
To access the system's API, users are required to send a One-Time Password (OTP) generated from the Zentral Labor Zurich (ZLZ) platform. This OTP serves as a secure token for user verification and is available for both production and testing environments.
By utilizing the ZLZ identity management system and the OTP generated from ZLZ, we ensure a secure and reliable means of API access for authorized users.

## TLS tunnel
A TLS tunnel is established (with single-way authentication) between the primary system and the API gateway. To facilitate this secure connection, a primary system certificate is used.

The primary system certificate is specifically configured for this purpose, ensuring the integrity and confidentiality of the communication between the primary system and the API gateway. This certificate helps establish trust and enables secure data transmission over the TLS tunnel.

By implementing this TLS tunnel with single-way authentication and utilizing the primary system certificate, we ensure that the communication between the primary system and the API gateway remains secure and protected from unauthorized access or tampering.

## Content signature
The data sent to the REST API is signed using the private key of the certificate issued by the primary system certificate.

The process is as follows, given the JSON payload that needs to be sent (containing data for the Order Service, including the one-time password):
1. The primary system creates a canonicalized text representation of the payload by removing all spaces, tabs, carriage returns, and newlines. The regular expression ```/[\n\r\t ]/gm``` can be utilized for this purpose.
2. The primary system obtains a UTF-8 byte representation of the canonicalized text by encoding it.
3. The primary system signs the byte representation using the ```RSASSA-PKCS1-v1_5``` algorithm from [ RFC 3447](https://datatracker.ietf.org/doc/html/rfc3447). In most implementations, this algorithm is commonly referred to as ```SHA256withRSA```.
4. The primary system encodes the signature as a base64 string.
5. The primary system includes the base64-encoded signature in the request header labeled as ```X-Signature```.

**Java signature sample**

```java

import java.nio.charset.StandardCharsets;
import java.security.*;
import java.util.Base64;

public class Main {
    public static void main(String[] args) throws Exception {
         String jsonPayload = ; // Replace "" with your actual payload
        String xSignature = generateXSignature(jsonPayload);
    
        // X-Signature to use in API-Gateway
        System.out.println("X-Signature: " + xSignature);
    }

    public static String generateXSignature(String payload) throws Exception {
        // load the key
        PrivateKey key = primarySystem.getCertificate(); // Replace "primarySystem.getCertificate()" with the actual code to load the key
        
        // canonicalize

        String normalizedJson = payload.replaceAll("[\\n\\r\\t ]", "");
        byte[] bytes = normalizedJson.getBytes(StandardCharsets.UTF_8);
        
        // sign
        Signature signature = Signature.getInstance("SHA256withRSA");
        signature.initSign(key);
        signature.update(bytes);
        byte[] signatureBytes = signature.sign();
        String xSignature = Base64.getEncoder().encodeToString(signatureBytes);
        
        return xSignature;
    }
}
```


