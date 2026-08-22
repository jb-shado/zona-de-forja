![alt text](img/image.png)

# Contexto
My Dearest Hacker,

The TryHeartMe shop is open for business. Can you find a way to purchase the hidden “Valenflag” item? 

# Preguntas
- What is the flag?

# Solucion
## Nmap
Escaneamos en busca de puertos abiertos.

![alt text](img/image-1.png)

Buscamos que servicios corren en los puertos abierto.

![alt text](img/image-2.png)

## WEB
Accedemos al servicio web por el puerto 5000.

![alt text](img/image-3.png)

Nos creamos una cuenta en el **'sign up'** con credenciales **'a@a.a:123456'** .

![alt text](img/image-4.png)

Seleccionamos un producto, vemos que no tenemos creditos y tenemos un rol. Ademas hay un boton para comprar **'buy'**

![alt text](img/image-5.png)


## Caido
Interceptamos las peticiones para ver como se gestiona la accion de compra.

![alt text](img/image-6.png)

Lo mas relevante es el metodo de peticion **'POST'**, el enpoint **'/buy/love-letter'** y la cookie que parece ser un **'JWT'**.

Analizamos el *'JWT'*.

![alt text](img/image-7.png)

Confirmamos que es un *'JWT'*, asi que intentamos un ataque que solo consiste en cambiar los valores de **'role'** y **'credits'**. Algunas paginas no verifican la autenticidad de la **'key'** y este es el caso.

Cambiamos el valor de role por **'admin'** y el valor de credits  por **'2000'**.

![alt text](img/image-8.png)

Copia el nuevo *'jwt'* que producimos y reemplazamos la cookie desde el navegador.

![alt text](img/image-9.png)

Recargamos la pagina y vemos un apartado de admin, tenemos mas creditos y vemos el producto **'valeflag'**.

![alt text](img/image-10.png)

seleccionamos el producto de nuestro interes y lo compramos.

![alt text](img/image-11.png)

## Respuesta a 'What is the flag?'
Oprimimos el boton de **'buy'** y obtenemos la flag.

![alt text](img/image-12.png)