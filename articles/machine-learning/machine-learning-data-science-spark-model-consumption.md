<properties
	pageTitle="Pontuar modelos de aprendizado de máquina criados no Spark | Microsoft Azure"
	description="Como pontuar modelos de aprendizado que foram armazenados no WASB (Armazenamento de Blobs do Azure)."
	services="machine-learning"
	documentationCenter=""
	authors="bradsev"
	manager="paulettm"
	editor="cgronlun" />

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/19/2016"
	ms.author="deguhath;bradsev" />

# Pontuar modelos de aprendizado de máquina criados no Spark 

[AZURE.INCLUDE [machine-learning-spark-modeling](../../includes/machine-learning-spark-modeling.md)]


## Introdução

Este tópico descreve como carregar modelos de AM (aprendizado de máquina) que foram criados usando o MLlib Spark e armazenados no WASB (Armazenamento de Blobs do Azure) e pontuá-los com conjuntos de dados que também foram armazenados no WASB. Ele mostra como pré-processar dados de entrada, transformar os recursos usando as funções de indexação e de codificação no kit de ferramentas MLlib e criar um objeto de dados de ponto rotulado que pode ser usado como entrada para pontuação com os modelos de AM. Os modelos usados para pontuação incluem regressão linear, regressão logística, modelos de florestas aleatórias e de árvore de aumento gradiente.


## Pré-requisitos

1. Você precisa de uma conta do Azure e um cluster HDInsight Spark para iniciar este passo a passo. Confira os requisitos na [Visão geral de ciência de dados usando o Spark no Azure HDInsight](machine-learning-data-science-spark-overview.md) para obter uma descrição dos dados NYC 2013 Taxi usados aqui e para obter instruções sobre como executar código de um bloco de anotações do Jupyter no cluster Spark. O bloco de anotações **machine-learning-data-science-spark-model-consumption.ipynb** que contém os exemplos de código neste tópico estão disponíveis no [Github](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/Spark/Python).

2. Você também tem que criar modelos de aprendizado de máquina que serão pontuados aqui de acordo com o tópico [Exploração e modelagem de dados com Spark](machine-learning-data-science-spark-data-exploration-modeling.md).


[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]
 

## Configurar Spark e caminhos de diretório para dados e modelos armazenados 

O Spark é capaz de ler e gravar em um WASB (blob de armazenamento do Azure). Portanto, qualquer dado existente armazenado lá pode ser processado usando o Spark, e os resultados podem ser armazenados novamente no WASB.

Para salvar arquivos ou modelos no WASB, o caminho deve ser especificado corretamente. O contêiner padrão anexado ao cluster Spark pode ser referenciado usando uma caminho que começa com: *"wasb / /"*. O exemplo de código a seguir especifica o local dos dados a serem lidos e o caminho do diretório de armazenamento de modelo em que a saída do modelo será salva.


### Definir caminhos de diretório para locais de armazenamento no WASB

Os modelos são salvos em: "wasb:///user/remoteuser/NYCTaxi/Models". Se esse caminho não estiver definido corretamente, os modelos não serão carregados para pontuação.

Os resultados de pontuação foram salvos em: "wasb:///user/remoteuser/NYCTaxi/ScoredResults". Se o caminho para a pasta estiver incorreto, os resultados não serão salvos nessa pasta.

>AZURE.NOTE: os locais de caminho de arquivo podem ser copiados e colados em espaços reservados nesse código da saída da última célula do bloco de anotações **machine-learning-data-science-spark-data-exploration-modeling.ipynb**.

	# LOCATION OF DATA TO BE SCORED (TEST DATA)
	taxi_test_file_loc = "wasb://mllibwalkthroughs@cdspsparksamples.blob.core.windows.net/Data/NYCTaxi/JoinedTaxiTripFare.Point1Pct.Test.tsv";
	
	# SET THE MODEL STORAGE DIRECTORY PATH 
	# NOTE THE LAST BACKSLASH IN THIS PATH IS NEEDED
	modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/" 
	
	# SET SCORDED RESULT DIRECTORY PATH
	# NOTE THE LAST BACKSLASH IN THIS PATH IS NEEDED
	scoredResultDir = "wasb:///user/remoteuser/NYCTaxi/ScoredResults/"; 
	
	# FILE LOCATIONS FOR THE MODELS TO BE SCORED
	logisticRegFileLoc = modelDir + "LogisticRegressionWithLBFGS_2016-04-1817_40_35.796789"
	linearRegFileLoc = modelDir + "LinearRegressionWithSGD_2016-04-1817_44_00.993832"
	randomForestClassificationFileLoc = modelDir + "RandomForestClassification_2016-04-1817_42_58.899412"
	randomForestRegFileLoc = modelDir + "RandomForestRegression_2016-04-1817_44_27.204734"
	BoostedTreeClassificationFileLoc = modelDir + "GradientBoostingTreeClassification_2016-04-1817_43_16.354770"
	BoostedTreeRegressionFileLoc = modelDir + "GradientBoostingTreeRegression_2016-04-1817_44_46.206262"

	# RECORD START TIME
	import datetime
	datetime.datetime.now()

**SAÍDA:**

datetime.datetime(2016, 4, 19, 17, 21, 28, 379845)


### Importar bibliotecas necessárias e definir o contexto do Spark 

Defina o contexto do Spark e importe as bibliotecas necessárias com o código a seguir

	#IMPORT LIBRARIES
	import pyspark
	from pyspark import SparkConf
	from pyspark import SparkContext
	from pyspark.sql import SQLContext
	from pyspark.sql import Row
	from pyspark.sql.functions import UserDefinedFunction
	from pyspark.sql.types import *
	import atexit
	from numpy import array
	import numpy as np
	import datetime
	
	# SET SPARK CONTEXT
	sc = SparkContext(conf=SparkConf().setMaster('yarn-client'))
	sqlContext = SQLContext(sc)
	atexit.register(lambda: sc.stop())
	
	sc.defaultParallelism

**SAÍDA:**

4


## Ingerir dados e criar um quadro de dados limpo

Esta seção contém o código para uma série de tarefas necessárias para ingerir os dados a serem pontuados. Leia um exemplo de 0,1% associado do arquivo de corrida de táxi e tarifa (armazenado como um arquivo. tsv), formate os dados e crie um quadro de dados limpo.

Os arquivos de corrida de táxi e tarifa foram associados com base no procedimento fornecido no tópico: [O Processo do Cortana Analytics em ação: usando clusters Hadoop do HDInsight](machine-learning-data-science-process-hive-walkthrough.md).

	# INGEST DATA AND CREATE A CLEANED DATA FRAME

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# IMPORT FILE FROM PUBLIC BLOB
	taxi_test_file = sc.textFile(taxi_test_file_loc)
	
	# GET SCHEMA OF THE FILE FROM HEADER
	taxi_header = taxi_test_file.filter(lambda l: "medallion" in l)
	
	# PARSE FIELDS AND CONVERT DATA TYPE FOR SOME FIELDS
	taxi_temp = taxi_test_file.subtract(taxi_header).map(lambda k: k.split("\t"))\
	        .map(lambda p: (p[0],p[1],p[2],p[3],p[4],p[5],p[6],int(p[7]),int(p[8]),int(p[9]),int(p[10]),
	                        float(p[11]),float(p[12]),p[13],p[14],p[15],p[16],p[17],p[18],float(p[19]),
	                        float(p[20]),float(p[21]),float(p[22]),float(p[23]),float(p[24]),int(p[25]),int(p[26])))
	    
	# GET SCHEMA OF THE FILE FROM HEADER
	schema_string = taxi_test_file.first()
	fields = [StructField(field_name, StringType(), True) for field_name in schema_string.split('\t')]
	fields[7].dataType = IntegerType() #Pickup hour
	fields[8].dataType = IntegerType() # Pickup week
	fields[9].dataType = IntegerType() # Weekday
	fields[10].dataType = IntegerType() # Passenger count
	fields[11].dataType = FloatType() # Trip time in secs
	fields[12].dataType = FloatType() # Trip distance
	fields[19].dataType = FloatType() # Fare amount
	fields[20].dataType = FloatType() # Surcharge
	fields[21].dataType = FloatType() # Mta_tax
	fields[22].dataType = FloatType() # Tip amount
	fields[23].dataType = FloatType() # Tolls amount
	fields[24].dataType = FloatType() # Total amount
	fields[25].dataType = IntegerType() # Tipped or not
	fields[26].dataType = IntegerType() # Tip class
	taxi_schema = StructType(fields)
	
	# CREATE DATA FRAME
	taxi_df_test = sqlContext.createDataFrame(taxi_temp, taxi_schema)
	
	# CREATE A CLEANED DATA-FRAME BY DROPPING SOME UN-NECESSARY COLUMNS & FILTERING FOR UNDESIRED VALUES OR OUTLIERS
	taxi_df_test_cleaned = taxi_df_test.drop('medallion').drop('hack_license').drop('store_and_fwd_flag').drop('pickup_datetime')\
	    .drop('dropoff_datetime').drop('pickup_longitude').drop('pickup_latitude').drop('dropoff_latitude')\
	    .drop('dropoff_longitude').drop('tip_class').drop('total_amount').drop('tolls_amount').drop('mta_tax')\
	    .drop('direct_distance').drop('surcharge')\
	    .filter("passenger_count > 0 and passenger_count < 8 AND payment_type in ('CSH', 'CRD') AND tip_amount >= 0 AND tip_amount < 30 AND fare_amount >= 1 AND fare_amount < 150 AND trip_distance > 0 AND trip_distance < 100 AND trip_time_in_secs > 30 AND trip_time_in_secs < 7200" )
	
	# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
	taxi_df_test_cleaned.cache()
	taxi_df_test_cleaned.count()
	
	# REGISTER DATA-FRAME AS A TEMP-TABLE IN SQL-CONTEXT
	taxi_df_test_cleaned.registerTempTable("taxi_test")
	
	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SAÍDA:**

Tempo necessário para executar acima da célula: 15,36 segundos


## Preparar dados para pontuação no Spark 

Esta seção mostra como indexar, codificar e dimensionar recursos categóricos a fim de prepará-los para uso em algoritmos de aprendizado supervisionado de MLlib para classificação e regressão.

### Transformação de recurso: indexe e codifique recursos categóricos para entrada em modelos de pontuação 

Esta seção mostra como indexar dados categóricos usando recursos `StringIndexer` e codificar com entrada `OneHotEncoder` para os modelos.

O [StringIndexer](http://spark.apache.org/docs/latest/ml-features.html#stringindexer) codifica uma coluna de cadeia de caracteres de rótulos em uma coluna de índices de rótulo. Os índices são ordenados por frequências de rótulo.

O [OneHotEncoder](http://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) mapeia uma coluna de índices de rótulo para uma coluna de vetores binários com no máximo um valor único. Essa codificação permite que os algoritmos que esperam recursos valiosos contínuos, como a regressão logística, sejam aplicados em recursos categóricos.
	
	#INDEX AND ONE-HOT ENCODE CATEGORICAL FEATURES

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.ml.feature import OneHotEncoder, StringIndexer, VectorAssembler, OneHotEncoder, VectorIndexer
	
	# CREATE FOUR BUCKETS FOR TRAFFIC TIMES
	sqlStatement = """
	    SELECT *,
	    CASE
	     WHEN (pickup_hour <= 6 OR pickup_hour >= 20) THEN "Night" 
	     WHEN (pickup_hour >= 7 AND pickup_hour <= 10) THEN "AMRush" 
	     WHEN (pickup_hour >= 11 AND pickup_hour <= 15) THEN "Afternoon"
	     WHEN (pickup_hour >= 16 AND pickup_hour <= 19) THEN "PMRush"
	    END as TrafficTimeBins
	    FROM taxi_test 
	"""
	taxi_df_test_with_newFeatures = sqlContext.sql(sqlStatement)
	
	# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
	taxi_df_test_with_newFeatures.cache()
	taxi_df_test_with_newFeatures.count()
	
	# INDEX AND ONE-HOT ENCODING
	stringIndexer = StringIndexer(inputCol="vendor_id", outputCol="vendorIndex")
	model = stringIndexer.fit(taxi_df_test_with_newFeatures) # Input data-frame is the cleaned one from above
	indexed = model.transform(taxi_df_test_with_newFeatures)
	encoder = OneHotEncoder(dropLast=False, inputCol="vendorIndex", outputCol="vendorVec")
	encoded1 = encoder.transform(indexed)
	
	# INDEX AND ENCODE RATE_CODE
	stringIndexer = StringIndexer(inputCol="rate_code", outputCol="rateIndex")
	model = stringIndexer.fit(encoded1)
	indexed = model.transform(encoded1)
	encoder = OneHotEncoder(dropLast=False, inputCol="rateIndex", outputCol="rateVec")
	encoded2 = encoder.transform(indexed)
	
	# INDEX AND ENCODE PAYMENT_TYPE
	stringIndexer = StringIndexer(inputCol="payment_type", outputCol="paymentIndex")
	model = stringIndexer.fit(encoded2)
	indexed = model.transform(encoded2)
	encoder = OneHotEncoder(dropLast=False, inputCol="paymentIndex", outputCol="paymentVec")
	encoded3 = encoder.transform(indexed)
	
	# INDEX AND ENCODE TRAFFIC TIME BINS
	stringIndexer = StringIndexer(inputCol="TrafficTimeBins", outputCol="TrafficTimeBinsIndex")
	model = stringIndexer.fit(encoded3)
	indexed = model.transform(encoded3)
	encoder = OneHotEncoder(dropLast=False, inputCol="TrafficTimeBinsIndex", outputCol="TrafficTimeBinsVec")
	encodedFinal = encoder.transform(indexed)
	
	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SAÍDA:**

Tempo necessário para executar acima da célula: 4,88 segundos


### Crie objetos RDD com matrizes de recurso para entrada em modelos

Esta seção contém código que mostra como indexar dados de texto categóricos como objeto RDD e fazer codificação one-hot para que ele possa ser usado no treinamento e no teste de regressão logística MLlib e modelos baseados em árvore. Os dados indexados são armazenados em objetos [RDD (conjunto de dados distribuído resiliente)](http://spark.apache.org/docs/latest/api/java/org/apache/spark/rdd/RDD.html). Essas são as abstrações básicas no Spark. Um objeto RDD representa uma coleção imutável e particionada de elementos que pode ser operada em paralelo com o Spark.

Ele também contém o código que mostra como dimensionar dados com o `StandardScalar` fornecido por MLlib para uso na regressão linear com SGD (Stochastic Gradient Descent), um algoritmo popular de treinamento de uma grande variedade de modelos de aprendizado de máquina. O [StandardScaler](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.feature.StandardScaler) é usado para dimensionar os recursos de acordo com a variação de unidade. O dimensionamento de recursos, também conhecido como normalização de dados, faz com que recursos com valores amplamente distribuídos não tenham peso excessivo na função objetiva.


	# CREATE RDD OBJECTS WITH FEATURE ARRAYS FOR INPUT INTO MODELS

	# RECORD START TIME
	timestart = datetime.datetime.now()

	# IMPORT LIBRARIES
	from pyspark.mllib.linalg import Vectors
	from pyspark.mllib.feature import StandardScaler, StandardScalerModel
	from pyspark.mllib.util import MLUtils
	from numpy import array
	
	# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
	def parseRowIndexingBinary(line):
	    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex,
	                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
	                         line.trip_distance, line.fare_amount])
	    return  features
	
	# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO LOGISTIC RERESSION MODELS
	def parseRowOneHotBinary(line):
	    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
	                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
	                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
	                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
	    return  features
	
	# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
	def parseRowIndexingRegression(line):
	    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex, 
	                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
	                         line.trip_distance, line.fare_amount])
	    return  features
	
	# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO LINEAR REGRESSION MODELS
	def parseRowOneHotRegression(line):
	    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
	                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
	                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
	                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
	    return  features

	# FOR BINARY CLASSIFICATION TRAINING AND TESTING
	indexedTESTbinary = encodedFinal.map(parseRowIndexingBinary)
	oneHotTESTbinary = encodedFinal.map(parseRowOneHotBinary)
	
	# FOR REGRESSION CLASSIFICATION TRAINING AND TESTING
	indexedTESTreg = encodedFinal.map(parseRowIndexingRegression)
	oneHotTESTreg = encodedFinal.map(parseRowOneHotRegression)
	
	# SCALING FEATURES FOR LINEARREGRESSIONWITHSGD MODEL
	scaler = StandardScaler(withMean=False, withStd=True).fit(oneHotTESTreg)
	oneHotTESTregScaled = scaler.transform(oneHotTESTreg)
	
	# CACHE RDDS IN MEMORY
	indexedTESTbinary.cache();
	oneHotTESTbinary.cache();
	indexedTESTreg.cache();
	oneHotTESTreg.cache();
	oneHotTESTregScaled.cache();
	
	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SAÍDA:**

Tempo necessário para executar acima da célula: 9,94 segundos


## Pontue com o modelo de regressão logística e salve a saída em um blob

O código nesta seção mostra como carregar um modelo de regressão logística salvo no armazenamento de blobs do Azure e usá-lo para prever se uma gorjeta é dada ou não em uma viagem de táxi, pontuá-lo com métricas de classificação padrão, salvar e plotar os resultados no armazenamento de blobs. Os resultados de pontuação são armazenados em objetos RDD.


	# SCORE AND EVALUATE LOGISTIC REGRESSION MODEL

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# IMPORT LIBRARIES
	from pyspark.mllib.classification import LogisticRegressionModel
	
	## LOAD SAVED MODEL
	savedModel = LogisticRegressionModel.load(sc, logisticRegFileLoc)
	predictions = oneHotTESTbinary.map(lambda features: (float(savedModel.predict(features))))
	
	## SAVE SCORED RESULTS (RDD) TO BLOB
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	logisticregressionfilename = "LogisticRegressionWithLBFGS_" + datestamp + ".txt";
	dirfilename = scoredResultDir + logisticregressionfilename;
	predictions.saveAsTextFile(dirfilename)
	
	
	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds";

**SAÍDA:**

Tempo necessário para executar acima da célula: 32,46 segundos


## Classificar um modelo de regressão linear

Usamos [LinearRegressionWithSGD](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.regression.LinearRegressionWithSGD) no treinamento de um modelo de regressão linear usando SGD (Stochastic Gradient Descent) para otimização a fim de prever o valor das gorjetas pagas.

O código nesta seção mostra como carregar um modelo de regressão linear do armazenamento de blobs do Azure, pontuá-lo usando variáveis em escala e salvar os resultados no blob.

	#SCORE LINEAR REGRESSION MODEL

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	#LOAD LIBRARIES​
	from pyspark.mllib.regression import LinearRegressionWithSGD, LinearRegressionModel
	
	# LOAD MODEL AND SCORE USING ** SCALED VARIABLES **
	savedModel = LinearRegressionModel.load(sc, linearRegFileLoc)
	predictions = oneHotTESTregScaled.map(lambda features: (float(savedModel.predict(features))))
	
	# SAVE RESULTS
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	linearregressionfilename = "LinearRegressionWithSGD_" + datestamp;
	dirfilename = scoredResultDir + linearregressionfilename;
	predictions.saveAsTextFile(dirfilename)
	
	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL​
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**SAÍDA:**

Tempo necessário para executar acima da célula: 25,00 segundos


## Pontue modelos de florestas aleatórias de classificação e regressão

O código nesta seção mostra como carregar os modelos de florestas aleatórias de regressão e classificação salvos no armazenamento de blobs do Azure, pontuar seu desempenho com métricas de regressão e classificadoras padrão e salvar os resultados no armazenamento de blobs.

[Florestas aleatórias](http://spark.apache.org/docs/latest/mllib-ensembles.html#Random-Forests) são conjuntos de árvores de decisão. Elas combinam várias árvores de decisão para reduzir o risco de superajuste. As florestas aleatórias podem lidar com recursos categóricos e estender à configuração de classificação multiclasse, não exigem o dimensionamento de recursos e são capazes de capturar não linearidades e interações de recursos. As florestas aleatórias são um dos modelos de aprendizado de máquina com maior taxa de sucesso para classificação e regressão.

[spark.mllib](http://spark.apache.org/mllib/) dá suporte a florestas aleatórias na classificação binária, multiclasse e regressão, usando recursos contínuos e categóricos.

	# SCORE RANDOM FOREST MODELS FOR CLASSIFICATION AND REGRESSION

	# RECORD START TIME
	timestart = datetime.datetime.now()

	#IMPORT MLLIB LIBRARIES	
	from pyspark.mllib.tree import RandomForest, RandomForestModel
	
	
	# CLASSIFICATION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB
	savedModel = RandomForestModel.load(sc, randomForestClassificationFileLoc)
	predictions = savedModel.predict(indexedTESTbinary)
	
	# SAVE RESULTS
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	rfclassificationfilename = "RandomForestClassification_" + datestamp + ".txt";
	dirfilename = scoredResultDir + rfclassificationfilename;
	predictions.saveAsTextFile(dirfilename)
	

	# REGRESSION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB
	savedModel = RandomForestModel.load(sc, randomForestRegFileLoc)
	predictions = savedModel.predict(indexedTESTreg)
	
	# SAVE RESULTS
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	rfregressionfilename = "RandomForestRegression_" + datestamp + ".txt";
	dirfilename = scoredResultDir + rfregressionfilename;
	predictions.saveAsTextFile(dirfilename)

	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds";

**SAÍDA:**

Tempo necessário para executar acima da célula: 52,2 segundos


## Pontue modelos de árvore de aumento gradiente de classificação e regressão

O código nesta seção mostra como carregar modelos de aumento gradiente de classificação e regressão do armazenamento de blobs do Azure, pontuar seu desempenho com métricas de regressão e classificação padrão e salvar os resultados novamente no armazenamento de blobs.

**spark.mllib** dá suporte a GBTs para classificação binária e de regressão usando recursos contínuos e categóricos.

[GBTs (árvores de aumento gradiente)](http://spark.apache.org/docs/latest/ml-classification-regression.html#gradient-boosted-trees-gbts) são conjuntos de árvores de decisão. As GBTs treinam árvores de decisão iterativamente para minimizar uma função de perda. As GBTs podem manipular recursos categóricos, não exigem o dimensionamento de recursos e são capazes de capturar não linearidades e interações de recursos. Elas também podem ser usadas em uma configuração de classificação multiclasse.


	# SCORE GRADIENT BOOSTING TREE MODELS FOR CLASSIFICATION AND REGRESSION

	# RECORD START TIME
	timestart = datetime.datetime.now()

	#IMPORT MLLIB LIBRARIES
	from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel
	
	# CLASSIFICATION:LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB

	#LOAD AND SCORE THE MODEL
	savedModel = GradientBoostedTreesModel.load(sc, BoostedTreeClassificationFileLoc)
	predictions = savedModel.predict(indexedTESTbinary)
	
	# SAVE RESULTS
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	btclassificationfilename = "GradientBoostingTreeClassification_" + datestamp + ".txt";
	dirfilename = scoredResultDir + btclassificationfilename;
	predictions.saveAsTextFile(dirfilename)
	

	# REGRESSION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB

	# LOAD AND SCORE MODEL 
	savedModel = GradientBoostedTreesModel.load(sc, BoostedTreeRegressionFileLoc)
	predictions = savedModel.predict(indexedTESTreg)
	
	# SAVE RESULTS
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	btregressionfilename = "GradientBoostingTreeRegression_" + datestamp + ".txt";
	dirfilename = scoredResultDir + btregressionfilename;
	predictions.saveAsTextFile(dirfilename)


	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
	
**SAÍDA:**

Tempo necessário para executar acima da célula: 27,73 segundos

## Limpe objetos da memória e imprima a localização de arquivos pontuados

	# UNPERSIST OBJECTS CACHED IN MEMORY
	taxi_df_test_cleaned.unpersist()
	indexedTESTbinary.unpersist();
	oneHotTESTbinary.unpersist();
	indexedTESTreg.unpersist();
	oneHotTESTreg.unpersist();
	oneHotTESTregScaled.unpersist();


	# PRINT OUT PATH TO SCORED OUTPUT FILES
	print "logisticRegFileLoc: " + logisticregressionfilename;
	print "linearRegFileLoc: " + linearregressionfilename;
	print "randomForestClassificationFileLoc: " + rfclassificationfilename;
	print "randomForestRegFileLoc: " + rfregressionfilename;
	print "BoostedTreeClassificationFileLoc: " + btclassificationfilename;
	print "BoostedTreeRegressionFileLoc: " + btregressionfilename;


**SAÍDA:**

logisticRegFileLoc: LogisticRegressionWithLBFGS\_2016-04-1917\_22\_36.354603.txt

linearRegFileLoc: LinearRegressionWithSGD\_2016-04-1917\_23\_06.083178

randomForestClassificationFileLoc: RandomForestClassification\_2016-04-1917\_23\_33.994108.txt

randomForestRegFileLoc: RandomForestRegression\_2016-04-1917\_24\_00.352683.txt

BoostedTreeClassificationFileLoc: GradientBoostingTreeClassification\_2016-04-1917\_24\_21.465683.txt

BoostedTreeRegressionFileLoc: GradientBoostingTreeRegression\_2016-04-1917\_24\_32.371641.txt



## Consumir modelos do Spark por meio de uma interface Web

O Spark fornece um mecanismo para enviar remotamente trabalhos em lotes ou consultas interativas por meio de uma interface REST com um componente chamado Livy. O Livy está habilitado por padrão no cluster Spark no HDInsight. Para saber mais sobre o Livy, confira: [Enviar trabalhos em Spark remotamente usando o Livy](../hdinsight/hdinsight-apache-spark-livy-rest-interface.md).

Você pode usar Livy para enviar um trabalho que pontua em lotes um arquivo armazenado em um blob do Azure e grava os resultados em outro blob. Para fazer isso, carregue o script em Python do [Github](https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/Spark/Python/ConsumeGBNYCReg.py) no blob do cluster Spark. Você pode usar uma ferramenta como o **Explorador do Armazenamento do Microsoft Azure** ou o **AzCopy** para copiar o script no blob de cluster. Em nosso caso, carregamos o script em ***wasb:///example/python/ConsumeGBNYCReg.py***.

>AZURE.NOTE: as chaves de acesso de que você precisa podem ser encontradas no portal da conta de armazenamento associada ao cluster Spark.

Uma vez carregado nesse local, o script será executado dentro do cluster Spark em um contexto distribuído. Ele carregará o modelo e executará previsões em arquivos de entrada baseado no modelo.

Você pode chamar esse script remotamente fazendo uma simples solicitação HTTPS/REST no Livy. Aqui está um comando para construir a solicitação HTTP a fim de invocar o script em Python remotamente. Você precisa substituir CLUSTERLOGIN, CLUSTERPASSWORD, CLUSTERNAME pelos valores apropriados para seu cluster Spark.


	# CURL COMMAND TO INVOKE PYTHON SCRIPT WITH HTTP REQUEST

    curl -k --user "CLUSTERLOGIN:CLUSTERPASSWORD" -X POST --data "{"file": "wasb:///example/python/ConsumeGBNYCReg.py"}" -H "Content-Type: application/json" https://CLUSTERNAME.azurehdinsight.net/livy/batches

Você pode usar qualquer linguagem no sistema remoto para invocar o trabalho do Spark com o Livy fazendo uma simples chamada HTTPS com a autenticação básica.

>AZURE.NOTE: é conveniente usar a biblioteca de solicitações de Python ao fazer essa chamada HTTP, mas, atualmente, ela não é instalada por padrão no Azure Functions. Portanto, em vez disso, são usadas bibliotecas HTTP mais antigas.

Aqui está o código do Python para a chamada HTTP:

	#MAKE AN HTTPS CALL ON LIVY. 

	import os

	# OLDER HTTP LIBRARIES USED HERE INSTEAD OF THE REQUEST LIBRARY AS THEY ARE AVAILBLE BY DEFAULT
	import httplib, urllib, base64
	
	# REPLACE VALUE WITH ONES FOR YOUR SPARK CLUSTER
	host = '<spark cluster name>.azurehdinsight.net:443'
	username='<username>'
	password='<password>'
	
	#AUTHORIZATION
	conn = httplib.HTTPSConnection(host)
	auth = base64.encodestring('%s:%s' % (username, password)).replace('\n', '')
	headers = {'Content-Type': 'application/json', 'Authorization': 'Basic %s' % auth}
	
	# SPECIFY THE PYTHON SCRIPT TO RUN ON THE SPARK CLUSTER
	# IN THE FILE PARAMETER OF THE JSON POST REQUEST BODY
	r=conn.request("POST", '/livy/batches', '{"file": "wasb:///example/python/ConsumeGBNYCReg.py"}', headers )
	response = conn.getresponse().read()
	print(response)
	conn.close()


Você também pode adicionar esse código Python ao [Azure Functions](../functions/) para disparar um envio de trabalho do Spark que pontua um blob com base em vários eventos, como um timer, criação ou atualização de um blob.

Se você preferir uma experiência de cliente gratuito de código, use os [Aplicativos Lógicos do Azure](../app-service/logic/) para invocar a pontuação em lote do Spark definindo uma ação HTTP no **Designer de aplicativos lógicos** e definindo seus parâmetros.

- No Portal do Azure, crie um novo aplicativo lógico selecionando **+Novo** -> **Web + Móvel** -> **Aplicativo Lógico**. 
- Insira o nome do aplicativo lógico e o plano de serviço de aplicativo para abrir o **Designer de aplicativos lógicos**.
- Selecione uma ação HTTP e insira os parâmetros mostrados na figura abaixo:

![](./media/machine-learning-data-science-spark-model-consumption/spark-logica-app-client.png)

<!---HONumber=AcomDC_0420_2016-->