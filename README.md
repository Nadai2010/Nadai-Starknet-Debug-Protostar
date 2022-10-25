¡GRAPH[stark.png]

# Nadai Starknet Debugging de un Contrato Cairo con Protostar

En este tutorial explico como hacer el debugging a un contrato Cairo.

- ¿Tienes que hacerle un print a una variable para ver su valor? 
- ¿Tienes un contrato gigante y tenes un bug? 

Fuente oficial del [Tutorial](https://medium.com/@dpinoness/debugging-de-un-contrato-cairo-con-protostar-37183760b7e)

Fuente oficial de la [Repo en Github](https://github.com/dpinones/starknet-debug-protostar)

Clonar [Repo del Tutorial Nadai](https://github.com/Nadai2010/Nadai-Starknet-Debug-Protostar)

```bash
gh repo clone Nadai2010/Nadai-Starknet-Debug-Protostar
```

Usaremos una funcionalidad de Cairo llamada hints, que le permite inyectar python arbitrariamente en su código. El uso de hints está muy restringido en StarkNet y no se aplica en los contratos inteligentes. Pero es extremadamente útil para depurar su contrato.

## Como Hacer Debugging
Agregaremos anotaciones/pistas llamadas `hints`  en donde queramos hacer el debugging. En estos `hints` armaremos un `json` con los datos que queremos, y enviaremos una peticion a un server. Este server lo vamos a tener levantado de forma local y veremos las peticiones enviadas desde el contrato.

Una vez que terminamos de hacer el proceso de Debugging eliminamos los `hints`.

## Configuración
Deberemos instalar Protostar y también flasky requests para el server. Puede revisar la información oficial [Protostar](https://docs.swmansion.com/protostar/) y su [Repo de Github](https://github.com/software-mansion/protostar)

- Para instalar protostar usamos el siguiente comando.
```bash 
curl -L https://raw.githubusercontent.com/software-mansion/protostar/master/install.sh | bash
```

- Para instalar flask and requests
```bash
pip install flask requests
```

## Estructura del Proyecto
El proyecto debe de quedar con esta estrucutra.

¡GRAPH[estructura.png]

Vemos que tenemos un contrato y sus respectivos tests. Además tenemos el server donde veremos las peticiones enviadas por el contrato.

Repositorio del proyecto ()

### Contrato main
El contrato `main` tiene una `@storage_var` de nombre balance. También tiene la funcion de nombre `get_balance` que retorna el balance actual y tambien tiene la funcion de nombre `increase_balanc` para aumentar el balance. El balance inicia en 0.

- A la función “increase_value” le vamos a hacer debugging.

¡GRAPH[main.png]

## Test

- Test para la función `increase_value`.

¡GRAPH[test.png]

## Modificar la función `increase_value`
Agrego el `hint`, y armo un `json` con los datos que quiero enviar al server. En este caso quiero ver el balance actual y el nuevo balance.

¡GRAPH[debug.png]

## Correr el server
Levantamos el server en otra terminal y la dejamos abierta usando el siguiente código

```bash
python3 bigBrainDebug/server.py
```

¡GRAPH[server.png]

## Correr el test
Al ejecutar el comando `test` de protostar especifique que desea usar sugerencias no incluidas en la lista blanca de hints con la bandera `— disable-hint-validation`. Recorda que el uso de `hints` está muy restringido en StarkNet y no se aplica en los contratos inteligentes. Por lo tanto, los usaremos para hacer el debugging y luego de eso los eliminaremos.

- Ejecutar el test de nombre `test_increase_balance`.
```bash
protostar test ./tests/test_main.cairo::test_increase_balance — disable-hint-validation
```

## Resultados

- Como vemos en la terminal no da error ningun `test`.

¡GRAPH[test.png]


- Como vemos en la terminal del servidor se vizualizan el balance actual y el nuevo balance que es lo que queriamos. En el servidor como el test y `test_increase_balance` incrementa el balance en 3 oportunidades. Primero llama a la funcion con el valor `10` (entonces el balance es 10), luego con el valor `90` (entonces el valor es 100) y por último `50` entonces el balance final es `150`. Coincide con los valores mostrados en la terminal.

¡GRAPH[balance.png]


- Ahora realizaremos una prueba `ERRONEA`, modificando `test_main.cairo` con resultados incorrectos y veremos como nos ayudan los test en las interpretaciones.

¡GRAPH[testerror.png]


- Ejecutamos el test otra vez de `test_increase_balance`.
```bash
protostar test ./tests/test_main.cairo::test_increase_balance — disable-hint-validation
```

¡GRAPH[testerror1.png]


- Vemos que nos indica un error en cálculo de `150 !=140` y que el test está `failed`. Sin embargo en el server en los 3 últimos balance vemos los datos correctos que debemos tener.

¡GRAPH[balanceerror.png]


Así podemos ir haciendo nuestras pruebas y analizando los posibles `debug` que encontremos.


## Conclusión
Este tutorial está basado 100% en el repositorio [starknet-debug](https://github.com/starknet-edu/starknet-debug) de [starknet-edu](https://github.com/starknet-edu). En ese repositorio hay ejemplos usando el debugging en funciones recursivas y con structs.

La gran mayoría de veces no es necesario hacer el debugging de esta manera. Haciendo un test para una función particular y los assert correctos ya te alcanzaría. Con los test a veces no alcanza dadp que puedes perder el hilo al tener muchas funciones, muchas líneas de código y muchos llamados recursivos. El debugging de este tutorial puede ser útil en estos casos.

