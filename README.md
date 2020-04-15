### Escuela Colombiana de Ingeniería

### Arquitecturas de Software - ARSW

### Integrantes

- **Juan Alberto Mejía Schuster**
- **Johann Sebastian Páez Campos**

### Links a las preguntas

[Preguntas parte 1](#preguntas-parte-1)

[Preguntas parte 2](#preguntas-parte-2)

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/en-us/free/search/?&ef_id=Cj0KCQiA2ITuBRDkARIsAMK9Q7MuvuTqIfK15LWfaM7bLL_QsBbC5XhJJezUbcfx-qAnfPjH568chTMaAkAsEALw_wcB:G:s&OCID=AID2000068_SEM_alOkB9ZE&MarinID=alOkB9ZE_368060503322_%2Bazure_b_c__79187603991_kwd-23159435208&lnkd=Google_Azure_Brand&dclid=CjgKEAiA2ITuBRDchty8lqPlzS4SJAC3x4k1mAxU7XNhWdOSESfffUnMNjLWcAIuikQnj3C4U8xRG_D_BwE). Al hacerlo usted contará con $200 USD para gastar durante 1 mes.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Imágen 1](images/part1/part1-vm-basic-config.png)
   
2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM.

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    `npm install forever -g`

    `forever start FibinacciApp.js`

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
    * 1010000
    * 1020000
    * 1030000
    * 1040000
    * 1050000
    * 1060000
    * 1070000
    * 1080000
    * 1090000    

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

- Antes del cambio de tamaño

![Maquina A0](https://github.com/JohannPaez/ARSW-LAB8/blob/master/images/Test/1cpu-0.75%20GiB/1000000-1090000.PNG)

- Depues del cambio de tamaño

![Maquina A3](https://github.com/JohannPaez/ARSW-LAB8/blob/master/images/Test/1cpu-0.75%20GiB/1000000-1090000-A3machine.PNG)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

- Antes del cambio de tamaño

![Maquina A0](https://github.com/JohannPaez/ARSW-LAB8/blob/master/images/Test/1cpu-0.75%20GiB/newman.PNG)

- Depues del cambio de tamaño

![Maquina A3](https://github.com/JohannPaez/ARSW-LAB8/blob/master/images/Test/1cpu-0.75%20GiB/newman%20A3.PNG)


10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`. ** No fue posible seleccionar el tamaño B2ms por lo que se seleccionó la A3 **

![Imágen 3](images/part1/part1-vm-resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

## Preguntas parte 1

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?

   Hay 6 recursos que Azure crea junto con la VM y son:
      - Disco
      - Interfaz de Red
      - Dirección IP Pública
      - Grupo de Seguridad de Red
      - Red Virtual
      - Cuenta de Almacenamiento
2. ¿Brevemente describa para qué sirve cada recurso?
   -  Disco: Se utiliza para aumentar la capacidad de almacenamiento de la VM.
   - Interfaz de Red: Es como una tarjeta de red, permite que la VM pueda comunicarse con otros recursos de la red local, así como la comunación hacía internet. 
   - Dirección IP Pública: Es la dirección IP, que nos permite acceder al recurso remotamente, mediante SSH, por ejemplo.
   - Grupo de Seguridad de Red: Es el grupo que contiene las reglas o normas que nos permiten garantizar una mejor seguridad en nuestra VM, restringiendo el tráfico de entrada o salida hacía la red a partir de los recursos disponibles por Azure.
   - Red Virtual: Es una red privada en Azure la cual permite comunicar la VM con distintos recursos (internet, redes on-premise) de manera segura.
   - Cuenta de Almacenamiento: Es la que contiene todos los datos de Azure Storage (archivos, discos, etc).
3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?
   - La aplicación se cae debido a que al cerrar la conexión ssh con la VM, se lanza un llamado a todos los procesos para que se cierren, luego forever tiene la tarea de mantener la ejecución del script, esto para que cuando se intente cerrar el proceso, lo vuelva a ejecutar y se mantenga el servicio activo.
   - Debemos crear un *Inbound port rule* para abrir el puerto utilizado en los experimientos, permitiendo el tráfico de entrada y salida de red.
4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

Número | Tamaño A0 (min) | Tamaño A3 (min)
--- |--- |--- |
1000000 | 2:10 | 1:10
1010000 | 2:16 | 1:11
1020000 | 2:19 | 1:13
1030000 | 2:28 | 1:15
1040000 | 2:36 | 1:17
1050000 | 2:44 | 1:20
1060000 | 2:50 | 1:22
1070000 | 2:53 | 1:23
1080000 | 2:58 | 1:25
1090000 | 3:04 | 1:26

- La función se demora tanto debido a que no guarda los resultados ya calculados, cada solicitud que llega se debe procesar desde 0 como si la máquina acabara de encender, Fibonacci necesita el número anterior para calcular el siguiente número de la secuencia por esto el echo de no guardar nada afecta el rendimiento

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

![](https://github.com/JohannPaez/ARSW-LAB8/blob/master/images/Test/1cpu-0.75%20GiB/1000000-1090000.PNG)

Cada pico de la gráfica es una consulta, no se puede ver en la imagen pero la línea a la que todas llegan es el 100% de la CPU, en esta primera parte solo tenemos un procesador, no es posible para el servidor distribuir el cálculo en varios procesadores y la aplicación no está optimizada lo que causa el 100% del uso.

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:

![](https://github.com/JohannPaez/ARSW-LAB8/blob/master/images/Test/1cpu-0.75%20GiB/newman.PNG)

  - En la imagen podemos ver que el tiempo promedio de las 10 peticiones fue de 2 minutos con 26 segundos, la petición más demorada tomó 4        minutos y 11 segundos en ser completada.

7. ¿Cuál es la diferencia entre los tamaños `A0` y `A3` (no solo busque especificaciones de infraestructura)?

      La principal diferencia es que los despliegues del cliente pueden afectar el rendimiento del tamaño A0 ya que este está suscrito en el hardware físico, sin embargo el tamaño A0 es más económico que el tamaño A3.
      
      En la siguiente tabla se muestran algunas diferencias entre estos tamaños:

Nombre | vCPU | Memoria | NIC s | Tamaño total de disco | Tamaño máximo de la información(1023GBc/u) | Max. IOPS (300 per disk)
--- |--- |--- |--- |--- |--- |--- |
A0 | 1 | 768 MB | 2 | 20 GB | 1 | 1 x 300
A3 | 4 | 7 GB | 2 | 120 GB | 8 | 8 x 300

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?

      No, debido a que no se puede determinar con seguridad cual va a ser la carga del servicio, para el ejercicio que realizamos si podemos ver una mejora en el rendimiento, pero en un caso real la solución solo es temporal si se aumenta la demanda del servicio un escalamiento vertical no es la estrategia a seguir para mantener el servicio corriendo.
      
   ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?
   
      La aplicación ahora cuenta con mas recursos para calcular la secuencia, se ve una mejora en los tiempos de respuesta, pero no soluciona la mala optimización de la aplicación.
      
9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

      Al cambiar el tamaño la maquina se debe reiniciar, esto afecta la disponibilidad de nuestro servicio , por unos minutos cualquier petición será ignorada.
      
10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

      Si, los tiempos de respuesta se redujeron a la mitad y el consumo de la cpu no supero el 30%, esto se debe a que la aplicación contó con más recursos para realizar los cálculos y podía manejar más solicitudes simultaneas
      
11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

      No es porcentualmente mejor, pues la cantidad de peticiones realizadas con exito disminuyó aproximadamente un 15% con respecto a las 2 peticiones anteriores, sin embargo en ambos casos se presento el mismo consumo de CPU.

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```
![Hello world](https://github.com/JohannPaez/ARSW-LAB8/blob/master/images/part2/Balanceador/Url_Balanceador_HelloWorld.png)

![finoacci 1](https://github.com/JohannPaez/ARSW-LAB8/blob/master/images/part2/Balanceador/Url_Balanceador_Fibonnaci.png)

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

## Informe
![](https://github.com/JohannPaez/ARSW-LAB8/blob/master/images/part2/Balanceador/Parte2_PruebasParte1.png)

![](https://github.com/JohannPaez/ARSW-LAB8/blob/master/images/part2/Balanceador/Parte2_PruebasParte1_Tabla.png)

Comparando los dos tipos de escalamiento que realizamos podemos concluir varias cosas:
   -	El tiempo de respuesta para el escalamiento horizontal es mucho menor (20.7s) comparado con el tiempo de respuesta para el escalamiento vertical (2m 26s)
   -	Las dos respondieron todas las peticiones que se le realizaron
   -	En costos para el escalamiento vertical se usó en nuestro caso (Por restricciones con Azure) los tamaños A0 y A3 ($14.60 y $175.20) y para el escalamiento horizontal se usaron las B1ls (4 x $4.53). Esto nos da un mejor costo con el escalamiento horizontal con un costo de $18.12  

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```
**Newman** 

![](https://github.com/JohannPaez/ARSW-LAB8/blob/master/images/part2/Balanceador/Parte3_Pruebas.png)

![](https://github.com/JohannPaez/ARSW-LAB8/blob/master/images/part2/Balanceador/Parte3_Pruebas_Tabla.png)

**CPU**

![VM 1](https://github.com/JohannPaez/ARSW-LAB8/blob/master/images/part2/Balanceador/VM1_Estadisticas.png)


![VM 2](https://github.com/JohannPaez/ARSW-LAB8/blob/master/images/part2/Balanceador/VM2_Estadisticas.png)


![VM 3](https://github.com/JohannPaez/ARSW-LAB8/blob/master/images/part2/Balanceador/VM3_Estadisticas.png)

**Análisis**

Aumentó el éxito de las peticiones debido a que el servidor tenia la capacidad para recibirlas y procesarlas, a diferencia del otro que se llenaba con una sola petición y le tocaba ignorar las que llegaban hasta que tenia espacio para recibirlas

## Preguntas parte 2

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?

Azure ofrece dos tipos de balanceador de carga, público e interno,

   - Balanceador de carga Público (Public Load Balance): asigna la dirección IP pública y el puerto del tráfico entrante a la dirección IP privada, se usa para conectar la infraestructura interna con el internet.
   
   - Balanceador de carga interno (Internal Load Balancer): Usan IP privadas solo en la interfaz. Estos balanceadores se utilizan para equilibrar el tráfico dentro de una red virtual.

* ¿Qué es SKU, qué tipos hay y en qué se diferencian?

SKU (Stock Keeping Unit): En español Unidad de mantenimiento de existencias, es un número o código asignado a un elemento para poder identificarlo. Existen 6 tipos diferentes de SKU:	
   Estándar

   •	Un producto estándar.
   
   •	Los SKUs estándar pueden venderse individualmente o como partes de conjuntos, paquetes o colecciones.
      
   Componente
   
   •	Los SKUs de los componentes son partes incluidas en los ensamblajes, paquetes y colecciones.
   
   •	Los SKUs de componentes no pueden venderse como productos independientes.
   
   Ensamblaje
   
   •	Un producto que debe ser ensamblado antes de su envío. El proceso de ensamblaje puede ser tan simple como poner todos los SKUs asociados en la misma caja. O puede ser tan complicado como soldar las piezas juntas.
   
   •	Todos los SKUs asociados necesarios para el ensamblaje deben estar ubicados dentro de la misma instalación, ya sea su instalación local o la de un proveedor de Dropship/JIT/3PL.
   
   •	Para vender SKUs de ensamblaje, todas las cantidades necesarias de todos los SKUs asociados deben estar disponibles.
   
   Paquete
   
   •	Un producto que incluye SKUs asociados que no necesitan ser ensamblados juntos antes del envío.
   
   •	Los SKUs asociados incluidos en el paquete pueden ser enviados a los clientes desde diferentes fuentes de cumplimiento.
   
   •	Para vender SKU de paquetes, deben estar disponibles todas las cantidades necesarias de todos los SKU asociados.
   
   Recogida
   
   •	Una colección de SKUs asociados vinculados con fines de comercialización para la venta superior o cruzada de productos relacionados.
   
   •	El conjunto de SKU en Colección actúa como una especie de SKU matriz para todos los SKU asociados. Sin embargo, el SKU de Colección no puede venderse en sí mismo. Sólo se pueden vender los SKU asociados.
   
   •	Los SKU asociados de la colección pueden seguir vendiéndose si alguno de los otros SKU asociados se vende.
   
   Virtual
   
   •	Una SKU que no necesita ser cumplida físicamente, como una suscripción a una revista o una cuota de envoltura de regalo. 
   
   •	Estos SKU no requieren inventario, y operan con un nivel de inventario de 0.
   
   •	Los SKU virtuales no sincronizan el catálogo en la actualidad, sino que se asocian a los pedidos que se importan con un SKU correspondiente. 

* ¿Por qué el balanceador de carga necesita una IP pública?

El balanceador de carga necesita una IP pública para que pueda cumplir su función,  el objetivo es que todas las solicitudes lleguen al balanceador para que este pueda distribuirlas si no fuera accesible al público no podría recibir las solicitudes.

* ¿Cuál es el propósito del *Backend Pool*?

Un Backend Pool se refiere al conjunto de backends que reciben un tráfico similar para su aplicación. En otras palabras, es una agrupación lógica de las instancias de su aplicación en todo el mundo que reciben el mismo tráfico y responden con el comportamiento esperado. 

* ¿Cuál es el propósito del *Health Probe*?

El propósito del Healt Probe es mantener un registro de que instancias están activas en el Backend Pool para que el balanceador de carga sepa con que recursos cuenta y pueda asignar tareas a estas instancias

* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.

Se utiliza una Load Balancing Rule para definir cómo se distribuye el tráfico a las máquinas virtuales. Se define la configuración de IP del frontend para el tráfico entrante y el pool de IP del backend para recibir el tráfico, junto con el puerto de origen y destino requerido.
Azure maneja 3 tipos:

   - Ninguno (basado en el hash) - Especifica que las sucesivas solicitudes del mismo cliente pueden ser manejadas por cualquier máquina virtual.
   
   - IP del cliente (afinidad de IP de origen 2-tupla) - Especifica que las sucesivas solicitudes de la misma dirección IP del cliente serán manejadas por la misma máquina virtual.
   
   - IP y protocolo del cliente (afinidad de IP de origen 3-tupla) - Especifica que las sucesivas solicitudes de la misma combinación de dirección IP y protocolo del cliente serán manejadas por la misma máquina virtual.

Puede afectar la escalabilidad del sistema al restringir la cantidad de máquinas virtuales que atienden las solicitudes, algunas reglas restringen la máquina virtual a una IP específica, si esta IP realiza muchas solicitudes puede que la maquina virtual asignada no sea capaz de manejar esa carga.

* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?

   - Virtual Network: La Red Virtual Azure (VNet) es el elemento fundamental de la red privada en Azure. VNet permite que muchos tipos de recursos de Azure, como las Máquinas Virtuales Azure (VM), se comuniquen de forma segura entre sí, con Internet y con las redes locales. VNet es similar a una red tradicional que se opera en un centro de datos, pero trae consigo beneficios adicionales de la infraestructura de Azure, tales como escalabilidad, disponibilidad y aislamiento.
   
   - Subnet: Las subredes permiten segmentar la red virtual en una o más subredes y asignar una porción del espacio de direcciones de la red virtual a cada subred. Luego puede desplegar recursos de Azure en una subred específica. Al igual que en una red tradicional, las subredes le permiten segmentar el espacio de direcciones de la red virtual en segmentos que son apropiados para la red interna de la organización.
   
   - Address space: Cuando se crea una VNet, se debe especificar un espacio de direcciones IP privadas personalizadas utilizando direcciones públicas y privadas (RFC 1918). Azure asigna a los recursos de una red virtual una dirección IP privada del espacio de direcciones que usted asigne. Por ejemplo, si usted despliega un VM en una VNet con espacio de direcciones, 10.0.0.0/16, al VM se le asignará una IP privada como 10.0.0.4.
   
   - Address range: Es el numero que indica cuantas direcciones tenemos en ese espacio de direcciones y depende de la cantidad de recursos que se necesiten en la red virtual, el rango sera mayor o menor.
   
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?

   - Availability Zone: Availability Zones es una oferta de alta disponibilidad que protege sus aplicaciones y datos de las fallas del centro de datos. Las zonas de disponibilidad son ubicaciones físicas únicas dentro de una región azul. Cada zona está compuesta por uno o más centros de datos equipados con energía, refrigeración y redes independientes. Para garantizar la resiliencia, hay un mínimo de tres zonas separadas en todas las regiones habilitadas y seleccionamos tres zonas para hacer uso de los beneficios que se ofrecen si solo seleccionamos una perdemos la seguridad contra fallos.
   
   - zone-redundant: La Ip por si sola no es zone-redundant, lo que se considera que es zone-redundant es la gateway y significa que al usar una gateway zone-redundant se quiere aportar resistencia, escalabilidad y mayor disponibilidad. El despliegue de gateways en las zonas de disponibilidad del Azur separa física y lógicamente las puertas de enlace dentro de una región, a la vez que protege la conectividad de su red local al Azur de los fallos a nivel de zona. 
   
* ¿Cuál es el propósito del *Network Security Group*?

Se puede utilizar el grupo de seguridad de red Azure para filtrar el tráfico de red hacia y desde los recursos Azure en una red virtual Azure. Un grupo de seguridad de red contiene reglas de seguridad que permiten o deniegan el tráfico de red entrante a, o el tráfico de red saliente de varios tipos de recursos Azure. Para cada regla, puede especificar el origen y el destino, el puerto y el protocolo.

* Informe de newman 1 (Punto 2)

     [Informe](#informe)

* Presente el Diagrama de Despliegue de la solución.

![](images/Component%20Diagram.png)

## Bibliografía 

https://docs.microsoft.com/en-us/azure/virtual-network/public-ip-address-prefix

https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview

https://docs.microsoft.com/en-us/azure/virtual-network/manage-virtual-network

https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview

https://docs.microsoft.com/en-us/azure/availability-zones/az-overview

https://docs.microsoft.com/bs-latn-ba/azure/vpn-gateway/about-zone-redundant-vnet-gateways




