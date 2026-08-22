![alt text](img/image.png)

# Contexto
Tiny packets. Odd hours. Suspiciously regular. Someone's smuggling out the data equivalent of a hotel towel every night, folded neatly inside traffic that looks ordinary until you decode it.

A short capture from the guest network is all VERA could pull before the connection dropped. Somewhere in that traffic, a quiet little errand is running on a loop, and it isn't part of any service the hotel actually offers.

# Preguntas
- What is the flag?

# Solucion
Descomprimimos el zip del reto y obtenemos un archivo **'traffic.pcapng'** que habriremos en **wireshark**.

## Wireshark
Al abrir el archivo se ve asi:

![alt text](img/image-1.png)

Primero determinamos que ip's tienen mas participacion en el archivo *'pcapng'*.

Nos dirigimos a *'Statistics -> Conversations '*

![alt text](img/image-2.png)

Nos dirigimos a la pestaña *'IPv4'*, ordenamos segun los *'packets'* y en este caso nos quedamos con las ip's **"192.168.1.141 y 34.41.103.191"**.

![alt text](img/image-3.png) 

Filtramos segun la ip **'34.41.103.191'**  para ver los paquetes en donde esta ip estuvo involucrada.
    
    ip.addr eq 34.41.103.191

![alt text](img/image-4.png)

De estos paquete revisamos que protocolos fueron mas usados, nos vamos a *'Statistics -> Protocol Hierarchy'*.

![alt text](img/image-5.png)

El protocolo que nos llama la atencion es **'Hypertext Transfer Protocol'** ya que rapidamente lo podemos a asociar a un servicio web, que usa html o algo similar.

![alt text](img/image-6.png)

Y agregamos otro filtro para el protocolo.

    ip.addr eq 34.41.103.191 && http 

![alt text](img/image-7.png)

Nos vamos a *'File -> Export Objects -> http...'*.

![alt text](img/image-8.png)

Entonces podemos ver los objetos http en los paquetes, el archivo que nos interesa es el **'updates.py'** y lo guardamos.

![alt text](img/image-9.png)

Puedes analizar el *'updates.py'* de cualquier forma que desees, lo importante es que es un script que simula un keylogger que cifra cada caracter y lo envia a una url atraves de las cookie en cada peticion.

![alt text](img/image-10.png)

Si revisamos los otros paquetes de wireshark y revisamos las cookies parecen ser caracteres sin sentido, pero gracias al archivo '*updates.py*' sabemos que es un caracter cifrado.

![alt text](img/image-19.png)

## Tshark

con tshark filtramos los paquetes para quedarnos con las cookies.

    tshark -r traffic.pcapng -Y "http.request" -T fields -e http.cookie 

![alt text](img/image-11.png)

Con la terminal filtramos las cadenas para quedarnos con las partes que nos interesa.

    tshark -r traffic.pcapng -Y "http.request" -T fields -e http.cookie | cut -d "_" -f3 | sed 's/state=//'  

![alt text](img/image-12.png)

## Respuesta a 'What is the flag?'

Segun el archivo *'updates.py'* el cifrado es en este orden 

- caracter normal
- cifrado xor
- cifrado base64
- caracter cifrado

Para el cifrado xor se usa la siguiente key.

![alt text](img/image-13.png)

Para realizar el proceso inverso se puede hacer con cyberchef o un script propio.

![alt text](img/image-14.png)

![alt text](img/image-15.png)

![alt text](img/image-16.png)

![alt text](img/image-17.png)

Pero el proceso es lento con cyberchef, asi que usare un script proporcionado por IA para obtener la flag.

![alt text](img/image-18.png)