#Escribir Datos a la Red de Solana
## Objetivos de la Lección
*Al final de esta lección, serás capaz de:*
 
- Explicar que es un par de llaves.
- Usar **@solana/web3.js** para generar un par de llaves.
- Use **@solana/web3.js** para crear un par de llaves usando una llave privada. 
- Explicar transacciones
- Explicar tarifas de transacción
- Usar **@solana/web3.js** para enviar sol 
- Usar **@solana/web3.js** para firmar transacciones
- Usar el explorador de bloques de Solana para ver transacciones. 


# Terminología

- Un **par de llaves** en Solana es una combinación de una llave pública y una llave privada. La llave pública se utiliza como una dirección para identificar una cuenta en la red de Solana, mientras que la llave privada (o secreta) se utiliza para verificar la identidad o la autoridad. Como su nombre indica, siempre debes mantener las llaves privadas en *secreto* . **@solana/web3.js** proporciona funciones de ayuda para crear una nueva pareja de llaves o para construir un par de llaves utilizando una llave privada existente. Es importante destacar que no debes compartir tu llave privada con nadie ya que esto podría dar acceso a tus fondos a una persona malintencionada.
- Las **transacciones** son esencialmente un paquete de instrucciones que invocan programas de Solana. El resultado de cada transacción depende del programa que esta siendo llamado. Todas las modificaciones de los datos en la cadena se realizan a través de transacciones. Por ejemplo, para transferir una cantidad de SOL de una cuenta a otra, se crea una transacción que especifica el origen y el destino de la transferencia, así como la cantidad de SOL a transferir.


# Resumen
## Par de Llaves
Como su nombre lo sugiere, un par de llaves  es la combinación de una llave pública y una llave privada. 

- La llave pública se utiliza como una "dirección" que apunta a una cuenta en la red de Solana. 
- La llave privada se utiliza para verificar la identidad o la autoridad. Es importante mantener siempre las llaves privadas en secreto. 

Un par de llaves es necesario para la gran mayoría de interacciones en la red de Solana. Si aún no tienes un par de llaves o si deseas generar uno nuevo para un propósito específico, **@solana/web3.js** proporciona una función de ayuda para crear un nuevo par de llaves.

Un par de llaves es del tipo de datos **Par de llaves** y se puede deconstruir en una llave pública: 

… o en una llave privada:

Si ya tienes un par de llaves que deseas utilizar, puedes crear un **Par de llaves** a partir de la llave privada utilizando la función **keypair.fromSecretKey()** . Para asegurarte de que tu llave privada permanezca segura, recomendamos inyectarla a través de una variable de entorno y no comprometer tu archivo **.env** . 

## Transacciones
Toda modificación de los datos en la cadena ocurre mediante transacciones enviadas a programas.

Las instrucciones de transacción contienen:
- Un identificador del programa al que deseas invocar.
- Un array de cuentas desde las cuales se leerán y/o escribirán.
- Datos estructurados como un array de bytes especificado para el programa invocado.

Cuando envías una transacción a un clúster de Solana, se invoca un programa de Solana con las instrucciones incluidas en la transacción.

Como puedes esperar, **@solana/web3.js** proporciona funciones de ayuda para crear transacciones e instrucciones. Puedes crear una nueva transacción con el constructor, **new Transaction()** . Una vez creada, puedes agregar instrucciones a la transacción con el método **add()** .

Las instrucciones pueden complicarse cuando se trabaja con programas personalizados. Afortunadamente, **@solana/web3.js** tiene funciones de comodidad para algunos de los programas nativos de Solana y operaciones básicas, como transferir SOL.

La función **SystemProgram.transfer()** requiere que se pase como parámetros:
- Una llave pública correspondiente a la cuenta del remitente
- Una llave pública correspondiente a la cuenta del destinatario
- La cantidad de SOL a enviar en lamports.

Esta función devuelve entonces la instrucción para enviar SOL del remitente al destinatario, después de lo cual la instrucción se puede agregar a la transacción.
Una vez creada, una transacción necesita ser enviada al clúster y confirmada.


La función **sendAndConfirmTransaction()** toma como parámetros:
- Una conexión al clúster
- Una transacción a enviar
- Una designación de par de llaves que representa a los firmantes de la transacción. En este ejemplo, el remitente es el único firmante.


## Instrucciones
El ejemplo de enviar SOL es una excelente introducción para enviar transacciones, pero gran parte del desarrollo web3 involucra llamar a programas no nativos. En el ejemplo anterior, la función **SystemProgram.transfer()** asegura que se pasen todos los datos necesarios para crear la instrucción, y luego crea la instrucción para usted. Sin embargo, al trabajar con programas no nativos, deberá ser muy específico al crear instrucciones estructuradas para que coincidan con el programa correspondiente.
Con **@solana/web3.js** , puede crear instrucciones no nativas con el constructor **TransactionInstruction** . Este constructor toma un solo argumento del tipo de datos **TransactionInstructionCtorFields** .

Según la definición anterior, el objeto pasado al constructor **TransactionInstruction** requiere:
- Un array de llaves de tipo **AccountMeta** , que representan las cuentas a las que afecta la instrucción.
- La llave pública del programa al que se llama, el cual ejecutará la instrucción
- Un **Buffer** opcional que contiene los datos para pasar al programa.

Ignoraremos el campo de **datos** por ahora y lo volveremos a visitar en una lección futura.

El campo **programId** es bastante autoexplicativo: es la llave pública asociada al programa. Necesitarás conocer esto con anticipación al llamar al programa de la misma manera en que necesitarías conocer la llave pública de alguien a quien quieres enviar SOL. 
El array de **llaves** requiere un poco más de explicación. Cada objeto en este arreglo representa una cuenta que se leerá o escribirá durante la ejecución de una transacción. Esto significa que necesita conocer el comportamiento del programa que está llamando y asegurarse de proporcionar todas las cuentas necesarias en el arreglo. 

Cada objeto en el arreglo de **llaves** debe incluir lo siguiente:
- **pubkey** : la llave pública de la cuenta
- **isSigner** : un booleano que representa si la cuenta es una firma en la transacción
- **isWritable** : un booleano que representa si la cuenta se escribe durante la ejecución de la transacción.

Juntando todo esto, podríamos obtener algo como lo siguiente:

## Tarifas de Transacción
Las tarifas de transacción están incorporadas en la economía de Solana como compensación para la red de validadores por los recursos de CPU y GPU requeridos en el procesamiento de transacciones. A diferencia de muchas redes que tienen un mercado de tarifas donde los usuarios pueden pagar tarifas más altas para aumentar sus posibilidades de ser incluidos en el próximo bloque, las tarifas de transacción de Solana son deterministas.
El primer firmante incluido en el arreglo de firmantes en una transacción es responsable de pagar la tarifa de transacción. Si este firmante no tiene suficientes SOL en su cuenta para cubrir la tarifa de transacción, la transacción será descartada.
Al probar, ya sea local o en devnet, puede usar el comando Solana CLI **solana airdrop 1** para obtener SOL de prueba gratis en su cuenta para pagar las tarifas de transacción.


## Explorador de Solana
Todas las transacciones en la cadena de bloques son visibles públicamente en el **explorador de bloques de Solana** . Por ejemplo, puede tomar la firma devuelta por **sendAndConfirmTransaction()** en el ejemplo anterior, buscar esa firma en el Explorador de Bloques de Solana, y ver:
- cuándo ocurrió
- en qué bloque fue incluido
- la tarifa de transacción
- y más!


# Demostración
Vamos a crear un script para hacer ping a un programa simple que incrementa un contador cada vez que se le hace ping. Este programa existe en la red de desarrollo de Solana en la dirección **ChT1B39WKLS8qUrkLvFDXMhEJ4F1XZzwUNHUt4AU9aVa** . El programa almacena los datos de conteo en una cuenta específica en la dirección **Ah9K7dQ8EHaZqcAsgBW8w37yN2eAy3koFmUn4x3CJtod** .

## 1. Esquema básico
Empecemos con un esquema básico. Puedes configurar tu proyecto de la manera que te resulte más apropiada, pero utilizaremos un sencillo proyecto de Typescript con una dependencia del paquete @solana/web3.js. Si deseas utilizar nuestro esquema, puedes usar los siguientes comandos en la línea de comandos:

Este proceso será para:
1. Creará un nuevo directorio para el proyecto con un subdirectorio **src**
2. Moverá el indicador de comando dentro del directorio del proyecto
3. Creará un archivo **index.ts** dentro de **src**
4. Inicializará un repositorio git con un archivo **.gitignore**
5. Creará un nuevo paquete **npm**
6. Agregará una dependencia de desarrollador para typescript
7. Agregará una dependencia de desarrollador para **ts-node**
8. Creará un archivo **.tsconfig**
9. Instalará la dependencia **@solana/web3.js**
10. Instalará la dependencia **.dotenv**
11. Creará un archivo **.env**

Si deseas que tu código coincida exactamente con el nuestro, reemplaza el contenido del archivo **tsconfig.json** con el siguiente:


Añade lo siguiente al archivo **.gitignore** :

Y finalmente, añade lo siguiente al objeto **scripts** en el archivo **package.json** :

## 2. Generar un Par de Llaves nuevo
Antes de poder hacer cualquier cosa, necesitarás un par de llaves. Vamos a generar uno en el archivo **index.ts** :

La mayor parte de este código es solo un esqueleto para ejecutar el archivo correctamente. Las líneas dentro de la función **main()** generan un nuevo par de llaves y muestran la llave privada en la consola.
Ejecuta **npm start** después de guardar este archivo y deberías ver una matriz de números impresa en la consola. Esta matriz representa la llave privada de tu nuevo par de llaves. **No utilices** este par de llaves para operaciones en Mainnet. **Utiliza solo este par de llaves para las pruebas** .
Copia la matriz de la llave privada del registro de la consola y pégala en el archivo **.env** como una variable de entorno llamada **PRIVATE_KEY** . De esta manera, podremos reutilizar este par de llaves en el desarrollo futuro en lugar de generar un nuevo par de llaves cada vez que ejecutemos algo. Debería verse algo así, pero con números diferentes:


## 3. Inicializa el Par de Llaves desde la llave privada
Ahora que hemos generado con éxito un par de llaves y lo hemos copiado en el archivo **.env** , podemos eliminar el código dentro de la función **main()** .

Regresaremos a la función **main()** pronto, pero por ahora creemos una nueva función fuera de **main()** llamada **initializePar de llaves()** . Dentro de esta nueva función:

1. Parsear la variable de entorno **PRIVATE_KEY** como **number[]**
2. Utilizarlo para inicializar un **Uint8Array**
3. Inicializar y devolver un **Par de llaves** utilizando ese **Uint8Array** .


## 4. Programa Ping
Ahora que tenemos una manera de inicializar nuestro par de llaves, necesitamos establecer una conexión con el Devnet de Solana. En **main()** , invoquemos **initializePar** de llaves() y creamos una conexión:

Ahora crea una función async fuera de **main()** llamada **pingProgram()** con dos parámetros que requieren una conexión y un par de llaves del pagador como argumentos:

Dentro de esta función, necesitamos:
1. Crear una transacción
2. Crear una instrucción
3. Agregar la instrucción a la transacción
4. Enviar la transacción.

Recuerda, la pieza más desafiante aquí es incluir la información correcta en la instrucción. Sabemos la dirección del programa al que estamos llamando. También sabemos que el programa escribe datos en una cuenta separada cuyo dirección también tenemos. Agreguemos las versiones de cadena de ambas como constantes en la parte superior del archivo **index.ts** :


Ahora, en la función **pingProgram()** , creemos una nueva transacción, luego inicializamos una **llave pública** para la cuenta del programa y otra para la cuenta de datos.

A continuación, creamos la instrucción. Recuerda, la instrucción debe incluir la llave pública del programa y también debe incluir una matriz con todas las cuentas que se leerán o escribirán. En este ejemplo de programa, solo se necesita la cuenta de datos mencionada anteriormente.

A continuación, agregaremos la instrucción a la transacción que creamos al comienzo de la función. Luego, llamaremos a la función **sendAndConfirmTransaction()** pasando la conexión, la transacción y el pagador. Finalmente, registraremos el resultado de esa llamada a la función para que podamos buscarlo en el Solana Explorer.

Finalmente, invocaremos la función **pingProgram()** dentro de **main()** utilizando la **conexión** y el **pagador** :


## 5. Airdrop
Ahora ejecuta el código con **npm start** y mira si funciona. Es posible que te encuentres con el siguiente error en la consola:
*Este mensaje de error indica que la simulación de transacción falla porque intenta debitar (retirar) fondos de una cuenta que no tiene registro de un crédito previo* .
Si obtienes este error, es porque tu par de llaves es completamente nuevo y no tiene ningún SOL para cubrir las tarifas de transacción. Vamos a solucionar esto agregando la siguiente línea en **main()** antes de la llamada a **pingProgram()** :

Este proceso depositará 1 SOL en su cuenta, el cual podrá utilizar para realizar pruebas. Sin embargo, esto no funcionará en Mainnet, donde tendría un valor real. Pero es muy conveniente para probar localmente y en Devnet.

## 6. Valida el Explorador de Solana
Ahora ejecuta el código de nuevo. Puede tomar un momento o dos, pero ahora el código debería funcionar y deberías ver una cadena larga impresa en la consola, como la siguiente:

Copia esta firma de confirmación. Abre un navegador y ve a **https://explorer.solana.com/?cluster=devnet** (el parámetro de consulta al final de la URL asegurará que explorarás las transacciones en Devnet en lugar de Mainnet). Pega la firma en la barra de búsqueda en la parte superior del explorador de Devnet de Solana y presiona enter. Deberías ver todos los detalles sobre la transacción. Si desplazas todo el camino hasta abajo, verás los Registros de programas, que muestran cuántas veces se ha **llamado al programa**, incluyendo tu llamada.

Si deseas hacer más fácil mirar las transacciones en Solana Explorer en el futuro, simplemente cambia tu **console.log** en **pingProgram()** por lo siguiente:

¡Y así de fácil estás llamando programas en la red Solana y escribiendo datos en la cadena! 

En las próximas lecciones aprenderás cómo:
1. hacer esto de forma segura desde el navegador en lugar de desde la ejecución de un script, 
2. agregar datos personalizados a tus instrucciones 
3. deserializar datos de la cadena.
 
# Reto 
Continúe y creé un script desde cero que le permitirá transferir SOL de una cuenta a otra en Devnet. Asegúrese de imprimir la firma de la transacción para poder verla en Solana Explorer.
Si te quedas atascado, no dudes en echar un vistazo al **código de la solución** .



# Interactuar con Billeteras (Wallets)
## Objetivos de la Lección
*Al final de esta lección, podrás:*
• Explicar los monederos (wallets)
• Instalar la extensión Phantom
• Configurar el monedero Phantom en **Devnet**
• Utilizar el adaptador de monederos para que los usuarios firmen transacciones.
 
# Terminología
- Los **monederos** almacenan su llave privada y manejan la firma segura de transacciones
- Los **monederos de hardware** almacenan su llave privada en un dispositivo separado
- Los **monederos de software** utilizan su computadora para almacenar de forma segura
- Los monederos de software a menudo son **extensiones de navegador** que facilitan la conexión a sitios web
- La **biblioteca Wallet-Adapter** de Solana simplifica el soporte de las extensiones de monederos de navegador, lo que le permite construir sitios web que puedan solicitar la dirección de monedero de un usuario y proponer transacciones para que las firmen.


# Resumen
## Wallets
En las dos lecciones anteriores discutimos las llaves de par. Las llaves de par se utilizan para localizar las cuentas y firmar transacciones. Si bien la llave pública de un par de llaves es perfectamente segura para compartir, la llave privada siempre debe mantenerse en un lugar seguro. Si la llave privada de un usuario se expone, entonces un actor malicioso podría vaciar su cuenta de todos los activos y ejecutar transacciones con la autoridad de ese usuario.
Un "monedero" se refiere a cualquier cosa que almacene una llave privada con el fin de mantenerla segura. Estas opciones de almacenamiento seguro se pueden describir generalmente como "monederos de hardware" o "monederos de software". Los monederos de hardware son dispositivos de almacenamiento que están separados de su computadora. Los monederos de software son aplicaciones que puedes instalar en tus dispositivos existentes.
Los monederos de software a menudo vienen en forma de una extensión de navegador. Esto hace posible que los sitios web interactúen fácilmente con el monedero. Estas interacciones suelen estar limitadas a:

1. Ver la llave pública (dirección) del monedero
2. Enviar transacciones para la aprobación del usuario
3. nviar una transacción aprobada a la red.

Una vez que se envía una transacción, el usuario final puede "confirmar" la transacción y enviarla a la red con su "firma". 
Firmar transacciones requiere el uso de su llave privada. Al permitir que un sitio envíe una transacción a su monedero y que el monedero maneje la firma, asegura que nunca exponga su llave privada al sitio web. En cambio, solo comparte la llave privada con la aplicación del monedero.
A menos que estés creando una aplicación de monedero tú mismo, tu código nunca debería necesitar pedirle a un usuario su llave privada. En cambio, puedes pedirle a los usuarios que se conecten a tu sitio utilizando un monedero confiable.

## Phantom Wallet
**Phantom** es una billetera de software ampliamente utilizada en el ecosistema Solana. Soporta navegadores populares y tiene una aplicación móvil para conectarse en cualquier momento. Se recomienda soportar varias billeteras para las aplicaciones descentralizadas, pero este curso se centra específicamente en Phantom.

## Adaptador d Wallets de Solana
El adaptador de billeteras de Solana es un conjunto de bibliotecas que se pueden utilizar para simplificar el proceso de soporte para las extensiones de navegador de billeteras.
El adaptador de billeteras de Solana consta de varios paquetes modulares. La funcionalidad principal se encuentra en **@solana/wallet-adapter-base** y **@solana/wallet-adapter-react** . 
También hay paquetes que proporcionan componentes para marcos de interfaz de usuario comunes. En esta lección y en todo este curso, estaremos utilizando componentes de **@solana/wallet-adapter-react-ui** . 
Finalmente, hay paquetes que son adaptadores para billeteras específicas, incluyendo Phantom. Puedes usar **@solana/wallet-adapter-wallets** para incluir todas las billeteras soportadas o puedes elegir un paquete de billetera específico como **@solana/wallet-adapter-phantom** .

### Instalar Bibliotecas de Adaptador de Wallet 
Al agregar soporte para billetera a una aplicación react existente, comienzas instalando los paquetes apropiados. Necesitarás **@solana/wallet-adapter-base** , **@solana/wallet-adapter-react** , el paquete (s) para las billeteras que deseas admitir y **@solana/wallet-adapter-react-ui** si planeas utilizar los componentes de react proporcionados, por ejemplo.

## Conectarse a Wallets
**@solana/wallet-adapter-react** nos permite persistir y acceder a los estados de conexión de la billetera a través de ganchos y proveedores de contexto, a saber:
- **useWallet**
- **WalletProvider**
- **useConnection**
- **ConnectionProvider**
Para que estos funcionen correctamente, cualquier uso de **useWallet** y **useConnection** debería estar envuelto en **WalletProvider** y **ConnectionProvider** . Una de las mejores formas de asegurar esto es envolver toda tu aplicación en **ConnectionProvider** y **WalletProvider** .

Ten en cuenta que **ConnectionProvider** requiere una propiedad **endpoint** y que **WalletProvider** requiere una propiedad **wallets** . Seguimos utilizando el endpoint para el clúster Devnet y, por ahora, solo estamos utilizando **PhantomWalletAdapter** para las **billeteras** .

En este punto, puedes conectarte con **wallet.connect()** , lo que efectivamente instruirá a la billetera para solicitar al usuario permiso para ver su llave pública y solicitar la aprobación para las transacciones.
 
Aunque podrías hacerlo en un gancho **useEffect** , normalmente desearás proporcionar una funcionalidad más sofisticada. Por ejemplo, es posible que desees que los usuarios puedan elegir de una lista de billeteras admitidas o desconectarse después de haberse conectado.
 
## **@solana/wallet-adapter-react-ui**
Puedes crear componentes personalizados para esto o puedes aprovechar los componentes proporcionados por **@solana/wallet-adapter-react-ui** . La forma más sencilla de proporcionar opciones extensas es usar **WalletModalProvider** y **WalletMultiButton** :

El **WalletModalProvider** agrega funcionalidad para presentar una pantalla modal para que los usuarios puedan seleccionar qué billetera desean usar. El **WalletMultiButton** cambia el comportamiento para que coincida con el estado de conexión:

 También puedes usar componentes más granulares si necesitas una funcionalidad más específica:
- **WalletConnectButton**
- **WalletModal**
- **WalletModalButton**
- **WalletDisconnectButton**
- **WalletIcon**

## Accede a la Información de la Cuenta
Una vez que tu sitio esté conectado a una billetera, **useConnection** recuperará un objeto **Connection** y **useWallet** obtendrá el **WalletContextState** . **WalletContextState** tiene una propiedad **publicKey** que es **nula** cuando no está conectada a una billetera y tiene la llave pública de la cuenta del usuario cuando se conecta una billetera. Con una llave pública y una conexión, puedes recuperar información de la cuenta y más.

## Enviar Transacciones
**WalletContextState** también proporciona una función **sendTransaction** que puedes usar para enviar transacciones para su aprobación.

Cuando se llama a esta función, la billetera conectada mostrará la transacción para la aprobación del usuario. Si es aprobada, entonces se enviará la transacción.

# Demostración
Vamos a tomar el programa Ping de la última lección y construye una interfaz que permita a los usuarios aprobar una transacción que envía un ping al programa. Como recordatorio, la llave pública del programa es: **ChT1B39WKLS8qUrkLvFDXMhEJ4F1XZzwUNHUt4AU9aVa** y la llave pública de la cuenta de datos es **Ah9K7dQ8EHaZqcAsgBW8w37yN2eAy3koFmUn4x3CJtod** .

## 1. Descarga la extensión del Navegador Phantom, y configúrala para DevNet
Si aún no lo tienes, descarga la **extensión del navegador Phantom** . En el momento de escribir esto, admite los navegadores Chrome, Brave, Firefox y Edge, por lo que también necesitarás tener uno de esos navegadores instalado. Sigue las instrucciones de Phantom para crear una nueva cuenta y una nueva billetera.
Una vez que tengas una billetera, haz clic en el engranaje de configuración en la parte inferior derecha en la interfaz de usuario de Phantom. Desplázate hacia abajo y haz clic en el elemento "Cambiar red" y selecciona "Devnet". Esto garantiza que Phantom estará conectado a la misma red que usaremos en esta demostración.

## 2. Descarga el Código Inicial
Descarga el código inicial para este proyecto **aquí** . Este proyecto es una simple aplicación Next.js. Está en su mayoría vacía excepto por el componente **AppBar** . Construiremos el resto a lo largo de esta demostración.

Puedes ver su estado actual con el comando **npm run dev** en la consola.

## 3. Envuelve la aplicación en los Proveedores de Contexto
Para empezar, vamos a crear un nuevo componente para contener los diversos proveedores de Wallet-Adapter que usaremos. Crea un nuevo archivo dentro de la carpeta de **componentes** llamado **WalletContextProvider.tsx** .

Empecemos con algo de la plantilla de un componente funcional:

Para conectarse correctamente a la billetera del usuario, necesitaremos un **ConnectionProvider** , **WalletProvider**  y **WalletModalProvider** . Comience importando estos componentes de **@solana/wallet-adapter-react** y **@solana/wallet-adapter-react-ui** . Luego, agrégalos al componente **WalletContextProvider** . Tenga en cuenta que **ConnectionProvider** requiere un parámetro **endpoint** y **WalletProvider** requiere un array de **billeteras** . Por ahora, solo usa una cadena vacía y una array vacía, respectivamente.

Lo último que necesitamos es un endpoint actual para **ConnectionProvider** y las billeteras admitidas para **WalletProvider** .
Para el endpoint, utilizaremos la misma función **clusterApiUrl** de la biblioteca **@solana/web3.js** que hemos utilizado antes, por lo que necesitarás importarla. Para la matriz de billeteras, también necesitarás importar la biblioteca **@solana/wallet-adapter-wallets** .

Después de importar estas bibliotecas, crea una constante **endpoint** que utilice la función **clusterApiUrl** para obtener la url de Devnet. Luego, crea una constante **wallets** y establece una matriz que contenga un **PhantomWalletAdapter** recién construido. Finalmente, reemplaza la cadena vacía y la matriz vacía en **ConnectionProvider** y **WalletProvider** , respectivamente.
Para completar este componente, agrega **require('@solana/wallet-adapter-react-ui/styles.css')** ; debajo de tus importaciones para garantizar un estilo y comportamiento adecuado de los componentes de la biblioteca Wallet Adapter.


## 4. Agregar Wallet Multi-botón
A continuación, configuraremos el botón Conectar. El botón actual es solo un marcador de posición porque, en lugar de usar un botón estándar o crear un componente personalizado, utilizaremos el "multi-botón" de Wallet-Adapter. Este botón se comunica con los proveedores que configuramos en **WalletContextProvider** y permite a los usuarios elegir una billetera, conectarse a una billetera y desconectarse de una billetera. Si alguna vez necesitas más funcionalidad personalizada, puedes crear un componente personalizado para manejar esto.
Antes de agregar el "multi-botón", debemos envolver la aplicación en el **WalletContextProvider** . Hazlo importándolo en **index.tsx** y agregándolo después de la etiqueta **</Head>** de cierre:

Si ejecutas la aplicación, todo debería seguir siendo igual ya que el botón actual en la esquina superior derecha sigue siendo solo un marcador de posición. Para remediar esto, abre **AppBar.tsx** y reemplaza **<button>Connect</button>** con **<WalletMultiButton/>** . Necesitarás importar **WalletMultiButton** desde **@solana/wallet-adapter-react-ui** .

En este punto, deberías poder ejecutar la aplicación e interactuar con el botón multi en la esquina superior derecha de la pantalla. Ahora debería leer "Select Wallet" (Seleccionar billetera). Si tienes la extensión Phantom y estás conectado, deberías poder conectar tu billetera Phantom al sitio utilizando este nuevo botón.

## 5. Crear un botón para hacer Ping al Programa
¡Ahora que nuestra aplicación puede conectarse a la billetera Phantom, hagamos que el botón Ping! realmente haga algo.

Comience abriendo el archivo **PingButton.tsx** . Vamos a reemplazar el **console.log** dentro de **onClick** con código que creará una transacción y la enviará a la extensión Phantom para la aprobación del usuario final.
Primero, necesitamos una conexión, la llave pública de la billetera y la función **sendTransaction** de Wallet-Adapter. Para obtener esto, necesitamos importar **useConnection** y **useWallet** de **@solana/wallet-adapter-react** . Mientras estemos aquí, también importemos **@solana/web3.js** ya que lo necesitaremos para crear nuestra transacción.

Ahora utiliza el gancho **useConnection** para crear una constante de **conexión** y el gancho **useWallet** para crear constantes **publicKey** y **sendTransaction** .

Con eso, podemos completar el cuerpo de **onClick** .
Primero, comprueba que tanto la **conexión** como **publicKey** existen (si alguno de ellos no existe, la billetera del usuario aún no está conectada).
A continuación, construye dos instancias de **PublicKey** , una para el ID del programa **ChT1B39WKLS8qUrkLvFDXMhEJ4F1XZzwUNHUt4AU9aVa** y otra para la cuenta de datos **Ah9K7dQ8EHaZqcAsgBW8w37yN2eAy3koFmUn4x3CJtod** .
A continuación, construye una **Transacción**, luego una nueva **instrucción de transacción** que incluye la cuenta de datos como una llave escribible.
A continuación, agrega la instrucción a la transacción.
Finalmente, llama a **sendTransaction** .

¡Y eso es todo! Si actualizas la página, conectas tu billetera y haces clic en el botón ping, Phantom debería mostrarte una ventana emergente para confirmar la transacción.

## 6. Añade algunos detalles finos
Hay muchas cosas que podrías hacer para mejorar aún más la experiencia del usuario aquí. Por ejemplo, podrías cambiar la interfaz de usuario para que solo te muestre el botón Ping cuando esté conectada una billetera y muestra algún otro aviso en caso contrario. Podrías vincular a la transacción en Solana Explorer después de que un usuario confirme una transacción para que puedan ver fácilmente los detalles de la transacción. Cuanto más experimentes con ello, más cómodo te sentirás, ¡así que sé creativo!
Si necesitas pasar algún tiempo mirando el código fuente completo de esta demostración para entender todo esto en contexto, echa un vistazo **aquí** .
 
# Reto 
Ahora es tu turno de construir algo de forma independiente. Crea una aplicación que permita a un usuario conectar su billetera Phantom y enviar SOL a otra cuenta.
1. Puedes construir esto desde cero o descargar el código de inicio aquí.
2. Envuelve la aplicación de inicio en los proveedores de contexto apropiados.
3. En el componente de formulario, configura la transacción y envíala a la billetera del usuario para su aprobación.
4. Sé creativo con la experiencia de usuario. ¡Agrega un enlace para que el usuario pueda ver la transacción en Solana Explorer o algo más que te parezca genial!

Si te atascas realmente, no dudes en revisar el código de solución aquí.
 

#Serializar Datos con Instrucciones Personalizadas 
##Objetivos de la Lección
*Al final de esta lección, podrás:*
- Explicar los contenidos de una transacción
- Explicar las instrucciones de transacción
- Explicar los conceptos básicos de las optimizaciones de tiempo de ejecución de Solana
- Explicar Borsh
- Usar Borsh para serializar datos de instrucciones personalizadas

# Terminología 
- Las transacciones están compuestas por una serie de instrucciones, una sola transacción puede tener cualquier número de instrucciones, cada una dirigida a su propio programa. Cuando se envía una transacción, el tiempo de ejecución de Solana procesará sus instrucciones en orden y de manera atómica, lo que significa que si alguna de las instrucciones falla por cualquier razón, la transacción completa fallará al ser procesada.
- Cada instrucción está compuesta por 3 componentes: el ID del programa destinado, una matriz de todas las cuentas involucradas y un búfer de datos de instrucción.
- Cada transacción contiene: una matriz de todas las cuentas de las que pretende leer o escribir, una o más instrucciones, un blockhash reciente y una o más firmas.
- Para pasar los datos de la instrucción desde un cliente, debe ser serializado en un búfer de bytes. Para facilitar este proceso de serialización, usaremos Borsh.
- Las transacciones pueden fallar al ser procesadas por la cadena de bloques por cualquier número de razones, discutiremos algunas de las más comunes aquí.
 
#Resumen
## Transacciones
Las transacciones son cómo enviamos información a la cadena de bloques para que sea procesada. Hasta ahora, hemos aprendido a crear transacciones muy básicas con funcionalidad limitada. Pero las transacciones y los programas a los que se envían pueden ser diseñados para ser mucho más flexibles y manejar mucha más complejidad de lo que hemos tratado hasta ahora.

### Contenidos de las Transacciones
Cada transacción contiene:
- Un array que incluye cada cuenta de la que se intenta leer o escribir
- Una o más instrucciones
- Un hash de bloque reciente
- Una o más firmas
**@solana/web3.js** simplifica este proceso para que solo tengas que enfocarte en agregar instrucciones y firmas. La biblioteca construye el arreglo de cuentas basado en esa información y maneja la lógica para incluir un hash de bloque reciente.

## Instrucciones
Cada instrucción contiene:
- La ID del programa (llave pública) al que se destina
- Un arreglo que lista cada cuenta de la que se leerá o escribirá durante la ejecución
- Un búfer de bytes de datos de instrucción

La identificación del programa mediante su llave pública asegura que la instrucción se lleve a cabo por el programa correcto.
Incluir una matriz de cada cuenta que se leerá o escribirá permite a la red realizar una serie de optimizaciones que permiten una alta carga de transacciones y una ejecución más rápida.
El búfer de bytes le permite pasar datos externos a un programa.
Puede incluir varias instrucciones en una sola transacción. El tiempo de ejecución de Solana procesará estas instrucciones en orden y de forma atómica. En otras palabras, si cada instrucción tiene éxito, entonces la transacción en su conjunto tendrá éxito, pero si una sola instrucción falla, entonces toda la transacción fallará inmediatamente sin efectos secundarios.
Una nota sobre la matriz de cuentas y la optimización:
No es solo una matriz de las llaves públicas de las cuentas. Cada objeto en la matriz incluye la llave pública de la cuenta, si es o no una firma en la transacción y si es o no escribible. Incluir si una cuenta es escribible durante la ejecución de una instrucción permite al tiempo de ejecución facilitar el procesamiento paralelo de los contratos inteligentes. Debido a que debe definir qué cuentas son de solo lectura y a las que escribirá, el tiempo de ejecución puede determinar qué transacciones son no solapadas o de solo lectura y permitir que se ejecuten de forma concurrente. Para obtener más información sobre el tiempo de ejecución de Solana, consulte esta **publicación de blog** .


### Datos de Instrucción
La capacidad de agregar datos arbitrarios a una instrucción asegura que los programas pueden ser lo suficientemente dinámicos y flexibles para un amplio rango de usos, al igual que el cuerpo de una solicitud HTTP permite crear APIs REST dinámicas y flexibles.
Al igual que la estructura del cuerpo de una solicitud HTTP depende del punto final al que desea llamar, la estructura del búfer de bytes utilizado como datos de instrucción depende completamente del programa receptor. Si está construyendo una aplicación completa de una sola vez, deberá copiar la misma estructura que utilizó al construir el programa en el código del lado del cliente. Si trabaja con otro desarrollador que se encarga del desarrollo del programa, puede coordinar para garantizar una disposición de búfer coincidente.
Imaginemos un ejemplo concreto. Imagina trabajar en un juego Web3 y ser responsable de escribir el código del lado del cliente que interactúa con un programa de inventario de jugadores. El programa fue diseñado para permitir al cliente:
- Agregar inventario en función de los resultados de juego del jugador
- Transferir inventario de un jugador a otro
- Equipar a un jugador con elementos de inventario seleccionados
Este programa se estructuraría de tal manera que cada uno de ellos esté encapsulado en su propia función. 
Sin embargo, cada programa solo tiene un punto de entrada. Le indicaría al programa en qué de estas funciones ejecutar mediante los datos de instrucción.
También incluiría en los datos de instrucción cualquier información que la función necesite para ejecutarse correctamente, por ejemplo, el ID de un elemento de inventario, un jugador para transferir inventario, etc. 
Exactamente cómo se estructuraría esta información dependería de cómo se escribió el programa, pero es común que el primer campo en los datos de instrucción sea un número que el programa pueda asociar a una función, después de lo cual los campos adicionales actúan como argumentos de función.
 
## Serialización
Además de saber qué información incluir en un búfer de datos de instrucción, también necesita serializarlo correctamente. El serializador más común utilizado en Solana es **Borsh** . Según el sitio web:

*”Borsh significa Binary Object Representation Serializer for Hashing (Serializador de representación de objeto binario para la generación de huellas digitales). Se destina a ser utilizado en proyectos críticos para la seguridad ya que prioriza la consistencia, la seguridad y la velocidad, y viene con una especificación estricta.”*

Borsh mantiene una **biblioteca JS** que maneja la serialización de tipos comunes en un búfer. También hay otros paquetes construidos sobre borsh que intentan hacer este proceso aún más fácil. Utilizaremos la biblioteca **@project-serum/borsh** que se puede instalar mediante **npm** .
A partir del ejemplo anterior de inventario de juegos, veamos un escenario hipotético en el que estamos instruyendo al programa para equipar a un jugador con un determinado elemento. Asuma que el programa está diseñado para aceptar un búfer que representa una estructura con las siguientes propiedades:
 1. **variante** como un entero sin signo de 8 bits que indica al programa qué instrucción o función ejecutar.
2. **playerId** como un entero sin signo de 16 bits que representa el ID del jugador que se equipará con el elemento dado.
3. **itemId** como un entero sin signo de 256 bits que representa el ID del elemento que se equipará al jugador dado.
Todo esto se pasará como un búfer de bytes que se leerá en orden, por lo que es crucial asegurar el orden correcto de la disposición del búfer. Crearía el esquema o plantilla de disposición de búfer para lo anterior de la siguiente manera:

Luego, puede codificar los datos utilizando este esquema con el método de **codificación**. Este método acepta como argumentos un objeto que representa los datos que se deben serializar y un búfer. En el ejemplo siguiente, asignamos un nuevo búfer que es mucho más grande de lo necesario, luego codificamos los datos en ese búfer y lo cortamos en un nuevo búfer que solo es tan grande como se necesita.

Una vez que se ha creado correctamente un búfer y se ha serializado los datos, lo único que queda es construir la transacción. Esto es similar a lo que ha hecho en lecciones anteriores. El ejemplo a continuación supone que:
- **player** , **playerInfoAccount** y **PROGRAM_ID** ya están definidos en algún lugar fuera del fragmento de código
- **player** es la llave pública de un usuario
- **playerInfoAccount** es la llave pública de la cuenta donde se escribirán los cambios del inventario
- **SystemProgram** se utilizará en el proceso de ejecución de la instrucción.

 # Demostración
Practiquemos juntos construyendo una aplicación de Revisión de Películas que permite a los usuarios enviar una revisión de una película y almacenarla en la red de Solana. Construiremos esta aplicación poco a poco en las próximas lecciones, agregando nuevas funcionalidades en cada lección.
 
La llave pública del programa Solana que utilizaremos para esta aplicación es **CenYq6bDRB7p73EjsPEpiYN7uveyPUTdXkDkgUduboaN** .

“” 1. Descargar el Código Inicial
Antes de comenzar, descargue **el código inicial** .
El proyecto es una aplicación Next.js bastante simple. Incluye el **WalletContextProvider** que creamos en la lección de Wallets, un componente **Card** para mostrar una revisión de película, un componente **MovieList** que muestra revisiones en una lista, un componente **Form** para enviar una nueva revisión y un archivo **Movie.ts** que contiene una definición de clase para un objeto **Movie** .
Tenga en cuenta que por ahora, las películas que se muestran en la página cuando ejecuta **npm run dev** son mocks. En esta lección, nos enfocaremos en agregar una nueva revisión, pero no podremos ver esa revisión en pantalla. En la próxima lección, nos enfocaremos en deserializar datos personalizados de cuentas en cadena.

## 2. Crear la Disposición del Búfer
Recuerde que para interactuar adecuadamente con un programa de Solana, necesita saber cómo se espera que los datos estén estructurados. Nuestro programa de Revisión de Películas espera que los datos de instrucción contengan:
1. **variante** como un entero sin signo de 8 bits que representa qué instrucción debe ejecutarse (en otras palabras, qué función del programa debe llamarse).
2. **título** como una cadena que representa el título de la película que está revisando.
3. **puntuación** como un entero sin signo de 8 bits que representa la puntuación de 5 que está dando a la película que está revisando.
4. **descripción** como una cadena que representa la parte escrita de la revisión que está dejando para la película.

Configuremos un esquema de instrucción **borsh** en la clase **Movie** . Comience importando **@project-serum/borsh**. A continuación, cree una propiedad **borshInstructionSchema** y establezca en el struct **borsh** apropiado que contiene las propiedades listadas anteriormente.

 Ten en cuenta que el orden importa. Si el orden de las propiedades aquí difiere de cómo está estructurado el programa, la transacción fallará.

## 3. Crear un Método para Serializar Datos
Ahora que tenemos la disposición del búfer configurada, creemos un método en **Movie** llamado **serialize()** que devolverá un **Buffer** con las propiedades de un objeto **Movie** codificadas en el diseño adecuado.

El método mostrado anteriormente primero crea un buffer lo suficientemente grande para nuestro objeto, luego codifica **{ ...this, variant: 0 }** en el buffer. Debido a que la definición de la clase **Movie** contiene 3 de las 4 propiedades requeridas por la disposición del buffer y utiliza el mismo nombre, se puede usar directamente con el operador de propagación y solo agregar la propiedad **variante** . Finalmente, el método devuelve un nuevo buffer que descarta la porción no utilizada del original.

## 4. Enviar Transacción cuando el usuario envía el formulario
Ahora que tenemos los bloques de construcción para los datos de instrucción, podemos crear y enviar la transacción cuando un usuario envía el formulario. Abra **Form.tsx** y busque la función **handleTransactionSubmit** . Esta se llama cada vez que un usuario envía el formulario de reseña de películas.
Dentro de esta función, crearemos y enviaremos la transacción que contiene los datos enviados a través del formulario.
Comience importando **@solana/web3.js** e importando **useConnection** y **useWallet** de **@solana/wallet-adapter-react** .


A continuación, antes de la función **handleSubmit** , llame a **useConnection()** para obtener un objeto de **conexión** y llame a **useWallet()** para obtener **publicKey** y **sendTransaction** .

Antes de implementar **handleTransactionSubmit** , hablemos sobre lo que necesita ser hecho. Necesitamos:
1. Comprobar que **publicKey** existe para asegurarnos de que el usuario ha conectado su billetera.
2. Llamar a **serialize()** en la **película** para obtener un buffer que representa los datos de instrucción.
3. Crear un nuevo objeto de **Transacción** .
4. Obtener todas las cuentas de las que la transacción leerá o escribirá.
5. Crear un nuevo objeto de **Instrucción** que incluya todas estas cuentas en los argumentos **keys** , incluya el buffer en el argumento **data** y incluya la llave pública del programa en el argumento **programId** .
6. Agregar la instrucción del paso anterior a la transacción.
7. Llamar a **sendTransaction** , pasando la transacción armada.

¡Esto es bastante para procesar! Pero no te preocupes, se vuelve más fácil a medida que lo haces más. Comencemos con los primeros 3 pasos de arriba:

El siguiente paso es obtener todas las cuentas de las que la transacción leerá o escribirá. En lecciones anteriores, se te ha dado la cuenta donde se almacenarán los datos. Esta vez, la dirección de la cuenta es más dinámica, por lo que debe ser calculada. Cubriremos esto en profundidad en la próxima lección, pero por ahora puedes usar lo siguiente, donde **pda** es la dirección de la cuenta donde se almacenarán los datos:

Además de esta cuenta, el programa también necesitará leer de **SystemProgram** , por lo que nuestro arreglo también debe incluir **web3.SystemProgram.programId** .
Con eso, podemos terminar los pasos restantes:

¡Y eso es todo! Ahora deberías poder usar el formulario en el sitio para enviar una reseña de película. Aunque no verás la actualización de la interfaz gráfica para reflejar la nueva reseña, puedes ver los registros de programa de la transacción en Solana Explorer para ver que fue exitosa.
Si necesitas un poco más de tiempo con este proyecto para sentirte cómodo, echa un vistazo al **código de solución** completa.

# Reto
Ahora es tu turno de construir algo de manera independiente. ¡Crea una aplicación que permita a los estudiantes de este curso presentarse a sí mismos! El programa Solana que soporta esto está en **HdE95RSVsdb315jfJtaykXhXY478h53X6okDupVfY9yf** .

1. Puedes construir esto desde cero o descargar el código de inicio **aquí** .
2. Crea la estructura del buffer de instrucción en **StudentIntro.ts** . El programa espera que los datos de instrucción contengan:
1. **variante** como un entero sin signo de 8 bits que represente la instrucción a ejecutar (debe ser 0).
2. **nombre** como una cadena que represente el nombre del estudiante.
3. **mensaje** como una cadena que represente el mensaje que el estudiante está compartiendo sobre su viaje en Solana.

3. Crea un método en **StudentIntro.ts** que utilice la estructura del buffer para serializar un objeto **StudentIntro**.
4. En el componente **Form**, implementa la función **handleTransactionSubmit** para que serialice un **StudentIntro** , construya la transacción y las instrucciones de transacción apropiadas y envíe la transacción a la billetera del usuario.
5. ¡Ahora deberías poder enviar presentaciones y tener la información almacenada en la cadena! Asegúrate de registrar el ID de transacción y verificarlo en Solana Explorer para verificar que funcionó.
Si te atascas, puedes consultar el código de solución **aquí** .
No dudes en ser creativo con estos desafíos y llevarlos aún más lejos. ¡Las instrucciones no están aquí para retenerte!
 

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

En programas más complejos que almacenan datos específicos del usuario, es común usar la llave pública del usuario como semilla. Esto separa los datos de cada usuario en su propio PDA. Esta separación permite al cliente localizar los datos de cada usuario encontrando la dirección utilizando el ID del programa y la llave pública del usuario.

Además, cuando hay varias cuentas por usuario, un programa puede usar una o varias semillas adicionales para crear e identificar cuentas. Por ejemplo, en una aplicación de toma de notas puede haber una cuenta por nota, donde cada PDA se deriva con la llave pública del usuario y el título de la nota.

## Obteniendo Múltiples Cuentas de Programa
Además de derivar direcciones, puedes obtener todas las cuentas creadas por un programa utilizando **connection.getProgramAccounts (programId)** . Esto devuelve una matriz de objetos donde cada objeto tiene una propiedad **pubkey** que representa la llave pública de la cuenta y una propiedad de **cuenta** de tipo **AccountInfo**. Puede utilizar la propiedad de **cuenta** para obtener los datos de la cuenta.


## Deserializando Datos de Cuenta Personalizados
La propiedad de **datos** en un objeto **AccountInfo** es un buffer. Para usarlo de manera eficiente, necesitarás escribir código que lo deserialice en algo más utilizable. Esto es similar al proceso de serialización que cubrimos en la última lección. Al igual que antes, utilizaremos **Borsh** y **@project-serum/borsh** . Si necesita repasar alguno de estos, eche un vistazo a la lección anterior.
La deserialización requiere conocer el diseño de la cuenta de antemano. Al crear sus propios programas, lo definirá como parte de ese proceso. Muchos programas también tienen documentación sobre cómo deserializar los datos de la cuenta. De lo contrario, si el código del programa está disponible, puede ver el código fuente y determinar la estructura de esa manera.
Para deserializar correctamente los datos de un programa en cadena, deberá crear un esquema del lado del cliente que refleje cómo se almacenan los datos en la cuenta. Por ejemplo, lo siguiente podría ser el esquema para una cuenta que almacena metadatos sobre un jugador en un juego en cadena.

 Una vez que tenga su diseño definido, simplemente llame a **.decode(buffer)** en el esquema.

# Demostracion
Practiquemos juntos continuando trabajando en la aplicación de revisión de películas de la última lección. No se preocupe si acaba de llegar a esta lección, debería ser posible seguir de cualquier manera.
Como recordatorio, este proyecto utiliza un programa Solana implementado en Devnet que permite a los usuarios revisar películas. En la última lección, agregamos funcionalidad al esqueleto de la interfaz frontal para que los usuarios pudieran enviar reseñas de películas, pero la lista de reseñas todavía muestra datos ficticios. Solucionemos eso recuperando las cuentas de almacenamiento del programa y deserializando los datos almacenados allí.

## 1. Descargue el Codigo Inicial
Si no completó la demostración de la lección anterior o simplemente quiere asegurarse de no haber perdido nada, puede descargar el **código inicial** .
El proyecto es una aplicación Next.js bastante simple. Incluye el **WalletContextProvider** que creamos en la lección de monederos, un componente **Card** para mostrar una revisión de película, un componente **MovieList** que muestra las reseñas en una lista, un componente **Form** para enviar una nueva reseña y un archivo **Movie.ts** que contiene una definición de clase para un objeto **Película** .
Tenga en cuenta que al ejecutar **npm run dev** , las reseñas que se muestran en la página son falsas. Sustituiremos esas por las verdaderas.

## 2. Crear el Diseño del Búfer
Recuerde que para interactuar adecuadamente con un programa Solana, necesita saber cómo está estructurada su información.
El programa de revisión de películas crea una cuenta separada para cada revisión de película y almacena los siguientes datos en los **datos** de la cuenta:
1. **Inicializado** como un booleano que representa si la cuenta se ha inicializado o no.
2. **Calificación** como un entero sin signo de 8 bits que representa la calificación de 5 que el revisor dio a la película.
3. **Título** como una cadena que representa el título de la película revisada.
4. **Descripción** como una cadena que representa la parte escrita de la reseña.
Configuremos un esquema **borsh** en la clase **Movie** para representar el diseño de datos de la cuenta de películas. Comience importando **@project-serum/borsh** . A continuación, cree una propiedad estática **borshAccountSchema** y establezca el struct **borsh** apropiado que contenga las propiedades mencionadas anteriormente.

Recuerda, el orden aquí importa. Debe coincidir con cómo está estructurada la información de la cuenta.

## 3. Crear un Método para Deserializar Datos
Ahora que tenemos el diseño del buffer configurado, creemos un método estático en **Movie** llamado **deserialize** que tomará un **Buffer** opcional y devolverá un objeto **Movie** o **null**.

El método primero verifica si el buffer existe y devuelve **null** si no lo hace. Luego, utiliza el diseño que creamos para decodificar el buffer, luego utiliza los datos para construir y devolver una instancia de **Movie** . Si la decodificación falla, el método registra el error y devuelve **null** .

## 4. Recuperar Cuentas de Reseñas de Películas
Ahora que tenemos una forma de deserializar los datos de la cuenta, necesitamos recuperar realmente las cuentas. Abra **MovieList.tsx** e importe **@solana/web3.js** . Luego, cree una nueva **conexión** dentro del componente **MovieList** . Finalmente, reemplace la línea **setMovies(Movie.mocks)** dentro de **useEffect** con una llamada a **connection.getProgramAccounts** . Tome el arreglo resultante y conviértalo en un arreglo de películas y llame a **setMovies** .

En este punto, ¡debería poder ejecutar la aplicación y ver la lista de reseñas de películas recuperadas del programa!
Dependiendo de cuántas reseñas se hayan enviado, esto puede tardar mucho tiempo en cargar o incluso bloquear completamente su navegador. Pero no se preocupe, en la próxima lección aprenderemos a dividir y filtrar las cuentas para poder ser más quirúrgico con lo que se carga.
Si necesita más tiempo con este proyecto para sentirse cómodo con estos conceptos, eche un vistazo al **código de solución** antes de continuar.

# Reto 
Ahora es tu turno de construir algo de forma independiente. En la última lección, trabajaste en la aplicación Student Intros para serializar los datos de instrucción y enviar una nueva introducción a la red. Ahora es el momento de recuperar y deserializar los datos de la cuenta del programa. Recuerde, el programa Solana que admite esto está en **HdE95RSVsdb315jfJtaykXhXY478h53X6okDupVfY9yf** .

1. Puedes construir esto desde cero o descargar el código de inicio aquí.
2. Crea el diseño del buffer de cuenta en **StudentIntro.ts.** La información de la cuenta contiene:
1. **inicializado** como un entero sin signo de 8 bits que representa la instrucción a ejecutar (debería ser 1).
2. **nombre** como una cadena que representa el nombre del estudiante.
3. **mensaje** como una cadena que representa el mensaje que el estudiante compartió sobre su viaje Solana.

3. Crea un método estático en **StudentIntro.ts** que utilizará el diseño del buffer para deserializar un buffer de datos de cuenta en un objeto **StudentIntro**.
4. En el componente de **useEffect** de **StudentIntroList** , obtiene las cuentas del programa y deserializa sus datos en una lista de objetos **StudentIntro** .
5. ¡En lugar de datos ficticios, ahora deberías ver las presentaciones de estudiantes de la red!

Si te atascas mucho, no dudes en revisar el código de solución **aquí** .
¡Como siempre, sé creativo con estos desafíos y lleva más allá de las instrucciones si lo deseas!
# Página, pedido y filtro de cuenta personalizados. 
## Objetivos de la Lección
Al final de esta lección, podrá:
 
-  Paginar, ordenar e filtrar cuentas
- Obtener cuentas sin datos de antemano
- Determinar dónde se almacena un dato específico en la estructura de buffer de una cuenta
- Obtener cuentas con un subconjunto de datos que se pueden utilizar para ordenar cuentas
- Recuperar solo cuentas cuyos datos coincidan con criterios específicos
- Obtener un subconjunto de cuentas totales utilizando **getMultipleAccounts**

# Terminología
- Esta lección se adentra en algunas funcionalidades de las llamadas RPC que utilizamos en la lección de deserialización de datos de cuenta
- Para ahorrar tiempo de cómputo, puede obtener un gran número de cuentas sin sus datos filtrándolas para devolver solo una matriz de llaves públicas
- Una vez que tenga una lista filtrada de llaves públicas, puede ordenarlas y obtener los datos de la cuenta a la que pertenecen.

# Resumen
Es posible que haya notado en la última lección que, aunque pudiéramos obtener y mostrar una lista de datos de cuenta, no tuviéramos ningún control granular sobre cuántas cuentas obtener o su orden. En esta lección, aprenderemos sobre algunas opciones de configuración para la función **getProgramAccounts** que habilitarán cosas como la paginación, ordenar cuentas y filtrar.

## Usar dataSlice para solo obtener los datos que necesita
Imagina la aplicación de reseñas de películas en la que trabajamos en lecciones anteriores con cuatro millones de reseñas de películas y que la reseña promedio sea de 500 bytes. Esto haría que la descarga total de todas las cuentas de reseñas fuera más de 2GB. Definitivamente no es algo que quieras que tu frontend descargue cada vez que se actualice la página.
Afortunadamente, la función **getProgramAccounts** que utilizas para obtener todas las cuentas toma un objeto de configuración como argumento. Una de las opciones de configuración es **dataSlice** que te permite proporcionar dos cosas:
- **offset** - el desplazamiento desde el principio del búfer de datos para comenzar el trozo
- **longitud** - el número de bytes a devolver, comenzando desde el offset proporcionado

Cuando incluyes un **dataSlice** en el objeto de configuración, la función solo devolverá el subconjunto del búfer de datos que especificaste.

## Paginación de Cuentas
Una de las áreas en las que esto es útil es con la paginación. Si desea tener una lista que muestre todas las cuentas, pero hay tantas cuentas que no desea obtener todos los datos de una vez, puede obtener todas las cuentas sin datos. Luego, puede asignar el resultado a una lista de llaves de cuenta cuyos datos solo puede obtener según sea necesario.
   	
Con esta lista de llaves, luego puede obtener datos de cuenta en "páginas" utilizando el método **getMultipleAccountsInfo** :

## Ordenando Cuentas
La opción **dataSlice** también es útil cuando necesita ordenar una lista de cuentas mientras se hace paginación. Todavía no deseas obtener todos los datos de una vez, pero si necesitas todas las llaves y una forma de ordenarlas de antemano. En este caso, necesitas entender el diseño de los datos de la cuenta y configurar el trozo de datos para que solo sea la información que necesitas usar para ordenar.
Por ejemplo, podrías tener una cuenta que almacene información de contacto de la siguiente manera:
- **inicializado** como un booleano
- **número de teléfono** como un entero sin signo de 64 bits
- **primer nombre** como una cadena
- **segundo nombre** como una cadena

Si desea ordenar todas las llaves de cuenta alfabéticamente en función del nombre del usuario, debe descubrir el desplazamiento donde comienza el nombre. El primer campo, **inicializado** , ocupa el primer byte, luego el **número de teléfono** ocupa otro 8, por lo que el campo **firstName** comienza en offset **1 + 8 = 9** .
Luego debe determinar la longitud para hacer el trozo de datos. Dado que la longitud es variable, no podemos saberlo con certeza, pero puede elegir una longitud que sea lo suficientemente grande para cubrir la mayoría de los casos y lo suficientemente corta como para no ser una carga para obtener. 15 bytes es suficiente para la mayoría de los primeros nombres, pero resultaría en una descarga pequeña incluso con un millón de usuarios.

Una vez que ha obtenido cuentas con el trozo de datos dado, puede usar el método **sort** para ordenar la matriz antes de asignarla a una matriz de llaves públicas.

Tenga en cuenta que en el fragmento anterior no comparamos los datos tal como se dan. Esto se debe a que para tipos de tamaño dinámico como las cadenas, Borsh coloca un entero sin signo de 32 bits al principio para indicar la longitud de los datos que representan ese campo. Por lo tanto, para comparar directamente los nombres, necesitamos obtener la longitud de cada uno, luego crear un trozo de datos con un desplazamiento de 4 bytes y la longitud adecuada.

## Usa filtros para recuperar solo cuentas específicas
Limitar los datos recibidos por cuenta es genial, pero ¿qué pasa si solo deseas devolver cuentas que coincidan con un criterio específico en lugar de todas ellas? Ahí es donde entra en juego la opción de **filtros** . Esta opción es una matriz que puede tener objetos que coincidan con lo siguiente:
- **memcmp** - compara una serie de bytes proporcionados con los datos de cuenta del programa en un punto de offset específico. Campos:
- **offset** - el número de desplazamiento en los datos de cuenta del programa antes de comparar los datos
- **bytes** - una cadena codificada en base 58 que representa los datos para comparar; limitado a menos de 129 bytes
- **dataSize** - compara la longitud de los datos de cuenta del programa con el tamaño de datos proporcionado

Esto te permite filtrar en función de los datos que coinciden y/o el tamaño total de los datos.
Por ejemplo, podrías buscar en una lista de contactos incluyendo un filtro **memcmp** :

Dos cosas a tener en cuenta en el ejemplo anterior:
1. Estamos estableciendo el desplazamiento en 9 porque eso es lo que determinamos anteriormente es el desplazamiento donde comienza **firstName** en el diseño de datos.
2. Estamos usando una biblioteca de terceros **bs58** para realizar la codificación base-58 en el término de búsqueda. Puedes instalarlo usando **npm install bs58** .

# Demostración
¿Recuerdas la aplicación de reseñas de películas en la que trabajamos en las últimas dos lecciones? Vamos a darle un toque especial al paginando la lista de reseñas, ordenando las reseñas para que no sean tan aleatorias e incluyendo una funcionalidad de búsqueda básica. No te preocupes si acabas de comenzar esta lección sin haber mirado las anteriores, siempre y cuando tengas los conocimientos previos, deberías ser capaz de seguir la demostración sin haber trabajado en este proyecto específico todavía.

## 1. Descargue el código de inicio
Si no completó la demostración de la última lección o solo desea asegurarse de no haber perdido nada, puede descargar el **código de inicio** .
El proyecto es una aplicación Next.js bastante simple. Incluye el **WalletContextProvider** que creamos en la lección de monederos, un componente **Card** para mostrar una reseña de película, un componente **MovieList** que muestra reseñas en una lista, un componente **Form** para enviar una nueva reseña y un archivo **Movie.ts** que contiene una definición de clase para un objeto **Película** .

## 2. Agregue paginación de reseñas
Primero que nada, creemos un espacio para encapsular el código para obtener los datos de la cuenta. Crea un nuevo archivo **MovieCoordinator.ts** y declara una clase **MovieCoordinator** . Luego, movamos la constante **MOVIE_REVIEW_PROGRAM_ID** de **MovieList** a este nuevo archivo ya que vamos a mover todas las referencias a ella.

Ahora podemos usar **MovieCoordinator** para crear una implementación de paginación. Una nota rápida antes de sumergirnos: esta será una implementación de paginación tan simple como sea posible para que podamos centrarnos en la parte compleja de interactuar con las cuentas de Solana. Puedes y debes hacerlo mejor para una aplicación en producción.
Con eso fuera de camino, creemos una propiedad estática **accounts** de tipo **web3.PublicKey[]** , una función estática **prefetchAccounts (conexión: web3.Connection)** y una función estática **fetchPage (conexión: web3.Connection, página: number, porPágina: number): Promise<Movie[]>** . También necesitará importar **@solana/web3.js** y **Movie** .

La llave para la paginación es pre-cargar todas las cuentas sin datos. Rellenemos el cuerpo de **prefetchAccounts** para hacerlo y establezcamos las llaves públicas recuperadas en la propiedad **accounts** estática.

Ahora, llenemos el método **fetchPage** . Primero, si las cuentas aún no se han pre-cargado, deberemos hacerlo. Luego, podemos obtener las llaves públicas de las cuentas que corresponden a la página solicitada y llamar a **connection.getMultipleAccountsInfo** . Finalmente, deserializamos los datos de la cuenta y devolvemos los objetos **Película** correspondientes.

Con eso hecho, podemos reconfigurar **MovieList** para usar estos métodos. En **MovieList.tsx** , agregue **const [page, setPage] = useState (1)** cerca de las llamadas **useState** existentes. Luego, actualice **useEffect** para llamar a **MovieCoordinator.fetchPage** en lugar de recuperar las cuentas en línea.

Por último, necesitamos agregar botones en la parte inferior de la lista para navegar a diferentes páginas:

En este punto, ¡deberías poder ejecutar el proyecto y hacer clic entre las páginas!

## 3. Ordenar reseñas alfabeticamente
Si miras las reseñas, es posible que te des cuenta de que no están en ningún orden específico. Podemos arreglar esto agregando de nuevo suficientes datos en nuestra porción de datos para ayudarnos a hacer algunas clasificaciones. Las diversas propiedades en el búfer de datos de la reseña de la película están dispuestas de la siguiente manera:
- **inicializado** - entero sin signo de 8 bits; 1 byte
- **calificación** - entero sin signo de 8 bits; 1 byte
- **título** - cadena; número desconocido de bytes
- **descripción** - cadena; número desconocido de bytes

Basado en esto, el desplazamiento que necesitamos proporcionar a la porción de datos para acceder al **título** es 2. La longitud, sin embargo, es indeterminada, por lo que podemos proporcionar solo una longitud razonable. Me quedaré con 18 ya que cubrirá la longitud de la mayoría de los títulos sin tener que recuperar demasiados datos cada vez.
Una vez que hemos modificado la porción de datos en **getProgramAccounts** , entonces necesitamos realmente ordenar el arreglo devuelto. Para hacer esto, necesitamos comparar la parte del búfer de datos que realmente corresponde al **título** . Los primeros 4 bytes de un campo dinámico en Borsh se utilizan para almacenar la longitud del campo en bytes. Entonces, en cualquier búfer de **datos** dado que está recortado de la manera en que discutimos anteriormente, la porción de cadena es **data.slice(4, 4 + data[0])** .
Ahora que hemos pensado en esto, modifiquemos la implementación de **prefetchAccounts** en **MovieCoordinator** :

Y así es como, deberías poder ejecutar la aplicación y ver la lista de reseñas de películas ordenadas alfabéticamente.
 
## 4. Agregar búsqueda
Lo último que haremos para mejorar esta aplicación es agregar alguna capacidad de búsqueda básica. Agreguemos un parámetro de **búsqueda** a **prefetchAccounts** y reconfiguramos el cuerpo de la función para usarlo.
Podemos usar la propiedad **filtros** del parámetro **config** de **getProgramAccounts** para filtrar cuentas por datos específicos. El desplazamiento a los campos de título es 2, pero los primeros 4 bytes son la longitud del título, por lo que el desplazamiento real al string en sí es 6. Recuerda que los bytes deben ser codificados en base 58, así que instalemos e importemos **bs58** .

Ahora, agregue un parámetro de **búsqueda** a **fetchPage** y actualice su llamada a **prefetchAccounts*** para pasarlo. También necesitaremos agregar un parámetro de **recarga** booleano a **fetchPage** para que podamos forzar una actualización de la pre-carga de cuentas cada vez que cambie el valor de búsqueda.

Con eso en su lugar, actualicemos el código en **MovieList** para llamarlo correctamente.

Primero, agregue **const [search, setSearch] = useState('')** cerca de las otras llamadas **useState** . Luego, actualice la llamada a **MovieCoordinator.fetchPage** en el **useEffect** para pasar el parámetro de **búsqueda** y para recargar cuando **search !== ''** .

Finalmente, agregue una barra de búsqueda que establecerá el valor de **search**:
 
¡Y eso es todo! La aplicación ahora tiene reseñas ordenadas, paginación y búsqueda.
Eso fue mucho para digerir, pero lo lograste. Si necesitas pasar más tiempo con los conceptos, no dudes en volver a leer las secciones que fueron más desafiantes para ti y/o echa un vistazo al **código de solución** .
# Reto 
Ahora es tu turno de intentar hacerlo por tu cuenta. Utilizando la aplicación de presentación de estudiantes de la lección anterior, agrega la división en páginas, ordena alfabéticamente por nombre y busca por nombre.
 
1. Puedes construir esto desde cero o puedes descargar el **código inicial** .
2. Agrega la paginación al proyecto pre-cargando cuentas sin datos, luego solo cargando los datos de la cuenta para cada cuenta cuando sea necesario.
3. Ordena las cuentas que se muestran en la aplicación alfabéticamente por nombre.
4. Agrega la capacidad de buscar a través de las presentaciones por el nombre de un estudiante.
Este es un desafío. Si te quedas atascado, no dudes en consultar el **código de solución** . ¡Con esto completas el Módulo 1! ¿Cómo ha sido tu experiencia? ¡Sientete libre de compartir alguna retroalimentación rápida **aquí** , para que podamos seguir mejorando el curso!
¡Como siempre, sé creativo con estos desafíos y lleva los más allá de las instrucciones si lo deseas!