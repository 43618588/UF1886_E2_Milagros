# Arquitectura

### Contenedores activos

```
NAMES         IMAGE                PORTS                                                                                      STATUS
odoo.18       odoo:18.0            0.0.0.0:8001->8069/tcp, [::]:8001->8069/tcp, 0.0.0.0:8002->8072/tcp, [::]:8002->8072/tcp   Up 40 minutes
postgres.db   postgres:16-alpine   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp                    
```
# Network

### comando

NETWORK ID     NAME                   DRIVER    SCOPE
6a5d88b9d36a   uf1886_e2_rm_default   bridge    local

# Volume

### comando

DRIVER    VOLUME NAME
local     uf1886_e2_rm_odoo-db-data
local     uf1886_e2_rm_odoo-web-data













# Recursos del host

``` 

DeviceID Name                                           Caption
-------- ----                                           -------
CPU0     11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz Intel64 Family 6 Model...


TotalVisibleMemorySize FreePhysicalMemory
---------------------- ------------------
              16507808            6568812

# Arquitectura lógica
### comando:

docker port odoo.18
8069/tcp -> 0.0.0.0:8001
8069/tcp -> [::]:8001
8072/tcp -> 0.0.0.0:8002
8072/tcp -> [::]:8002

# Confirmar Apache Hop está en el host

hop-conf.bat -v

===[Environment Settings - hop-conf.bat]===================================

Java identified as java

HOP_OPTIONS=-Xmx2048m -DHOP_AUDIT_FOLDER=.\audit -DHOP_PLATFORM_OS=Windows -DHOP_PLATFORM_RUNTIME=Conf -DHOP_AUTO_CREATE_CONFIG=Y --add-opens java.xml/jdk.xml.internal=ALL-UNNAMED --add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.lang.invoke=ALL-UNNAMED --add-opens java.base/java.lang.reflect=ALL-UNNAMED --add-opens java.base/java.io=ALL-UNNAMED --add-opens java.base/java.net=ALL-UNNAMED --add-opens java.base/java.nio=ALL-UNNAMED --add-opens java.base/java.util=ALL-UNNAMED --add-opens java.base/java.util.concurrent=ALL-UNNAMED --add-opens java.base/java.util.concurrent.atomic=ALL-UNNAMED --add-opens java.base/sun.nio.ch=ALL-UNNAMED --add-opens java.base/sun.nio.cs=ALL-UNNAMED --add-opens java.base/sun.security.action=ALL-UNNAMED --add-opens java.base/sun.util.calendar=ALL-UNNAMED --add-opens java.security.jgss/sun.security.krb5=ALL-UNNAMED --add-exports java.base/sun.nio.ch=ALL-UNNAMED

Command to start Hop will be:
java -classpath lib\core\*;lib\beam\*;lib\swt\win64\* -Djava.library.path=lib\core;lib\beam -Xmx2048m -DHOP_AUDIT_FOLDER=.\audit -DHOP_PLATFORM_OS=Windows -DHOP_PLATFORM_RUNTIME=Conf -DHOP_AUTO_CREATE_CONFIG=Y --add-opens java.xml/jdk.xml.internal=ALL-UNNAMED --add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.lang.invoke=ALL-UNNAMED --add-opens java.base/java.lang.reflect=ALL-UNNAMED --add-opens java.base/java.io=ALL-UNNAMED --add-opens java.base/java.net=ALL-UNNAMED --add-opens java.base/java.nio=ALL-UNNAMED --add-opens java.base/java.util=ALL-UNNAMED --add-opens java.base/java.util.concurrent=ALL-UNNAMED --add-opens java.base/java.util.concurrent.atomic=ALL-UNNAMED --add-opens java.base/sun.nio.ch=ALL-UNNAMED --add-opens java.base/sun.nio.cs=ALL-UNNAMED --add-opens java.base/sun.security.action=ALL-UNNAMED --add-opens java.base/sun.util.calendar=ALL-UNNAMED --add-opens java.security.jgss/sun.security.krb5=ALL-UNNAMED --add-exports java.base/sun.nio.ch=ALL-UNNAMED org.apache.hop.config.HopConfig -v

===[Starting HopConfig]=========================================================
WARNING: package sun.security.action not in java.base
WARNING: A terminally deprecated method in sun.misc.Unsafe has been called
WARNING: sun.misc.Unsafe::objectFieldOffset has been called by com.google.common.util.concurrent.AbstractFuture$UnsafeAtomicHelper (file:/C:/Users/Docente/Documents/hop/lib/core/guava-33.3.1-jre.jar)
WARNING: Please consider reporting this to the maintainers of class com.google.common.util.concurrent.AbstractFuture$UnsafeAtomicHelper
WARNING: sun.misc.Unsafe::objectFieldOffset will be removed in a future release
2.16.0












### Datos disponibles


### Crear Schema Staging y tabla destino
```


```

