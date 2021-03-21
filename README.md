# Spring Boot Configuration with Jasypt

# 1. Introdução
Jasypt (Java Simplified Encryption) Spring Boot fornece utilitários para criptografar fontes de propriedade em aplicativos de inicialização.

Neste artigo, discutiremos como podemos adicionar o suporte do jasypt-spring-boot e usá-lo.

# 2. Por que Jasypt?
Sempre que precisamos armazenar informações confidenciais no arquivo de configuração - isso significa que estamos essencialmente tornando essas informações vulneráveis; isso inclui qualquer tipo de informação sensível, como credenciais, mas certamente muito mais do que isso.

Ao usar o Jasypt, podemos fornecer criptografia para os atributos do arquivo de propriedade e nosso aplicativo fará o trabalho de descriptografá-lo e recuperar o valor original.

# 3. Maneiras de usar JASYPT com Spring Boot
Vamos discutir as diferentes maneiras de usar Jasypt com Spring Boot.

### 3.1. Usando jasypt-spring-boot-starter
Precisamos adicionar uma única dependência ao nosso projeto:

```
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>
```

O Maven Central tem a versão mais recente do jasypt-spring-boot-starter.

Vamos agora criptografar o texto “Password@1” com a chave secreta “password” e adicioná-la ao criptografado.properties:

```
encrypted.property=ENC(uTSqb9grs1+vUv3iN8lItC0kl65lMG+8)
```

E vamos definir uma classe de configuração AppConfigForJasyptStarter - para especificar o arquivo criptografado.properties como um PropertySource:

```
@Configuration
@PropertySource("encrypted.properties")
public class AppConfigForJasyptStarter {
}
```

Agora, escreveremos um bean de serviço PropertyServiceForJasyptStarter para recuperar os valores de encryption.properties. O valor descriptografado pode ser recuperado usando a anotação @Value ou o método getProperty() da classe Environment:

```
@Service
public class PropertyServiceForJasyptStarter {

    @Value("${encrypted.property}")
    private String property;

    public String getProperty() {
        return property;
    }

    public String getPasswordUsingEnvironment(Environment environment) {
        return environment.getProperty("encrypted.property");
    }
}
```

Finalmente, usando a classe de serviço acima e definindo a chave secreta que usamos para criptografia, podemos facilmente recuperar a senha descriptografada e usar em nosso aplicativo:

```
@Test
public void whenDecryptedPasswordNeeded_GetFromService() {
    System.setProperty("jasypt.encryptor.password", "password");
    PropertyServiceForJasyptStarter service = appCtx
      .getBean(PropertyServiceForJasyptStarter.class);
 
    assertEquals("Password@1", service.getProperty());
 
    Environment environment = appCtx.getBean(Environment.class);
 
    assertEquals(
      "Password@1", 
      service.getPasswordUsingEnvironment(environment));
}
```

### 3.2. Usando jasypt-spring-boot
Para projetos que não usam @SpringBootApplication ou @EnableAutoConfiguration, podemos usar a dependência jasypt-spring-boot diretamente:

```
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot</artifactId>
    <version>2.0.0</version>
</dependency>
```

Da mesma forma, vamos criptografar o texto “Senha@2” com a chave secreta “senha” e adicioná-la às propriedades criptografadasv2.propriedades:

```
encryptedv2.property=ENC(dQWokHUXXFe+OqXRZYWu22BpXoRZ0Drt)
```

E vamos ter uma nova classe de configuração para a dependência jasypt-spring-boot.

Aqui, precisamos adicionar a anotação @EncryptablePropertySource:

```
@Configuration
@EncryptablePropertySource("encryptedv2.properties")
public class AppConfigForJasyptSimple {
}
```

Além disso, um novo bean PropertyServiceForJasyptSimple para retornar criptografadov2.properties é definido:

```
@Service
public class PropertyServiceForJasyptSimple {
 
    @Value("${encryptedv2.property}")
    private String property;

    public String getProperty() {
        return property;
    }
}
```

Finalmente, usando a classe de serviço acima e definindo a chave secreta que usamos para criptografia, podemos recuperar facilmente a propriedade criptografadav2.property:

```
@Test
public void whenDecryptedPasswordNeeded_GetFromService() {
    System.setProperty("jasypt.encryptor.password", "password");
    PropertyServiceForJasyptSimple service = appCtx
      .getBean(PropertyServiceForJasyptSimple.class);
 
    assertEquals("Password@2", service.getProperty());
}
```

### 3.3. Usando o criptografador JASYPT personalizado

Os criptografadores definidos na seção 3.1. e 3.2. são construídos com os valores de configuração padrão.

No entanto, vamos definir nosso próprio criptografador Jasypt e tentar usá-lo em nosso aplicativo.

S0, o bean criptografador personalizado será semelhante a:


```
@Bean(name = "encryptorBean")
public StringEncryptor stringEncryptor() {
    PooledPBEStringEncryptor encryptor = new PooledPBEStringEncryptor();
    SimpleStringPBEConfig config = new SimpleStringPBEConfig();
    config.setPassword("password");
    config.setAlgorithm("PBEWithMD5AndDES");
    config.setKeyObtentionIterations("1000");
    config.setPoolSize("1");
    config.setProviderName("SunJCE");
    config.setSaltGeneratorClassName("org.jasypt.salt.RandomSaltGenerator");
    config.setStringOutputType("base64");
    encryptor.setConfig(config);
    return encryptor;
}
```

Além disso, podemos modificar todas as propriedades do SimpleStringPBEConfig.

Além disso, precisamos adicionar uma propriedade “jasypt.encryptor.bean” ao nosso application.properties, para que o Spring Boot saiba qual Custom Encryptor ele deve usar.

Por exemplo, adicionamos o texto personalizado “Password @ 3” criptografado com a chave secreta “password” em application.properties:

```
jasypt.encryptor.bean=encryptorBean
encryptedv3.property=ENC(askygdq8PHapYFnlX6WsTwZZOxWInq+i)
```

Depois de configurá-lo, podemos facilmente obter o encryptionv3.property do ambiente do Spring:

```
@Test
public void whenConfiguredExcryptorUsed_ReturnCustomEncryptor() {
    Environment environment = appCtx.getBean(Environment.class);
 
    assertEquals(
      "Password@3", 
      environment.getProperty("encryptedv3.property"));
}
```

# 4. Conclusão
Ao usar o Jasypt, podemos fornecer segurança adicional para os dados que o aplicativo manipula.

Ele nos permite focar mais no núcleo de nosso aplicativo e também pode ser usado para fornecer criptografia personalizada, se necessário.