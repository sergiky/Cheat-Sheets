# ^ y $
**Obliga que la cadena sea extricta**, esto significa que no vale que pongan 8 caracteres y una letra, si no que se pongan en el orden que yo he definido
``^`` --> Inicio
``$`` --> Final

# Corchetes
Los corchetes **indican lo que está incluido**, pueden ser números \[0-9] o de la a a la z \[a-z], las dos cosas \[0-9a-z], no hace falta que sea todo el rango, puede ser \[5-9] o de \[f-k], recuerda que **con las mayúsculas se vería así** \[A-Z]

### Espacio
para poner un espacio sería así \[ ] o incluso no haría falta ponerlo entre corchetes, ej:
````java
^([A-Z][a-z]+ ?){1,2}$
````

Fijate que entre el + y la ? **hay un espacio**

# Llaves
Las llaves para indicar **cuantas repeticiones queremos** de una cosa (por ejemplo de los corchetes puestos anteriormente) {8}, si queremos que puedan ser dos valores {7,8}

**Cuando solo queremos que sea un número/letra no hace falta indicarlo entre dichas llaves**

En caso de que quieras hacer un rango sería
Número introducido entre 0 y 3 decimales
````java
{0,3}
````

# Paréntesis ()
**Para ir separando por trozos** (como siempre)

````java
"^([A-Z]{1}[a-z]+[ ]?){1,2}$"
````
Estamos indicando que **todo lo que está dentro** de los paréntesis queremos que se repita una o dos veces, 

## Operador OR ( | )
Si no es este numero, que sea este otro, o este, o este... Al igual con las letras

````java
[T|R|W|A|G|M|Y|F|P|D|X|B]
````

O una mejor forma sería de la siguiente manera:
````java
[TRWAGMYFPDXB]
````
Cuando pones **una letra al lado** de otra estás haciendo la misma función

# Más ( + )
Que tiene que estar **una o más veces**, que minimamente este una letra o un número o que aparezca más veces

# Interrogación ( ? )
Significa que puede ser **0 o 1** ocurrencia
````java
[ ]? // Un espacio o ningún espacio
````
Estamos indicando que el espacio puede estar o no

### Pueda ser o no negativo (número entero)
Con el -? estamos indicando que puede existir un - o no
````java
^-?[0-9]+$
````

# asterisco ( * )
Significa que puede ser **0 o varias** veces
La diferencia de la ? es que si es más de un carácter este lo coge
````java
[ ]* // Ningún espacio o un espacio o más de un espacio
````

El segundo nombre se puede introducir pegado al primero sin el espacio, no poner el espacio como opcional

# punto ( . )
El punto significa **cualquier carácter (incluidos espacios...) menos un salto de línea**, en java se representar así (\\n)

# \[^] Qué no sea cierto carácter
Podemos indicar que no queremos cierto carácter, pero que los demás si sean válidos

Todos los caracteres menos el hastag
````java
[^#]
````

Todos los caracteres menos del 0 al 9
````java
[^0-9]
````

# Metacaracteres | Clases de caracteres abreviados
``\d`` = \[0-9] --> Cualquier caracter que sea del 0 al 9
``\D`` = \[^0-9] --> Cualquier carácter que **no sea** del 0 al 9

`\w` = \[a-zA-Z0-9_] --> Coincide con culaquier palabra que tenga letra,cifra o guión bajo
``\W`` --> Cualquiera que **no sea** una letra, una cifra o un guión bajo
`\s` = \[\t\n\v\f\r ] --> Coincide con cualquier carácter que sea un espacio en blanco
`\S` --> Cualquier caracter que **no sea** un espacio en blanco


# Ejemplos 

## Validar DNI 
Recuerda que no todas las letras del DNI **son oficiales**
````java
public static boolean validarDNI(String DNI){  
    return DNI.matches("^[0-9]{8}[TRWAGMYFPDXB]$");  
}
````
## Validar nombre

````java
public static boolean validarNombre(String nombre){  
    return nombre.matches("^([A-Z][a-z]+ ?){1,2}$");  
}
````

## Validar un número entero
````java
public static boolean validarNumeroEntero(String numero){  
    return numero.matches("^-?[0-9]+$");  
}
````

## Validar un número entero positivo
````java
public static boolean validarNumeroEnteroPositivo(String numero){  
    return numero.matches("^[0-9]+$");  
}
````

## Validar un número entero negativo
````java
public static boolean validarNumeroEnteroNegativo(String numero){  
    return numero.matches("^-[0-9]+$");  
}
````
Como no hemos puesto nada detrás del -, esto significa que tiene que estar este carácter si o si

## Validar un número binario
````java
public static boolean validarBinario(String binario){
	return binario.matches("[01]+");
}
````
En caso de poner espacios esto ya no funcionaría

## Validar un número octal

````java
public static boolean validarOctal(String octal){
	return octal.matches("[0-7]+");
}
````

## Validar un número hexadecimal
````java
public static boolean validarHexadecimal(String hexadecimal){
	return hexadecimal.matches("[0-9A-F]+");
}
````

## Validar dos caracteres cualquiera

Ya puede ser una letra, un número una interrogación, **menos con los saltos de línea**
````java
public static boolean validarDosCaracteres(String palabra){
	return palabra.matches(".{2}");
}
````

## Validar un número real
````java
System.out.println(numeroReal + " = " + numeroReal.matches("^-?[0-9]+[.,]?[0-9]*$"));
````

Recuerda que con asterisco indicamos que puede ser 0 o varias veces

## Validar fecha (del 2000 hasta el 2023)
````java
System.out.println(fecha + " = " + fecha.matches( "^(0?[1-9]|[12]\\d|3[01])/(0?[1-9]|1[0-2])/(20[0-2][0-3])"));
````

## Validar correo electrónico
````java
System.out.println(email + " = " + email.matches("^([^@]+)@([^@]+).([^@]+)$"));
````

# Fuentes
[Regex java-videos](https://www.youtube.com/watch?v=VyG7sWephmI&list=PLaxZkGlLWHGVp6fjX7Q2KuYgpG0N9pOYn)
[Google-Cheat-Sheet](https://support.google.com/a/answer/1371415?hl=es)
[Cheattography](https://cheatography.com/davechild/cheat-sheets/regular-expressions/)
