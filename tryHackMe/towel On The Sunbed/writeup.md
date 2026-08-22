![alt text](img/image.png)
# Contexto
Ponzi found the resort's wellness portal running a little side project called Ponzi — a crypto rewards app, poolside edition. He set his towel down, claimed his daily reward, and went to reapply sunscreen. He came back to find the sunbed had been "claimed" three times over while he wasn't looking.

He's convinced the app owes him a spot in the Whale Vault. The app disagrees, politely, once every 24 hours. Somewhere between his request and the server's clock, there's a gap wide enough to walk a whale through.

# Preguntas
- What is the flag?

# Solucion
## nmap 
Hacemos escaneo de puertos abiertos a la maquina victima.

![alt text](img/image-1.png)

Verificamos servicios que corren en los puertos abiertos.

![alt text](img/image-2.png)

## web
Accedemos al servicio web en el '**puerto 3000**'

![alt text](img/image-3.png)

Se intento *'sql injection'* pero no tuvo efecto, asi que nos registramos.

![alt text](img/image-4.png)

Usamos la credenciales *'test:123456'*

![alt text](img/image-5.png)

Accedemos a *'/dashboard'*

![alt text](img/image-6.png)

En esta pagina tenemos un boton **'/Claim Reward'**.

![alt text](img/image-7.png)

Al oprimirlo se desactiva el boton y tenemos un tiempo de espera para volver a presionar el boton.

![alt text](img/image-8.png)

Intentamos de activar el boton modificando el html eliminando el atributo **'disabled'** del boton.

![alt text](img/image-10.png)

Pero me nos sale el siguiente mensaje *'Reward already claimed. Please wait before claiming again.'*

![alt text](img/image-11.png)

Intentamos la accion del boton con '**curl**'

![alt text](img/image-13.png)

Pero no obtenemos resultados...

Revisamos el *'depurado'* del navegador y vemos estas lineas del script **'dashboard.js'**

![alt text](img/image-9.png)

Estas lineas nos dicen que se verifica **data.canClaim** para permitir reclamar los *'PONZI'* el valor de *'canClaim'* se obtine de '**/dashboard/api/me**'.

Usamos **curl** para verificar la ruta '**/dashboard/api/me**'.

![alt text](img/image-14.png)

y vemos que el valor es **false**.

Despues de intentar buscar alguna manera de cambiar ese valor a *'true'* se intento otra cosas totalmente diferente. 

Se intento explotar este problema en entornos informaticos https://www.geeksforgeeks.org/operating-systems/race-condition-in-operating-systems/ en teoria.

### Caido
Interceptamos la peticion que se realiza al oprimir el boton.

![alt text](img/image-12.png)

Cerramos la session que creamos anteriormente para **crear otra cuenta** con las credenciales **'test3:123456'**

![alt text](img/image-15.png)

En aqui aun no oprimimos el boton.

![alt text](img/image-16.png)

Obtenemos la cookie de esta cuenta.

![alt text](img/image-17.png)

Enviamos la peticion interceptada al **'Automate'** de caido y **cambiamos** la *antigua cookie* con la *nueva cookie* en la peticion interceptada. 

En donde lo ponemos en **parallel**, type en **Null Payload** y enviamos 10

![alt text](img/image-18.png)
![alt text](img/image-19.png)

Recargamos la pagina.

![alt text](img/image-20.png)

## Respueta a 'What is the flag?'
Oprimimos el boton **'Open Vault'** y vemos la flag.
![alt text](img/image-21.png)