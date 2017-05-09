# How to: Change the HTTPS certificate in KeyBox.

Instructions for changing the default https certificate in KeyBox (web-based SSH console).

**KeyBox Webpage**: http://sshkeybox.com/

## What is needed to change the default HTTPS certificate of KeyBox? 

- Key und Certificate file with certificate chain (PEM Format).
- openssl command.
- keytool command (JDK / JRE package).

## Steps to follow to change the KeyBox certificate.

```bash
# pkcs12 file is created with the private key and HTTPS certificate (+ certificate chain).
openssl pkcs12 -export -inkey XX.key -in XX.crt -out XX.pkcs12

# Create the new KeyStore (Enter a password).
keytool -importkeystore -srckeystore XX.pkcs12 -srcstoretype PKCS12 -destkeystore keystore
Enter destination keystore password:  
Re-enter new password: 
Enter source keystore password:  
Entry for alias 1 successfully imported.
Import command completed:  1 entries successfully imported, 0 entries failed or cancelled

# Back up the files to be replaced.
cp KeyBox-jetty/jetty/etc/keystore KeyBox-jetty/jetty/etc/keystore_old
cp KeyBox-jetty/jetty/etc/jetty-ssl-context.xml KeyBox-jetty/jetty/etc/jetty-ssl-context.xml_old

# Add the new KeyStore file.
cp keystore KeyBox-jetty/jetty/etc/
chown keybox:keybox KeyBox-jetty/jetty/etc/keystore

# Encrypt the password (Example: "S3xYF0t5!") to use it in the XML files.
# Usage - java org.eclipse.jetty.util.security.Password [<user>] <password>
java -cp KeyBox-jetty/jetty/lib/jetty-util-X.X.X.vXXX.jar org.eclipse.jetty.util.security.Password S3xYF0t5!
 
2016-03-01 03:32:03.055:INFO::main: Logging initialized @94ms
XXXXXXX
OBF:uu1wxx151woj1ugo1uvkkuum1uh21w871x1h1uva
MD5:a826e006a1d150cf79c96397163266ee
```

Edit the file "KeyBox-jetty/jetty/etc/jetty-ssl-context.xml" and add the hash.
```xml
<Set name="KeyStorePassword"><Property name="jetty.sslContext.keyStorePassword" deprecated="jetty.keystore.password" default="OBF:uu1wxx151woj1ugo1uvkkuum1uh21w871x1h1uva"/></Set>
<Set name="KeyManagerPassword"><Property name="jetty.sslContext.keyManagerPassword" deprecated="jetty.keymanager.password" default="OBF:uu1wxx151woj1ugo1uvkkuum1uh21w871x1h1uva"/></Set>
<Set name="TrustStorePassword"><Property name="jetty.sslContext.trustStorePassword" deprecated="jetty.truststore.password" default="OBF:uu1wxx151woj1ugo1uvkkuum1uh21w871x1h1uva"/></Set>
  ```
Now we only need restart KeyBox and check with the browser that the new certificate is in use.
