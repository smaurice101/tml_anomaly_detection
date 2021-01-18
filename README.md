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

# Produce Data to Kafka Cloud
import maads
import nest_asyncio
import json
import random

nest_asyncio.apply()

```


```python
# Set Global Host/Port for VIPER - You may change this to fit your configuration
VIPERHOST="http://192.168.0.13"
VIPERPORT=8000

#############################################################################################################
#                                      STORE VIPER TOKEN
# Get the VIPERTOKEN from the file admin.tok - change folder location to admin.tok
# to your location of admin.tok
def getparams():
        
     with open("c:/maads/golang/go/bin/admin.tok", "r") as f:
        VIPERTOKEN=f.read()
  
     return VIPERTOKEN

VIPERTOKEN=getparams()
#############################################################################################################
```


```python
#############################################################################################################
#                                     CREATE TOPICS IN KAFKA

# Set personal data
companyname="OTICS Advanced Analytics"
myname="Sebastian"
myemail="Sebastian.Maurice"
mylocation="Toronto"

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

description="test data"
producetotopic="viperdependentvariable"
result3=maads.vipercreatetopic(VIPERTOKEN,VIPERHOST,VIPERPORT,producetotopic,companyname,
                               myname,myemail,mylocation,description,enabletls,
                               brokerhost,brokerport,numpartitions,replication,
                               microserviceid)
y = json.loads(result3)
producetotopic=y['Topic']
producerid1=y['ProducerId']
print(producerid1)


# First Create a topic to produce to
producetotopic="viperindependentvariable1"
result3=maads.vipercreatetopic(VIPERTOKEN,VIPERHOST,VIPERPORT,producetotopic,companyname,
                               myname,myemail,mylocation,description,enabletls,
                               brokerhost,brokerport,numpartitions,replication,
                               microserviceid)
y = json.loads(result3)
producetotopic=y['Topic']
producerid2=y['ProducerId']
print(producerid2)


# First Create a topic to produce to
producetotopic="viperindependentvariable2"
result3=maads.vipercreatetopic(VIPERTOKEN,VIPERHOST,VIPERPORT,producetotopic,companyname,
                               myname,myemail,mylocation,description,enabletls,
                               brokerhost,brokerport,numpartitions,replication,
                               microserviceid)
y = json.loads(result3)
producetotopic=y['Topic']
producerid3=y['ProducerId']
print(producerid3)

# First Create a topic to produce to
producetotopic="textdata1"
result3=maads.vipercreatetopic(VIPERTOKEN,VIPERHOST,VIPERPORT,producetotopic,companyname,
                               myname,myemail,mylocation,description,enabletls,
                               brokerhost,brokerport,numpartitions,replication,
                               microserviceid)
y = json.loads(result3)
producetotopic=y['Topic']
producerid4=y['ProducerId']
print(producerid4)

# First Create a topic to produce to
producetotopic="textdata2"
result3=maads.vipercreatetopic(VIPERTOKEN,VIPERHOST,VIPERPORT,producetotopic,companyname,
                               myname,myemail,mylocation,description,enabletls,
                               brokerhost,brokerport,numpartitions,replication,
                               microserviceid)
y = json.loads(result3)
producetotopic=y['Topic']
producerid5=y['ProducerId']
print(producerid5)
```

    ProducerId-vTXnd4FJqfmBahbLPMjjFxIrvUqR11
    ProducerId-iq2YUiU6IbIzAOsAlTI95njaaHyWgk
    ProducerId-GEjx2o7jkfA4aezkEnGCowGlt2yTv0
    ProducerId-ULfsIr3jwn-I3Gvf7sCMLmULOwNMdl
    ProducerId-0fa7s8Fojf2Qkv1kDIIF5w1tVxuVVV
    


```python
#############################################################################################################
#                                     PRODUCE External Value to TOPIC
# produce to Topic streams

topics=["viperdependentvariable","viperindependentvariable1","viperindependentvariable2","textdata1","textdata2"]
producerids=[producerid1,producerid2,producerid3,producerid4,producerid5]

tx1=["One advanced","diverted", "domestic repeated bringing you old.", "Possible", "procured trifling laughter", "thoughts"]
    
# change this number to whatever you wish - note cloud charges may apply
numberofdatapoints=100

for j in range(numberofdatapoints):  
    for t,p in zip(topics,producerids):
      if t!="textdata1" and t!="textdata2":  
        num=str(random.randrange(1000)) 
        result=maads.viperproducetotopic(VIPERTOKEN,VIPERHOST,VIPERPORT,t,p,1,1000,'','', '',0,num)
      else:
        # write text data
         num1=random.randrange(5)
         result=maads.viperproducetotopic(VIPERTOKEN,VIPERHOST,VIPERPORT,t,p,1,1000,'','', '',0,tx1[num1])
        
        
    #print(result)
    
    
```


```python

```
