# Deserializar Datos de Cuenta Personalizados
## Obejtivos de la Lección
*Al final de esta lección, podrás:*
- Explicar las cuentas derivadas de programas
- Derivar PDAs dadas semillas específicas
- Recuperar las cuentas de un programa
- Utilizar Borsh para deserializar datos personalizados
 
# Terminología 
- Las **direcciones derivadas de programas** , o PDAs, son direcciones que no tienen una llave privada correspondiente. El concepto de PDAs permite que los programas firmen las transacciones ellos mismos y permite almacenar y localizar datos.
- Puede derivar una PDA utilizando el método **findProgramAddress(seeds, programid)** .
- Puede obtener un arreglo de todas las cuentas pertenecientes a un programa utilizando **getProgramAccounts(programId)** .
- Los datos de la cuenta deben ser deserializados utilizando la misma estructura utilizada para almacenarlos en primer lugar. Puedes usar **@project-serum/borsh** para crear un esquema.

# Resumen
En la última lección, serializamos los datos de instrucción personalizados que fueron almacenados posteriormente en la cadena por un programa Solana. En esta lección, cubriremos con más detalle cómo los programas utilizan las cuentas, cómo recuperarlas y cómo deserializar los datos que almacenan.
## Programas
Como se dice, todo en Solana es una cuenta. Incluso los programas. Los programas son cuentas que almacenan código y están marcados como ejecutables. Este código puede ser ejecutado por el tiempo de ejecución de Solana cuando se le instruye para hacerlo.
Sin embargo, los programas en sí mismos son sin estado. No pueden modificar los datos dentro de su cuenta. Solo pueden persistir el estado almacenando datos en otras cuentas que pueden ser referenciadas en algún otro momento. Comprender cómo se utilizan estas cuentas y cómo encontrarlas es crucial para el desarrollo de cliente en Solana.

### PDA
PDA significa Dirección Derivada de Programa. Como su nombre sugiere, se refiere a una dirección (llave pública) derivada de un programa y algunas semillas. En una lección anterior, discutimos las llaves públicas/privadas y cómo se utilizan en Solana. A diferencia de un par de llaves, un PDA no tiene una llave privada correspondiente. El propósito de un PDA es crear una dirección que un programa pueda firmar de la misma manera en que un usuario puede firmar una transacción con su billetera.
Cuando envías una transacción a un programa y esperas que el programa actualice el estado o almacene datos de alguna manera, ese programa está utilizando uno o más PDAs. Esto es importante de entender al desarrollar cliente por dos razones:
1. Al enviar una transacción a un programa, el cliente debe incluir todas las direcciones de las cuentas que se escribirán o se leerán. Esto significa que a diferencia de las arquitecturas cliente-servidor más tradicionales, el cliente necesita tener conocimientos específicos de la implementación sobre el programa Solana. El cliente debe saber qué PDA se utilizará para almacenar datos para que pueda incluir esa dirección en la transacción.
2. De manera similar, al leer datos de un programa, el cliente debe saber desde qué cuenta(s) leer.

### Encontrando PDAs
Los PDAs técnicamente no se crean. Más bien, se encuentran o derivan en base a una o varias semillas de entrada.
Los pares de llaves regulares de Solana se encuentran en la curva ed2559 Elíptica. Esta función criptográfica asegura que cada punto a lo largo de la curva tiene un punto correspondiente en algún otro lugar de la curva, lo que permite las llaves públicas/privadas. Los PDAs son direcciones que no se encuentran en la curva ed2559 Elíptica y por lo tanto no pueden ser firmadas por una llave privada (ya que no hay una). Esto asegura que el programa es el único firmante válido para esa dirección.
Para encontrar una llave pública que no se encuentra en la curva ed2559, el ID del programa y las semillas de elección del desarrollador (como una cadena de texto) se pasan a través de la función **findProgramAddress (seeds, programid)** . Esta función combina el ID del programa, las semillas y una semilla de aumento en un buffer y lo pasa a un hash SHA256 para ver si la dirección resultante está en la curva. Si la dirección está en la curva (~50% de posibilidades de que lo esté), entonces la semilla de aumento se decrementa en 1 y se vuelve a calcular la dirección. La semilla de aumento comienza en 255 y progresivamente se iterate hacia abajo en **bump = 254** , **bump = 253** , etc. hasta que se encuentra una dirección con las semillas y el aumento dado que no se encuentra en la curva ed2559. La función **findProgramAddress** devuelve la dirección resultante y el aumento utilizado para sacarlo de la curva. De esta manera, la dirección se puede generar en cualquier lugar siempre y cuando tenga el aumento y las semillas.

Los PDAs son un concepto único y son una de las partes más difíciles del desarrollo en Solana de entender. Si no lo entiendes de inmediato, no te preocupes. Tendrá más sentido a medida que practiques más.

## ¿Por qué esto es importante?
La derivación de los PDAs es importante porque las semillas utilizadas para encontrar un PDA son las que utilizamos para localizar los datos. Por ejemplo, un programa simple que solo utiliza un único PDA para almacenar el estado global del programa podría utilizar una frase de semilla simple como "ESTADO_GLOBAL". Si el cliente quería leer datos de este PDA, podría derivar la dirección utilizando el ID del programa y esta misma semilla.

```JavaScript
const [pda, bump] = await findProgramAddress(Buffer.from("GLOBAL_STATE"), programId)
```

En programas más complejos que almacenan datos específicos del usuario, es común usar la llave pública del usuario como semilla. Esto separa los datos de cada usuario en su propio PDA. Esta separación permite al cliente localizar los datos de cada usuario encontrando la dirección utilizando el ID del programa y la llave pública del usuario.

```JavaScript
const [pda, bump] = await web3.PublicKey.findProgramAddress(
	[
		publicKey.toBuffer()
	],
	programId
)
```

Además, cuando hay varias cuentas por usuario, un programa puede usar una o varias semillas adicionales para crear e identificar cuentas. Por ejemplo, en una aplicación de toma de notas puede haber una cuenta por nota, donde cada PDA se deriva con la llave pública del usuario y el título de la nota.

```JavaScript
const [pda, bump] = await web3.PublicKey.findProgramAddress(
	[
		publicKey.toBuffer(),
		Buffer.from('First Note')
	],
	programId
)
```

## Obteniendo Múltiples Cuentas de Programa
Además de derivar direcciones, puedes obtener todas las cuentas creadas por un programa utilizando **connection.getProgramAccounts (programId)** . Esto devuelve una matriz de objetos donde cada objeto tiene una propiedad **pubkey** que representa la llave pública de la cuenta y una propiedad de **cuenta** de tipo **AccountInfo**. Puede utilizar la propiedad de **cuenta** para obtener los datos de la cuenta.

```JavaScript
const accounts = connection.getProgramAccounts(programId).then(accounts => {
	accounts.map(({ pubkey, account }) => {
		console.log('Account:', pubkey)
		console.log('Data buffer:', account.data)
	})
})
```


## Deserializando Datos de Cuenta Personalizados
La propiedad de **datos** en un objeto **AccountInfo** es un buffer. Para usarlo de manera eficiente, necesitarás escribir código que lo deserialice en algo más utilizable. Esto es similar al proceso de serialización que cubrimos en la última lección. Al igual que antes, utilizaremos **Borsh** y **@project-serum/borsh** . Si necesita repasar alguno de estos, eche un vistazo a la lección anterior.

La deserialización requiere conocer el diseño de la cuenta de antemano. Al crear sus propios programas, lo definirá como parte de ese proceso. Muchos programas también tienen documentación sobre cómo deserializar los datos de la cuenta. De lo contrario, si el código del programa está disponible, puede ver el código fuente y determinar la estructura de esa manera.

Para deserializar correctamente los datos de un programa en cadena, deberá crear un esquema del lado del cliente que refleje cómo se almacenan los datos en la cuenta. Por ejemplo, lo siguiente podría ser el esquema para una cuenta que almacena metadatos sobre un jugador en un juego en cadena.

```JavaScript
import * as borsh from "@project-serum/borsh";

borshAccountSchema = borsh.struct([
	borsh.bool('initialized'),
	borsh.u16('playerId'),
	borsh.str('name')
])
```

 Una vez que tenga su diseño definido, simplemente llame a **.decode(buffer)** en el esquema.

```JavaScript
import * as borsh from "@project-serum/borsh";

borshAccountSchema = borsh.struct([
	borsh.bool('initialized'),
	borsh.u16('playerId'),
	borsh.str('name')
])

const { playerId, name } = borshAccountSchema.decode(buffer)
```

# Demostracion
Practiquemos juntos continuando trabajando en la aplicación de revisión de películas de la última lección. No se preocupe si acaba de llegar a esta lección, debería ser posible seguir de cualquier manera.
Como recordatorio, este proyecto utiliza un programa Solana implementado en Devnet que permite a los usuarios revisar películas. En la última lección, agregamos funcionalidad al esqueleto de la interfaz frontal para que los usuarios pudieran enviar reseñas de películas, pero la lista de reseñas todavía muestra datos ficticios. Solucionemos eso recuperando las cuentas de almacenamiento del programa y deserializando los datos almacenados allí.

## 1. Descargue el Codigo Inicial
Si no completó la demostración de la lección anterior o simplemente quiere asegurarse de no haber perdido nada, puede descargar el [código inicial](https://github.com/Unboxed-Software/solana-movie-frontend/tree/solution-serialize-instruction-data).

El proyecto es una aplicación Next.js bastante simple. Incluye el **WalletContextProvider** que creamos en la lección de monederos, un componente **Card** para mostrar una revisión de película, un componente **MovieList** que muestra las reseñas en una lista, un componente **Form** para enviar una nueva reseña y un archivo **Movie.ts** que contiene una definición de clase para un objeto **Movie**.


Tenga en cuenta que al ejecutar **npm run dev** , las reseñas que se muestran en la página son falsas. Sustituiremos esas por las verdaderas.

## 2. Crear el Diseño del Búfer
Recuerde que para interactuar adecuadamente con un programa Solana, necesita saber cómo está estructurada su información.
El programa de revisión de películas crea una cuenta separada para cada revisión de película y almacena los siguientes datos en los **datos** de la cuenta:

1. **Inicializado** como un booleano que representa si la cuenta se ha inicializado o no.
2. **Calificación** como un entero sin signo de 8 bits que representa la calificación de 5 que el revisor dio a la película.
3. **Título** como una cadena que representa el título de la película revisada.
4. **Descripción** como una cadena que representa la parte escrita de la reseña.

Configuremos un esquema **borsh** en la clase **Movie** para representar el diseño de datos de la cuenta de películas. Comience importando **@project-serum/borsh** . A continuación, cree una propiedad estática **borshAccountSchema** y establezca el struct **borsh** apropiado que contenga las propiedades mencionadas anteriormente.

```JavaScript
import * as borsh from '@project-serum/borsh'

export class Movie {
	title: string;
	rating: number;
	description: string;

	...

	static borshAccountSchema = borsh.struct([
		borsh.bool('initialized'),
		borsh.u8('rating'),
		borsh.str('title'),
		borsh.str('description'),
	])
}
```

Recuerda, el orden aquí importa. Debe coincidir con cómo está estructurada la información de la cuenta.

## 3. Crear un Método para Deserializar Datos
Ahora que tenemos el diseño del buffer configurado, creemos un método estático en **Movie** llamado **deserialize** que tomará un **Buffer** opcional y devolverá un objeto **Movie** o **null**.

```JavaScript
import * as borsh from '@project-serum/borsh'

export class Movie {
	title: string;
	rating: number;
	description: string;

	...

	static borshAccountSchema = borsh.struct([
		borsh.bool('initialized'),
		borsh.u8('rating'),
		borsh.str('title'),
		borsh.str('description'),
	])

	static deserialize(buffer?: Buffer): Movie|null {
		if (!buffer) {
			return null
		}

		try {
			const { title, rating, description } = this.borshAccountSchema.decode(buffer)
			return new Movie(title, rating, description)
		} catch(error) {
			console.log('Deserialization error:', error)
			return null
		}
	}
}
```

El método primero verifica si el buffer existe y devuelve **null** si no lo hace. Luego, utiliza el diseño que creamos para decodificar el buffer, luego utiliza los datos para construir y devolver una instancia de **Movie** . Si la decodificación falla, el método registra el error y devuelve **null** .

## 4. Recuperar Cuentas de Reseñas de Películas
Ahora que tenemos una forma de deserializar los datos de la cuenta, necesitamos recuperar realmente las cuentas. Abra **MovieList.tsx** e importe **@solana/web3.js** . Luego, cree una nueva **conexión** dentro del componente **MovieList** . Finalmente, reemplace la línea **setMovies(Movie.mocks)** dentro de **useEffect** con una llamada a **connection.getProgramAccounts** . Tome el arreglo resultante y conviértalo en un arreglo de películas y llame a **setMovies** .

```JavaScript
import { Card } from './Card'
import { FC, useEffect, useState } from 'react'
import { Movie } from '../models/Movie'
import * as web3 from '@solana/web3.js'

const MOVIE_REVIEW_PROGRAM_ID = 'CenYq6bDRB7p73EjsPEpiYN7uveyPUTdXkDkgUduboaN'

export const MovieList: FC = () => {
	const connection = new web3.Connection(web3.clusterApiUrl('devnet'))
	const [movies, setMovies] = useState<Movie[]>([])

	useEffect(() => {
		connection.getProgramAccounts(new web3.PublicKey(MOVIE_REVIEW_PROGRAM_ID)).then(async (accounts) => {
			const movies: Movie[] = accounts.map(({ account }) => {
				return Movie.deserialize(account.data)
			})

			setMovies(movies)
		})
	}, [])

	return (
		<div>
			{
				movies.map((movie, i) => <Card key={i} movie={movie} /> )
			}
		</div>
	)
}
```

En este punto, ¡debería poder ejecutar la aplicación y ver la lista de reseñas de películas recuperadas del programa!
Dependiendo de cuántas reseñas se hayan enviado, esto puede tardar mucho tiempo en cargar o incluso bloquear completamente su navegador. Pero no se preocupe, en la próxima lección aprenderemos a dividir y filtrar las cuentas para poder ser más quirúrgico con lo que se carga.
Si necesita más tiempo con este proyecto para sentirse cómodo con estos conceptos, eche un vistazo al **código de solución** antes de continuar.