![Logo_compuesto_vof_2025](https://github.com/user-attachments/assets/df7db686-cfb0-4b47-a125-8cc28e402332)

Este repositorio fue utilizado para mostrar el uso de **Hadoop y Spark** (con Docker) para la materia *Herramientas Avanzadas para el Manejo de Grandes Volúmenes de Datos* de la Licenciatura en Ciencia de Datos para Negocios (LCDN) de la Universidad Nacional Rosario Castellanos (UNRC).

# Docker + Hadoop + PySpark

Contenedor de Docker con Hadoop (HDFS) y Spark. Pero sin los grandes requisitos de memoria de un sandbox de Cloudera.  

## Pre-requisitos
Tener Docker Desktop instalado para Windows y MacOs. Para el caso de Google Chromebook debe tener instalado Docker (vía terminal) 

**Observación:**  
Los comandos utilizados en este repositorio se ejecutaron en los siguientes sistemas operativos (+ Docker):  
* _Windows 11_,  
* _macOS_ (con chip Intel y M2), y  
* Google Chromebook.

## Clonar repositorio
#### Opción 1 (Manual)
 1.- Hacer click en el botón __<> CODE__  
 2.- Hacer click en el botón __Download ZIP__   
<img width="407" alt="image" src="https://github.com/user-attachments/assets/ec0e5f1e-3254-4d2b-9b69-f19c88e9c205" />

#### Opción 2 (Terminal) 
Ejecutar  
```
  git clone https://github.com/mevangelista-alvarado/docker_hadoop_spark.git 
```

## Hadoop Docker
Para ejecutar los siguientes comandos, debe ubicarse en la carpeta que contenga el archivo `docker-compose.yml`.

### Iniciar el contenedor
Para desplegar el ejemplo de un clúster HDFS, ejecute en la consola de su computadora (según el caso, si es la primera vez o no) lo siguiente:

#### Instalar el contenedor por primera vez
```
docker-compose up
```

#### Levantar el contenedor, si ya está instalado
```
docker-compose start
```

#### Verificar que el contenedor está trabajando
Para comprobar que el contenedor está en funcionamiento:
```
docker-compose ps
```
Si el estado es `UP`, entonces el contenedor está trabajando correctamente. De lo contrario, ha ocurrido un error al inicializarlo.

#### Apagar el contenedor
```
docker-compose stop
```
**Observación:**  
Acceda a las interfaces de Hadoop con las siguientes URL:

* Namenode: http://localhost:9870/dfshealth.html#tab-overview
* Datanode: http://localhost:9864/datanode.html
* Spark master: http://localhost:8080/
* Spark worker: http://localhost:8081/

## Quick Start HDFS
Entramos a Hadoop
```
    docker exec -it namenode bash
```

Para crear una carpeta de HDFS, seguimos lo siguiente: 
 * `hdfs dfs -ls /` (Ver que contenido hay en HDFS)
 * `hdfs dfs -mkdir -p /user/data` (Crear el directorio /user/data)
 * `exit` (Salir del nodo)

Agregamos los siguientes archivos en HDFS
 * `el_quijote.txt` 
 * `afluenciastc_desglosado_04_2025.csv`
 * `breweries.csv`

#### Archivo `el_quijote.txt` 
1.- Copiamos el archivo de nuestra computadora al contendor de Docker
```
    docker cp ./datasets/el_quijote.txt namenode:/tmp
```  
2.- Entramos a Hadoop
```
    docker exec -it namenode bash
```  
3.- Ingresamos a la memorial temporal del nodo 
```
    cd /tmp
```
4.- Verficamos que el archivo se encuentra en la memoria temporal del nod
```
    ls
```
5.- mMovemos el archivo a Hadoop con HDFS
```
    hdfs dfs -put el_quijote.txt /user/data
```  
6.- Verificamos que el archivo se movio correctamente a Hadoop
```
    hdfs dfs -ls /user/data
```
7.- Finalmente, salimos del nodo
```
    exit 
```

#### Archivo `afluenciastc_desglosado_04_2025.csv`
0.- Debido que Github, no permite alojar archivos pesado, descargamos el archivo `afluenciastc_desglosado_04_2025.csv` haciendo [click aquí](https://drive.google.com/file/d/11_s7fHDGGuRmaeteG6Z5-FHYWN0ZByn5/view?usp=sharing).  

Movemos el archivo al directorio `\datasets` de nuestro repo.

1.- Copiamos el archivo de nuestra computadora al contendor de Docker
```
    docker cp ./datasets/afluenciastc_desglosado_04_2025.csv namenode:/tmp
```  
2.- Entramos a Hadoop
```
    docker exec -it namenode bash
```  
3.- Ingresamos a la memorial temporal del nodo 
```
    cd /tmp
```
4.- Verficamos que el archivo se encuentra en la memoria temporal del nod
```
    ls
```
5.- Movemos el archivo a Hadoop con HDFS
```
    hdfs dfs -put afluenciastc_desglosado_04_2025.csv /user/data
```  
6.- Verificamos que el archivo se movio correctamente a Hadoop
```
    hdfs dfs -ls /user/data
```
7.- Finalmente, salimos del nodo
```
    exit 
```

#### Archivo `breweries.csv`
1.- Copiamos el archivo de nuestra computadora al contendor de Docker
```
    docker cp ./datasets/breweries.csv namenode:/tmp
```  
2.- Entramos a Hadoop
```
    docker exec -it namenode bash
```  
3.- Ingresamos a la memorial temporal del nodo 
```
    cd /tmp
```
4.- Verficamos que el archivo se encuentra en la memoria temporal del nod
```
    ls
```
5.- Movemos el archivo a Hadoop con HDFS
```
    hdfs dfs -put breweries.csv /user/data
```  
6.- Verificamos que el archivo se movio correctamente a Hadoop
```
    hdfs dfs -ls /user/data
```
7.- Finalmente, salimos del nodo
```
    exit 
```

## Quick Start PySpark
Vaya a [http://localhost:8080/](http://localhost:8080/) en su computadora portátil para ver el estado del maestro Spark.  

Entramos al nodo de Spark master
```
    docker exec -it spark-master bash
```

#### 1.- Ejemplo de contar palabras
0.- Ingresamos a línea de comandos de Spark master e iniciamos PySpark.
```
    /spark/bin/pyspark --master spark://spark-master:7077 --conf spark.pyspark.python=python3
```  

1.- Obtenemos los datos desde Hadoop
```
    text_df = spark.read.text("hdfs://namenode:9000/user/data/el_quijote.txt")
```  

2.- Convierte el df en un RDD (Resilient Distributed Dataset) y aplica una función flatMap
```
    words = text.rdd.flatMap(lambda line: line.value.split())
```  

3.- Toma cada palabra de la variable RDD words y cuenta cada repetición   
 * ```
    word_pairs = words.map(lambda word: (word, 1))
   ```
 * ```
    word_counts = word_pairs.reduceByKey(lambda a, b: a + b)
   ```
 * ```
    word_counts.collect()
   ```
 * Guardamos el resultado en en archivo de HDFS
   ```
     word_counts.saveAsTextFile("hdfs://namenode:9000/user/data/output/el_quijote_wc.txt")
   ```

4.- Mostramos el top 10  
 * ```
   top_10 = word_counts.takeOrdered(10, key=lambda x: -x[1])
   ```
 * ```
   top_10
   ``` 
#### 2.- Ejemplo de como manipular un dataframe (breweries.csv)
0.- Ingresamos a línea de comandos de Spark master e iniciamos PySpark.
```
    /spark/bin/pyspark --master spark://spark-master:7077 --conf spark.pyspark.python=python3
```  

1.- Obtenemos los datos desde Hadoop
```
    df = spark.read.csv("hdfs://namenode:9000/user/data/breweries.csv", header=True)
```

2.- Mostrar los primeros elementos de df
```
    df.show()
```

3.- Obtener la longitud del df (similar shape en pandas)
 * ```
   num_filas = df.count()
   ```
 * ```
   num_columnas = len(df.columns)
   ```

4.- Obtener valores únicos de una columna
```
    from pyspark.sql.functions import col
    unicos = df.select(col("city")).distinct()
    unicos.show()
```

5.- Filtrado simple
```
    from pyspark.sql.functions import col
    _df = df.filter(col("city") == "Grand Rapids")
```  
Ó usando una expresión SQL directamente como string
```
   _df = df.filter("city = 'Grand Rapids'")
```

6.- Filtrado Combinado
```
    from pyspark.sql.functions import col
    _df = df.filter((col("city") == "Grand Rapids") & (col("state") == "MI"))
    _df.show()
```

#### 3.- Ejemplo de como manipular un dataframe (afluenciastc_desglosado_04_2025.csv.csv)
0.- Ingresamos a línea de comandos de Spark master e iniciamos PySpark.
```
    /spark/bin/pyspark --master spark://spark-master:7077 --conf spark.pyspark.python=python3
```  

1.- Obtenemos los datos desde Hadoop
```
    df = spark.read.csv("hdfs://namenode:9000/user/data/afluenciastc_desglosado_04_2025.csv", header=True)
```

2.- Mostrar los primeros elementos de df
```
    df.show()
```

3.- Obtener la longitud del df (similar shape en pandas)
 * ```
   num_filas = df.count()
   ```
 * ```
   num_columnas = len(df.columns)
   ```

4.- Obtener valores únicos de una columna
```
    from pyspark.sql.functions import col
    unicos = df.select(col("linea")).distinct()
    unicos.show()
```

5.- Filtrado simple
```
    from pyspark.sql.functions import col
    _df = df.filter(col("linea") == "LÃ­nea 1")
```  
Ó usando una expresión SQL directamente como string
```
   _df = df.filter("linea = 'LÃ­nea 1'")
```

6.- Filtrado Combinado
```
    from pyspark.sql.functions import col
    _df = df.filter((col("linea") == "LÃ­nea 1") & (col("anio") == "2024"))
    _df.show()
```

#### 4.- Ejemplo de como ejecutar un  script
**Obs**  
Debe tener creado un directorio `/user/output` en HDFS  

0.- Movemos el script para ejcutar en Spark
 * Creamos una carpeta de scripts
   ```
    mkdir scripts
   ```
 * Salimos del nodo
   ```
   exit
   ```
 * Copiamos el script a Spark Master
   ```
   docker cp ./scripts/el_quijote_wordcount.py spark-master:/scripts/
   ```
1.- Entramos al nodo de Spark master
```
    docker exec -it spark-master bash
```  
2.- Ejecutamos el script
```
   /spark/bin/spark-submit --conf spark.pyspark.python=python3 /scripts/el_quijote_wordcount.py
```

## Referencias

 * https://colab.research.google.com/github/bdm-unlu/2020/blob/master/guias/Guia_IntroSpark.ipynb#scrollTo=GVnGWSdGBiQ5
 * https://colab.research.google.com/github/gibranfp/CursoDatosMasivosI/blob/main/notebooks/2a_pyspark_colab.ipynb#scrollTo=rRSrQMP9LrVI
 * https://github.com/Marcel-Jan/docker-hadoop-spark/tree/master

