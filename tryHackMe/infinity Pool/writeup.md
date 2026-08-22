![alt text](img/image.png)

# Contexto
Byte Lotus Hotel promises a seamless stay powered by modern technology. Sometimes the most interesting systems are the ones guests were never meant to see.

# Preguntas
- What is the user flag?
- What is the root flag?

# Solucion

## Web
Accedemos al servicion web.

![alt text](img/image-1.png)

### Fuzzing
Para esta parte usaremos **'gobuster'** para hacer el fuzzing.

![alt text](img/image-2.png)

Encontramos el endpoint **'status'** y vemos que podemos hacer ping a direcciones que introduzcamos.

![alt text](img/image-3.png)

Intentamos introducir **';'** para ejecutar comandos y tuvimos exito.

![alt text](img/image-4.png)

Intentamos pasarnos un revershell con bash.

![alt text](img/image-5.png)

## Respuesta a 'What is the user flag?'

Obtuvimos acceso a la maquina con el usuario **'web'**, nos vamos a nuestro directorio home y podemos ver la bandera.

![alt text](img/image-6.png)

## Ssh
Para trabajar mejor nos creamos persistencia en el equipo, para poder tener una terminal mas facil de usar. 

De nuestra maquina atacante copiamos nuestra clave publica en el archivo **'authorized_keys'** de la maquina victima.

![alt text](img/image-7.png)

Esto para conectar por ssh sin necesidad de ingresar credenciales.

![alt text](img/image-8.png)

## Procesos corriendo

Para esta parte usaremos **'pspy64'** para poder ver los procesos que estan en ejecucion.

Asi que primero nos creamos un servidor http con python en nuestra maquina atacante.
 
    sudo python3 -m http.server PORT

Desde la maquina victima nos descargamos el recurso.

![alt text](img/image-10.png)

![alt text](img/image-9.png)

Le damos permisos de ejecucion y corremos la herramiento.

![alt text](img/image-11.png)

Hay dos procesos que nos llaman la atencion.

![alt text](img/image-12.png)

Estos procesos crean servicios en los puertos **'3000'** y **'9000'** visibles de manera local.

## Port forwarding

Manejaremos **'chisel'** para poder acceder a los puertos abierto en la maquina victima en nuestra maquina atacante.

Primero creamos un servidor con chisel en nuestra maquina atacante.
    
    chisel server -p PORT --reverse

![alt text](img/image-15.png)

Chisel tiene una version como binario asi que nos la pasamos a la maquina victima con el servidor python que creamos anteriormente.

![alt text](img/image-13.png)

![alt text](img/image-14.png)

Le damos permisos de ejecucion y usamos chisel en modo cliente para los puerto 3000 y 9000.

    ./chisel client IP_SERVER:PORT_SERVER R:PORT_ATACANTE:IP_VICTIMA:PORT_VICTMA
    
![alt text](img/image-16.png)

![alt text](img/image-17.png)

Desde nuestra maquina atacante ya podemos acceder.

### port 3000

Revisamos el puerto 3000 accediendo asi **'127.0.0.1:3000'**. Desde el navegador podemos ver un par de **'endpoints'**.

![alt text](img/image-18.png)

Revisando los endpoint obtenemos la siguiente informacion.
- "telephony_pass": "St4yN0t1c3d_2026",
- "telephony_portal": "http://127.0.0.1:8080/ucp",
- "telephony_user": "FreePBXUCPTemplateCreator"

![alt text](img/image-19.png)

De la informacion obtenida destaca un nuevo puerto **'8080'** que no contemplamos, un usuario, una contraseña y un endpoint para el puerto 8080 **'/ucp'** .

Con chisel volvemos a hacer lo mismo que inicimos con el puerto 3000 y 9000.

![alt text](img/image-20.png)

### port 9000
Revisamos el puerto y vemos que no hay nada en **'/'**. 

![alt text](img/image-21.png)

Hacemos fuzzing para descubrir endpoints.

![alt text](img/image-22.png)

Obtenemos el endpoint **'health'**, revisamos su contenido.

![alt text](img/image-23.png)

La informacion mas relevante es el endpoint **'/jobs/export'** y tambien que necesitamos un **'automation key'** que no tenemos aun.

### port 8080
Revisamos el puerto con el endpoint que nos salio al revisar el puerto 3000.

![alt text](img/image-24.png)

Ingresamos a la credenciales que encontramos en el puerto 3000.

![alt text](img/image-25.png)

Al ingresar se ve asi. Ahora seguimos los siguientes pasos, hacemos click en el simbolo **'+'**.

![alt text](img/image-26.png)

Le damos un nombre y click en el boton **'create dashboard'**.

![alt text](img/image-27.png)

Luego hacemos click en el simbolo **'+'**.

![alt text](img/image-28.png)

Nos vamos a *'voicemail'* y hacemos click en el simbolo **'+'**.

![alt text](img/image-29.png)

Entonces podemos ver el **'automation key'** que no teniamos para el puerto 9000.

![alt text](img/image-30.png)

Accedemos a **'/jobs/report'** con metodo POST y vemos que necesita un **'report'**.

![alt text](img/image-31.png)

Asi que lo intentamos de nuevo, agregamos el *'report'* segun las instrucciones que obtuvimos del puerto 9000.

![alt text](img/image-32.png)

Notamos que el nombre que ingresamos se paso tal cual al resultado de la peticion.

![alt text](img/image-33.png)

Asi que intentamos agregar un **';'** para ver si podemos ejecutar comandos.

![alt text](img/image-34.png)

Al final usamos el *';'* dos veces y vemos que se ejecuta el comando id. 

Ahora nos enviamos una revershell.

![alt text](img/image-35.png)

## Respuesta a 'What is the root flag?'
Nos dirigimos a directorio /root y podemos ver la flag.

![alt text](img/image-36.png) 