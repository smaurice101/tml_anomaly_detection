# TML and Anomaly Detection with Data Streams and AutoML
Pre-requisites:
1) MAADS-VIPER
2) MAADS-HPDE
3) Kafka cloud account (use Confluent Cloud)
4) Python
5) Python libraries
6) Beginner knowledge of Python, VIPER, HPDE, Kafka

```python
# Developed by: OTICS Advanced Analytics Inc.
# Date: 2021-01-18 
# Toronto, Ontario Canada
# For help email: support@otics.ca 

# import Python Libraries
import maadstml

# Uncomment IF using jupyter notebook
#import nest_asyncio

import json

# Uncomment IF using jupyter notebook
#nest_asyncio.apply()

# Set Global variables for VIPER and HPDE - You can change IP and Port for your setup of 
# VIPER and HPDE
VIPERHOST="http://127.0.0.1"
VIPERPORT=8000
hpdehost="http://127.0.0.1"
hpdeport=8001

# Set Global variable for Viper confifuration file - change the folder path for your computer
viperconfigfile="C:/viper/viper.env"

#############################################################################################################
#                                      STORE VIPER TOKEN
# Get the VIPERTOKEN from the file admin.tok - change folder location to admin.tok
# to your location of admin.tok
def getparams():
        
     with open("c:/viper/admin.tok", "r") as f:
        VIPERTOKEN=f.read()
  
     return VIPERTOKEN

VIPERTOKEN=getparams()

def performAnomalyDetection():
      #############################################################################################################
      #                                     JOIN DATA STREAMS 

      # Set personal data
      companyname="OTICS Advanced Analytics"
      myname="Sebastian"
      myemail="Sebastian.Maurice"
      mylocation="Toronto"

      # Joined topic name
      joinedtopic="joined-viper-test15"
      # Replication factor for Kafka redundancy
      replication=3
      # Number of partitions for joined topic
      numpartitions=3
      # Enable SSL/TLS communication with Kafka
      enabletls=1
      # If brokerhost is empty then this function will use the brokerhost address in your
      # VIPER.ENV in the field 'KAFKA_CONNECT_BOOTSTRAP_SERVERS'
      brokerhost=''
      # If this is -999 then this function uses the port address for Kafka in VIPER.ENV in the
      # field 'KAFKA_CONNECT_BOOTSTRAP_SERVERS'
      brokerport=-999
      # If you are using a reverse proxy to reach VIPER then you can put it here - otherwise if
      # empty then no reverse proxy is being used
      microserviceid=''

      streamstojoin="""viperdependentvariable,viperindependentvariable1,viperindependentvariable2,textdata1,textdata2"""

      description="Topic containing joined streams for Anomaly training"

      # Call MAADS python function to create joined stream topic
      result=maadstml.vipercreatejointopicstreams(VIPERTOKEN,VIPERHOST,VIPERPORT,joinedtopic,
                          streamstojoin,companyname,myname,myemail,description,mylocation,
                          enabletls,brokerhost,brokerport,replication,numpartitions,microserviceid)

      # Print the returned results
      print(result)
      # Load the results in JSON object and extract the producer ID
      try:
        y = json.loads(result,strict='False')
      except Exception as e:
        y = json.loads(result)
      topic=y['Topic']
      producerid=y['ProducerId']

      #############################################################################################################
      #                                    PRODUCE TO TOPIC STREAM

      # Roll back each data stream by 50 offsets - change this to a larger number if you want more data
      rollbackoffsets=50
      # Go to the last offset of each stream: If lastoffset=500, then this function will rollback the 
      # streams to offset=500-50=450
      startingoffset=-1
      # Max wait time for Kafka to response on milliseconds - you can increase this number if
      # Kafka takes longer to response.  Here we tell the functiont o wait 10 seconds
      delay=10000
      # Call the Python function to produce data from all the streams
      result=maadstml.viperproducetotopicstream(VIPERTOKEN,VIPERHOST,VIPERPORT,joinedtopic,producerid,
                                              startingoffset,rollbackoffsets,enabletls,delay,brokerhost,
                                              brokerport,microserviceid)

      # You can print the data - but it could be large amount of data 
      # The function returns a JSON object - you can load it in a Python variable
      # and store in the variable of your choosing - I chose Y
      try:
        y = json.loads(result,strict='False')
      except Exception as e:
        y = json.loads(result)

      # Get the partition by iterating through the JSON groups
      for elements in y:
        try:
          if 'Partition' in elements:
             stream_partition=elements['Partition'] 
        except Exception as e:
          continue

      #############################################################################################################
      #                                     SETUP TOPICS FOR PEER GROUP ANALYSIS

      description="Topic needed for peer group analysis"
      # Create a topic that will store peer group data
      producetotopic="anomalydatatest10"
      result=maadstml.vipercreatetopic(VIPERTOKEN,VIPERHOST,VIPERPORT,producetotopic,companyname,myname,
                                     myemail,mylocation,description,enabletls,
                                     brokerhost,brokerport,numpartitions,replication,microserviceid)
      # Load the JSON
      try:
        y = json.loads(result,strict='False')
      except Exception as e:
        y = json.loads(result)
      topic=y['Topic']
      # Get the producer id for this topic and save it in a variable
      produceridmain=y['ProducerId']
      print(produceridmain)

      # Create another topic to store the peer groups for anomaly prediction
      peergrouptotopic="anomalypeergroup"
      result=maadstml.vipercreatetopic(VIPERTOKEN,VIPERHOST,VIPERPORT,peergrouptotopic,companyname,
                                    myname,myemail,mylocation,description,enabletls,
                                    brokerhost,brokerport,numpartitions,replication,microserviceid)
      try:
        y = json.loads(result,strict='False')
      except Exception as e:
        y = json.loads(result)
      topic=y['Topic']
      # Get the producer id for this topic
      produceridpeergroup=y['ProducerId']
      print(produceridpeergroup)

      # Subscribe consumer to the topic just created with some information about yourself
      # If subscribing to a group and add group id here
      groupid=''
      description="This is a subscription for peer group analysis"
      result=maadstml.vipersubscribeconsumer(VIPERTOKEN,VIPERHOST,VIPERPORT,producetotopic,companyname,
                                          myname,myemail,mylocation,description,
                                          brokerhost,brokerport,groupid,microserviceid)
      print(result)
      # Load result in JSON object and extract the consumer id
      try:
        y = json.loads(result,strict='False')
      except Exception as e:
        y = json.loads(result)
      consumeridproduceto=y['Consumerid']
      print(consumeridproduceto)

      consumefrom = joinedtopic
      description="This is a subscription to consume from joined topic stream"
      result=maadstml.vipersubscribeconsumer(VIPERTOKEN,VIPERHOST,VIPERPORT,consumefrom,companyname,
                                          myname,myemail,mylocation,description,
                                          brokerhost,brokerport,groupid,microserviceid)
      print(result)
      # Load the JSON and extract the consumer id
      try:
        y = json.loads(result,strict='False')
      except Exception as e:
        y = json.loads(result)
      consumeridmain=y['Consumerid']
      consumeridjoinedtopic=consumeridmain
      print(consumeridmain,consumeridjoinedtopic)

      #############################################################################################################
      #                                     START ANOMALY TRAINING 
      # name the topic to produce to
      produceto = producetotopic
      # name the topic to produce peer group to
      producepeergroupto = peergrouptotopic
      # Assign the producer id of the peer group topic
      produceridpeergroup=produceridpeergroup
      # Assign the consumer id of the produceto topic
      consumeridproduceto=consumeridproduceto
      # Identify the streams to analyse for Anomalies
      streamstoanalyse="viperdependentvariable,viperindependentvariable1,viperindependentvariable2,textdata1,textdata2"
      # Assign the consumer id of the topic you are consuming the data for peer group analysis
      consumerid=consumeridmain
      # Assign the producer id you want to produce results to 
      producerid=produceridmain
      # Choose your flags for each of the stream variables
      flags="""topic=viperdependentvariable,topictype=numeric,threshnumber=300,lag=5,zthresh=2.5,
      influence=0.5~topic=viperindependentvariable1,topictype=numeric,threshnumber=300,lag=5,zthresh=2.5,
      influence=0.5~topic=viperindependentvariable2,topictype=numeric,threshnumber=300,lag=5,zthresh=2.5,
      influence=0.9~topic=textdata1,topictype=string,threshnumber=10~topic=textdata2,topictype=string,
      threshnumber=.80"""
      # Enable SSL/TLS
      enabletls=1
      # Assign the partition you extracted from the function: viperproducetotopicstream
      partition=stream_partition

      # Start Anomaly training 
      result=maadstml.viperanomalytrain(VIPERTOKEN,VIPERHOST,VIPERPORT,consumefrom,produceto,
                              producepeergroupto,produceridpeergroup,
                              consumeridproduceto, streamstoanalyse,companyname,consumerid,
                              producerid,flags,hpdehost,viperconfigfile, enabletls,partition,
                              hpdeport)
      # Load the JSON results and get the partition Kafka stored the peer groups to
      try:
        y = json.loads(result,strict='False')
      except Exception as e:
        y = json.loads(result)
      anomalytrain_partition=y['Partition'] 

      print("anomalytrain_partition=",anomalytrain_partition)

      ############################################################################################################
      #                                     SETUP TO PREDICT ANOMALIES

      # Assign the name of the topic to consume the peer groups from
      consumefrom = "anomalypeergroup"
      result=maadstml.vipersubscribeconsumer(VIPERTOKEN,VIPERHOST,VIPERPORT,consumefrom,companyname,
                                          myname,myemail,mylocation,description,
                                          brokerhost,brokerport,groupid,microserviceid)
      print(result)
      # Load the JSON object and extract the consumer id
      try:
        y = json.loads(result,strict='False')
      except Exception as e:
        y = json.loads(result)
      consumeridmainpredict=y['Consumerid']
      print(consumeridmainpredict)

      # Create a topic to store the anomaly results to- USE THIS TOPIC (anomalydataresults)
      # FOR VIPERviz visualization
      produceto="anomalydataresults"
      description="Topic to store the anomaly results"
      result=maadstml.vipercreatetopic(VIPERTOKEN,VIPERHOST,VIPERPORT,produceto,companyname,myname,
                                     myemail,mylocation,description,enabletls,
                                     brokerhost,brokerport,numpartitions,replication,microserviceid)
      # Load the JSON and extract the producer id
      try:
        y = json.loads(result,strict='False')
      except Exception as e:
        y = json.loads(result)
      topic=y['Topic']
      produceridmainpredict=y['ProducerId']
      print(produceridmainpredict)

      # Subscribe to the anomaly data results - YOU CAN USE THIS CONSUMER ID 
      # IN VIPERviz visualization
      result=maadstml.vipersubscribeconsumer(VIPERTOKEN,VIPERHOST,VIPERPORT,produceto,companyname,
                                          myname,myemail,mylocation,description,
                                          brokerhost,brokerport,groupid,microserviceid)
      print(result)
      # Load the JSON and extract the consumer id
      try:
        y = json.loads(result,strict='False')
      except Exception as e:
        y = json.loads(result)
      consumeridproducetopredict=y['Consumerid']
      print(consumeridproducetopredict)

      # Get the input stream topic - this is the topic in the function viperproducetotopicstream
      consumeinputstream = joinedtopic
      # Create a topic for the input stream - this is your test data for anomaly detection
      produceinputstreamtest="inputstreamdata"
      result=maadstml.vipercreatetopic(VIPERTOKEN,VIPERHOST,VIPERPORT,produceinputstreamtest,
                                    companyname,myname,myemail,mylocation,description,
                                    enabletls,brokerhost,brokerport,
                                    numpartitions,replication,microserviceid)
      # Load the JSON and get the producer id
      try:
        y = json.loads(result,strict='False')
      except Exception as e:
        y = json.loads(result)
      topic=y['Topic']
      produceridinputstreamtestpredict=y['ProducerId']
      print(produceridinputstreamtestpredict)

      ###########################################################################################################
      #                                  CREATE A CONSUMER GROUP
      #                       Use the Groupid in VIPERviz to consume from topic in parallel
      #                       across hundreds or thousands of consumers 
      consumergrouptopic="anomalydataresults"
      groupname="salesgroup"
      
      result=maadstml.vipercreateconsumergroup(VIPERTOKEN,VIPERHOST,VIPERPORT,consumergrouptopic,groupname,
                                      companyname,myname,myemail, description,mylocation,enabletls)
      print(result) 
      y = json.loads(result)
      groupid=y['Groupid']
      print(groupid)

      ############################################################################################################
      #                                     START ANOMALY PREDICTION

      # Name the topic to produce to
      consumeridproduceto=consumeridproducetopredict
      # Streams to analyse - we are analysing 5 streams - you can use any amount of streams
      streamstoanalyse="""viperdependentvariable,viperindependentvariable1,viperindependentvariable2,
      textdata1,textdata2"""

      # Assign variables
      consumerid=consumeridmainpredict
      producerid=produceridmainpredict

      # Specify flags 
      flags="""flags=riskscore=.4~complete=or~type=or,topic=viperdependentvariable,topictype=numeric,
      sc>500~type=and,topic=viperindependentvariable1,topictype=numeric,v1<100,sc>100~
      type=or,topic=textdata1,topictype=string,stringcontains=1,v2=valueany,sc>.6~type=or,
      topic=textdata2,topictype=string,stringcontains=0,v2=Failed Record^Failed Record^test record,
      sc>.210~type=or,topic=viperindependentvariable2,topictype=numeric,v1<100,sc>1000"""

      produceridinputstreamtest=produceridinputstreamtestpredict
      consumeridinputstream=consumeridjoinedtopic

      # Predict Anomalies
      result2=maadstml.viperanomalypredict(VIPERTOKEN,VIPERHOST,VIPERPORT,consumefrom,produceto,
                                        consumeinputstream,produceinputstreamtest,
                                        produceridinputstreamtest, streamstoanalyse, 
                                        consumeridinputstream,companyname,consumeridmainpredict,
                                        producerid,flags,hpdehost,viperconfigfile,enabletls,
                                        anomalytrain_partition, hpdeport)

      print(result2)

##########################################################################
# change this to any number
numanomalyruns=1000

for j in range(numanomalyruns):
  try:   
    performAnomalyDetection()
  except Exception as e:
    print(e)
    continue


```
