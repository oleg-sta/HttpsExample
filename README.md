## What is it?

Example for app to demonstrate HTTP and HTTPS connection.

### How to watch traffic on local interface
Example for port 8080:
`sudo tcpdump -nnSX -i lo0 port 8080`

### How to generate key store

keytool -genkeypair -keystore keystore.p12 -storetype PKCS12 -storepass password -alias key -keyalg RSA -keysize 2048 -validity 99999 -dname "CN=My SSL Certificate, OU=My Team, O=My Company, L=My City, ST=My State, C=SA" -ext san=dns:localhost,ip:127.0.0.1

## How to make HTTPS connection?

Put generated keypair into `src/resources`

Add file `src/resources/application.yml` with content:

```
server:
    ssl:
      key-store: classpath:keystore.p12
      key-store-password: password
      key-store-type: pkcs12
      key-alias: key
      key-password: password
    port: 8443
```

To make redirection from HTTP to HTTPS add class `ServerConfig` with content:

```
@Configuration
public class ServerConfig {

    @Bean
    public ServletWebServerFactory servletContainer() {
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory() {
            @Override
            protected void postProcessContext(Context context) {
                SecurityConstraint securityConstraint = new SecurityConstraint();
                securityConstraint.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection = new SecurityCollection();
                collection.addPattern("/*");
                securityConstraint.addCollection(collection);
                context.addConstraint(securityConstraint);
            }
        };
        tomcat.addAdditionalTomcatConnectors(getHttpConnector());
        return tomcat;
    }

    private Connector getHttpConnector() {
        Connector connector = new Connector(TomcatServletWebServerFactory.DEFAULT_PROTOCOL);
        connector.setScheme("http");
        connector.setPort(8081);
        connector.setSecure(false);
        connector.setRedirectPort(8443);
        return connector;
    }
}
```