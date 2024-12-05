## Flink Python examples

Esta carpeta contiene ejemplos de Kinesis Data Streams y Kinesis Data Firehose & data analytics (que utiliza servicio gestionado por AWS para Apache Flink) escritos en Python. 

Los ejemplos completos están extraidos de la web de AWS: 
[Crear y ejecutar un servicio administrado para la aplicación Apache Flink para Python](https://docs.aws.amazon.com/managed-flink/latest/java/gs-python-createapp.html)

---

### Crear recursos dependientes
En este tutorial se supone que estamos construyendo la aplicación en la región ***us-east-1***, que es la disponible para AwsAcademy. Si se utiliza otra región, deberán adaptarse todos los pasos en consecuencia.

***IMPORTANTE***  Recordad que debemos tener instalado AWS CLI en nuestro equipo local. Si no fuese el caso, podemos hacer todo lo que viene a continuación desde Cloud9 o desde el shell integrado en la consola de AWS.
Podemos verrificar que tenemos instalado AWS CLI con
~~~
aws --version
~~~

Debemos crear los siguientes recursos:
___Dos flujos de Kinesis Data Stream___ para entrada y salida. Podemos hacerlo desde la consola de AWS o por AWS CLI con Visual Studio Code. Lo haremos de esta última forma por ser más rápido.

~~~ 
aws kinesis create-stream \
--stream-name ExampleInputStream \
--shard-count 1 \
--region us-east-1
~~~ 

~~~
aws kinesis create-stream \
--stream-name ExampleOutputStream \
--shard-count 1 \
--region us-east-1 
~~~

En el ejemplo completo de AWS, se crea también un ***bucket de S3***, pero no lo haremos aquí sino que es objeto de estudio en otro laboratorio.

### Configurar el entorno de desarrollo local

Se supone que está instalado Visual Studio Code con la extensión de AWS Toolkit.

#### 1. Instalar la biblioteca PyFlink
Es conveniente utilizar la misma versión del entorno de ejecución de APache Flink que usa AWS. Podemos seleccionar la versión como vemos en el siguiente comando. Nosotros usaremos la 1.19.1

~~~~
pip install apache-flink==1.19.1
~~~~

#### 2. Comprobar que tenemos conexión con el stream de entrada

Recordar que si tenemos varios perfiles de AWS, debemos ejecutar
~~~~
export AWS_PROFILE=academy
~~~~

Escribimos cualquier dato en uno de los flujos de KDS
~~~~
aws kinesis put-record --stream-name ExampleOutputStream --data TEST --partition-key TEST
~~~~

#### 3. Descargar el codigo Python

El código de la aplicación Python para este ejemplo está disponible en GitHub. 
~~~~
https://github.com/juanhernu/aos/tree/main/kinesis/amazon-managed-service-for-apache-flink-examples
~~~~

El original de AWS puede obtenerse de
~~~~
git clone https://github.com/aws-samples/amazon-managed-service-for-apache-flink-examples.git
~~~~

#### 4. Revisar los componentes de la aplicación 

El envío de datos al strem de Kinesis está bajo el directorio 
~~~~ 
./python/data-generator
~~~~
Por su parte, el consumidor de datos se encuentra en ***main.py*** bajo el directorio 
~~~~ 
./python/GettingStarted
~~~~
Diferentes tipos de combinaciones de Kinesis y ventanas se encuentran en

~~~~ 
./python/FirehouseSink
./python/S3Sink
./python/Windowing
~~~~

> ***Nota*** 
Para una experiencia optimizada para el desarrollador, la aplicación está diseñada para ejecutarse sin ningún cambio de código tanto en Amazon Managed Service for Apache Flink como de forma local, para el desarrollo en su máquina. La aplicación utiliza la variable de entorno ___IS_LOCAL = true___ para detectar cuándo se está ejecutando de forma local. Nosotros configuraremos esa variable 
~~~~ 
IS_LOCAL = true
~~~~

---

#### 5. Dependencias de descarga y empaquetado
Este ejemplo tiene una serie de dependencas definidas en el archivo ___pom.xml___
Para descargarlas, 

~~~~
Vamos al directorio ./python/GettingStarted

Ejecutamos
$ mvn package
~~~~

Maven crea un nuevo archivo llamado ___./target/pyflink-dependencies.jar___. Cuando desarrollas localmente en tu máquina, la aplicación Python busca este archivo.

---


#### 6. Escribir registros en el flujo de entrada

Basta con ejecutar el archivo 
~~~~
python stock.py
~~~~
Este ejemplo genera eventos con este formato

~~~~
{
  "event_time" : "{{date.now("YYYY-MM-DDTkk:mm:ss.SSSSS")}},
  "ticker" : "{{random.arrayElement(
        ["AAPL", "AMZN", "MSFT", "INTC", "TBV"]
    )}}",
  "price" : {{random.number(100)}}          
} 
~~~~
Podemos comprobar que los datos se están generando correctamente si vamos a Kinesis desde la consola de AWS y visualizamos los datos. Los registros que se generan son similares a este

~~~~
{ "event_time" : "2024-06-12T15:08:32.04800, "ticker" : "INTC", "price" : 7 }
~~~~

#### 7. Escribir sentencias SQL - Kinesis paraa procesar los eventos. 

Para visualizar el comportamiento de los eventos, insertamos sentencias SQL que capturen los eventos deseados. Esto ya se explicó en la parte de teoría.

~~~~
Vamos al directorio ./python/GettingStarted

Ejecutamos
$ export IS_LOCAL=true
$ python main.py
~~~~

Podríamos ejecutar la aplicación directamente desde el servicio administrado para Apache Flink de Kinesis. 
Resumidamente, los pasos serían: 
1. Cargar la aplicación a un bucket de S3
2. ***Crear aplicación de streaming*** en el servicio ___"Aplicaciones Apache Flink"___

Todo esto está descrito en el apartado 
[Crear y configurar el servicio administrado para la aplicación Apache Flink](https://docs.aws.amazon.com/managed-flink/latest/java/gs-python-createapp.html#gs-python-7)

#### 8. Eliminar los DataStream de Kinesis. 

Podríamos hacerlo desde la consola de AWS o directamente desde AWS CLI.
~~~ 
aws kinesis delete-stream \
--stream-name ExampleInputStream 
~~~ 

~~~
aws kinesis delete-stream \
--stream-name ExampleOutputStream 
~~~