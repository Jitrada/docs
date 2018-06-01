% Encryption

## Jasypt Encryption Plugin

Provides an encryption [storage converter](storage-plugins.html#storage-converter) for the Storage facility.  Can be used to encrypt the contents of Key Storage,
and Project Configuration stored in the DB or on disk.

This plugin provides password based encryption for storage contents.  
It uses the [Jasypt][] encryption library. The built in Java JCE is used unless another provider is specified, [Bouncycastle][] can be used by specifying the 'BC' provider name.

[Jasypt]: (http://jasypt.org)
[Bouncycastle]: https://www.bouncycastle.org/

Password, algorithm, provider, etc can be specified directly, or via environment variables (the `*EnvVarName` properties), or Java System properties (the `*SysPropName` properties).

To enable it, see [Configuring - Storage Converter Plugins](configuring.html#storage-converter-plugins).

See also: [Key Storage](../administration/security/key-storage.html)

Provider type: `jasypt-encryption`

The following encryption properties marked with `*` can be set directly, 
using the property name shown,
but they can all also be set dynamically using either an Environment variable, 
or a Java System Property.  
Append either `EnvVarName` for the environment variable, 
or `SysPropName` to use the Java System Property.  
If a System Property is specified: it is read in once and used by the initialization of the converter plugin,
then the Java System Property is set to null so it cannot be read again.

Configuration properties:  

`encryptorType`

:   Jasypt Encryptor to use. Either `basic`, `strong`, or `custom`. Default: 'basic'.

	* `basic` uses algorithm `PBEWithMD5AndDES`
	* `strong` requires use of the JCE Unlimited Strength policy files. (Algorithm: `PBEWithMD5AndTripleDES`)
	* `custom` is required to specify the algorithm.

`password*`
:   the password.

`algorithm*`
:   the encryption algorithm.

`provider*`
:   the provider name. 'BC' indicates Bouncycastle.

`providerClassName*`
:   Java class name of the provider.

`keyObtentionIterations*`
:   Number of hashes to use for the password when generating the key, default is 1000.

Example configuration for the Key Storage facility:

	rundeck.storage.converter.1.type=jasypt-encryption
	rundeck.storage.converter.1.path=keys
	rundeck.storage.converter.1.config.encryptorType=custom
	rundeck.storage.converter.1.config.passwordEnvVarName=ENC_PASSWORD
	rundeck.storage.converter.1.config.algorithm=PBEWITHSHA256AND128BITAES-CBC-BC
	rundeck.storage.converter.1.config.provider=BC

Example configuration for the Project Configuration storage facility:

	rundeck.config.storage.converter.1.type=jasypt-encryption
	rundeck.config.storage.converter.1.path=/
	rundeck.config.storage.converter.1.config.password=sekrit
	rundeck.config.storage.converter.1.config.encryptorType=custom
	rundeck.config.storage.converter.1.config.algorithm=PBEWITHSHA256AND128BITAES-CBC-BC
	rundeck.config.storage.converter.1.config.provider=BC


File: `rundeck-jasypt-encryption-plugin-${VERSION}.jar`











## Pro encryption?





This plugin provides a way to store your `dataSource.password` in an encrypted form inside your 
`rundeck-config.properties` file.

**Note:**

This plugin behavior may change in the future.

## Usage

This plugin uses [Jasypt](http://www.jasypt.org/) to easily encrypt/decrypt passwords, and a simple way to use them within
your rundeck config file.

To encrypt the password, follow the instructions below:

1. Encrypt a password.  You do this by providing a master password.
2. Convert your rundeck config file to groovy format

When Rundeck server starts up, it requires the master password to be available to let it decrypt the database password.
You can provide the master password at startup in several ways:

1. Enter it on the console. (default). Rundeck will prompt for the master password at startup.
2. Define an environment variable available to the server at startup: `RD_ENCRYPTION_DEFAULT_PASSWORD`


You can define multiple "master passwords".  The default is called "default", but you can define a "db" password, etc.

You specify which "master password" configuration to use when you encrypt/decrypt the password. 

### Encrypt Password

To encrypt the database password, use the `encrypt` command.  

Usage: `encrypt [config] [value]`

* `config` (configuration name), default: "default"
* `value`  value to encrypt/decrypt (prompted if not provided)

Example:

```
[rundeck ~]$ java -cp server/exp/webapp/WEB-INF/lib/grails-plugin-encrypt-datasource-password-2.1.0.jar:server/exp/webapp/WEB-INF/lib/* \ 
  rundeck.codecs.EncryptCodec encrypt 'password'
Enter master password for [default]: 
```

### Move the rundeck-config.properties to a rundeck-config.groovy

See the [FAQ about converting to groovy format][faqconfiggroovy].

Specify the new file path at startup:

* Launcher:

        java -jar -Drundeck.config.name=rundeck-config.groovy rundeck-launcher.jar

* RPM: Add this to the `/etc/sysconfig/rundeckd` file:

        export RDECK_CONFIG_FILE="/etc/rundeck/rundeck-config.groovy"

* DEB: Add this to the `/etc/default/rundeckd` file:

        export RDECK_CONFIG_FILE="/etc/rundeck/rundeck-config.groovy"

[faqconfiggroovy]: https://github.com/rundeck/rundeck/wiki/Faq#how-do-i-convert-my-rundeck-config-file-to-groovy
[groovyconfig]: http://rundeck.org/docs/administration/configuration-file-reference.html#groovy-config-format

### Verify the groovy config change

After you convert to `rundeck-config.groovy` format, it is best to restart the Rundeck server and verify your configuration
works the same, before replacing the plaintext datasource password.

### Edit rundeck-config.groovy to encrypt the datasource password:

After you have converted to `.groovy` format, add the lines shown below.  In place of the cleartext database password, 
you would use `decrypt 'encryptedpassword','default', true`.

This means to decrypt the encrypted password, using the `default` configuration, and to allow prompting on the console for the password.

The default is the `default` configuration, and `true` for prompting on the console, so this is the same: `decrypt 'encryptedpassword'`.  You can specify a different configuration name, or use `false` to prevent console prompting.

{% highlight groovy %}

//add import statement for the decrypt command
import static rundeck.codecs.EncryptCodec.decrypt

dataSource.dbCreate="update"
dataSource.url="jdbc:mysql://server/rundeckdb?autoReconnect=true"
dataSource.username="rundeckuser"
dataSource.driverClassName="com.mysql.jdbc.Driver"

//set the password to the result of decrypting
dataSource.password=decrypt 'encryptedpassword'

{% endhighlight %}

And the system environment should be defined as:

    [rundeck ~]$ export RD_ENCRYPTION_DEFAULT_PASSWORD=myDefaultmasterpassword

**Note:** if you use a different configuration name, be sure to specify that in the `decrypt '...','configname'` line,
as well as the Environment variable, eg. `RD_ENCRYPTION_DB_PASSWORD=myDBmasterpassword`

### Restart Rundeck

* if the container is tomcat, and the secret key will be added using console. It is necessary to start tomcat with `catalina.sh start` or `catalina.bat start`
