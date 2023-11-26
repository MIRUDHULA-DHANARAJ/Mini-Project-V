## INDOOR AIR QUALITY MODELING BASED ON IOT 

The "Indoor Air Quality Modeling Based on IoT" project aims to implement a robust monitoring system using IoT devices to continuously assess and model indoor air quality parameters. The hardware includes selected sensors like the MQ series for gas detection and DHT for temperature and humidity, with a focus on calibration for accurate measurements. Data acquisition occurs in real-time, employing communication protocols like MQTT for transmission to a central hub. The project emphasizes data analysis and modeling, incorporating machine learning techniques to predict indoor air quality trends. A user-friendly interface displays real-time metrics and historical trends, accompanied by an alert system notifying users of parameter deviations. Power management strategies and thorough testing ensure system reliability. Documentation covers user manuals and technical details, while potential enhancements include integration with external systems for automated adjustments. This project not only addresses the immediate need for indoor air quality monitoring but also sets the groundwork for future expansions and improvements.


## FEATURES

* Continuous real-time assessment of indoor air quality parameters.
* Utilization of diverse sensors for accurate gas, temperature, and humidity measurements.
* Seamless communication for remote access and monitoring of air quality data.
* Advanced algorithms for predicting indoor air quality trends.
* Intuitive dashboard for easy visualization of metrics and trends.
* Timely notifications for deviations from predefined air quality thresholds.
* ptimization for energy-efficient and continuous operation.

## HARDWARE REQUIREMENTS
1. PM2.5 GP2Y1010AU0F Dust Smoke Particle Sensor
2. ESP8266 NodeMCU Board
3. Mq7 Carbon-monoxide Sensor
4. DHT11 Temperature & Humidity Sensor
5. Connecting wires & Connectors & BreadBoard

## SOFTWARE REQUIREMENTS 
1. Arduino IDE
2. ThingSpeak Platform

## ARCHITECTURE DIAGRAM

![MP](https://github.com/MIRUDHULA-DHANARAJ/Mini-Project-V/assets/94828147/241a8ed8-8a43-497f-bba8-f89f70bf1614)

## METHODOLOGY

1. Sensor Data Collection:
 Install and configure the sensor kit consisting of the
PM2.5 dust smoke sensor, MQ7 sensor (for carbon monoxide), DHT11 sensor (for
temperature and humidity), and Arduino node. The sensors collect real-time data
on PM2.5 levels, carbon monoxide levels, temperature, and humidity in the indoor
environment.

2. Data Transmission to Thingspeak:
 Connect the Arduino node to the internet and
transmit the sensor data to the Thingspeak IoT platform. Thingspeak acts as a data
repository and provides a web API for data retrieval and analysis. 

3. Dataset Creation and Preprocessing:
Retrieve the sensor data from Thingspeak
and create a dataset for IAQ prediction. Preprocess the dataset by handling missing
values, outliers, and performing data normalization if required. Additionally,
engineer relevant features that could potentially improve prediction accuracy.

4. Model Selection, Training & Evaluation:
 Choose a hybrid model for IAQ
prediction that combines Random Forest and Logistic Regression. The Random
Forest algorithm can handle non-linear relationships and capture complex
interactions, while Logistic Regression can provide interpretability and handle
binary classification tasks. Split the pre-processed dataset into training and testing
sets. Train the hybrid model using the training set. Evaluate the performance of the
trained hybrid model using the testing set

5. Prediction and Labelling:
 Deploy the trained hybrid model to make predictions
on new, unseen data. Input the real-time sensor values (PM2.5, carbon monoxide,
temperature, humidity) into the model, and obtain the predicted IAQ label (good,
bad, moderate). These labels can indicate the quality of the indoor air at a given
time.

6. Continuous Monitoring and Feedback:
Continuously monitor the IAQ
predictions and compare them with actual measurements. Collect feedback on the
accuracy and reliability of the predictions and incorporate it into the system. This
feedback loop helps to refine and improve the hybrid model over time. 

## PROGRAM
## Hardware program :
```
#include <ESP8266WiFi.h>
#include <SPI.h>
#include <Wire.h>
String apiKey = "VOTEBNCMF5IJJ26L";
const char *ssid = "vasanths 11R 5G";
const char *pass = "j772xt5q";
const char* server = "api.thingspeak.com";
int measurePin = A0;
int ledPower = 2;
int samplingTime = 280;
int deltaTime = 40;
int sleepTime = 9680;
float voMeasured = 0;
float calcVoltage = 0;
float dustDensity = 0;
const int DOUTpin=5;
int limit;
WiFiClient client;
void setup()
{
Serial.begin(9600);
pinMode(ledPower,OUTPUT);
pinMode(DOUTpin, OUTPUT);
WiFi.begin(ssid, pass);
while (WiFi.status() != WL_CONNECTED)
{
delay(500);
Serial.print(".");
}
Serial.println("");
Serial.println("WiFi connected");
}
int analgoread1()
{
digitalWrite(DOUTpin,LOW);
return analogRead(0);
}
void loop()
{
digitalWrite(ledPower,LOW); // power on the LED
delayMicroseconds(samplingTime);
voMeasured = analogRead(measurePin); // read the dust value
delayMicroseconds(deltaTime);
digitalWrite(ledPower,HIGH); // turn the LED off
delayMicroseconds(sleepTime);
calcVoltage = voMeasured * (5.0 / 1024.0);
dustDensity = 170 * calcVoltage - 0.1;
limit=analgoread1();
Serial.print("dustDensity ppm"); // unit: ug/m3
Serial.println(dustDensity); // unit: ug/m3
Serial.print("MQ7 ppm ");
Serial.println(limit); // unit: ug/m3
if (client.connect(server, 80)) // "184.106.153.149" or api.thingspeak.com
{
String postStr = apiKey;
postStr += "&field1=";
postStr += String(dustDensity);
postStr += "r\n";
postStr += "&field2=";
postStr += String(limit);
postStr += "r\n";
client.print("POST /update HTTP/1.1\n");
client.print("Host: api.thingspeak.com\n");
client.print("Connection: close\n");
client.print("X-THINGSPEAKAPIKEY: " + apiKey + "\n");
client.print("Content-Type: application/x-www-form-urlencoded\n");
client.print("Content-Length: ");
client.print(postStr.length());
client.print("\n\n");
client.print(postStr);
Serial.println("Data Send to Thingspeak");
}
client.stop();
Serial.println("Waiting...");
delay(2000); // thingspeak needs minimum
}
```
## Software program :
```
import numpy as np
import pandas as pd
import warnings
warnings.filterwarnings('ignore')
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestRegressor
from sklearn import metrics
from sklearn.metrics import
mean_absolute_error,mean_squared_error,r2_score,accuracy_score
from sklearn.linear_model import LogisticRegression
data=pd.read_csv('data.csv',encoding="ISO-8859-1")
data.fillna(0, inplace=True)
data.head()
data.shape
#data understanding
data.info()
data.isnull().sum()
data.describe()
data.nunique()
data.columns
#Function to calculate co individual pollutant index(coi)
def calculate_coi(co):
coi=0
if co<= 4.4:
coi = co * 50 / 4.4
elif co <= 9.4:
coi = 50 + ((co - 4.4) * 50 / 5)
elif co <= 12.4:
coi = 100 + ((co - 9.4) * 100 / 3)
elif co <= 15.4:
coi = 200 + ((co - 12.4) * 100 / 3)
else:
coi = 300 + ((co - 15.4) * 100 / 4.6)
return coi
data['coi']=data['CO'].apply(calculate_coi)
df= data[['CO','coi']]
df.head()
#Function to calculate pm2.5 individual pollutant index(pm25i)
def calculate_pm25i(pm25_concentration):
if pm25_concentration <= 12:
pm25i = pm25_concentration * 50 / 12
elif pm25_concentration <= 35.4:
pm25i = 50 + ((pm25_concentration - 12) * 50 / 23.4)
elif pm25_concentration <= 55.4:
pm25i = 100 + ((pm25_concentration - 35.4) * 100 / 20)
elif pm25_concentration <= 150.4:
pm25i = 200 + ((pm25_concentration - 55.4) * 100 / 95)
elif pm25_concentration <= 250.4:
pm25i = 300 + ((pm25_concentration - 150.4) * 100 / 100)
elif pm25_concentration <= 350.4:
pm25i = 400 + ((pm25_concentration - 250.4) * 100 / 100)
elif pm25_concentration <= 500.4:
pm25i = 500 + ((pm25_concentration - 350.4) * 100 / 150)
else:
pm25i = None
return pm25i
data['pm25i']=data['pm2_5'].apply(calculate_pm25i)
df= data[['pm2_5','pm25i']]
df.head()
def calculate_aqi(pm25i, coi, temperature, humidity):
aqi=0
if(pm25i>coi and pm25i>temperature and pm25i>humidity):
aqi=pm25i
if(coi>pm25i and coi>temperature and coi>humidity):
aqi=coi
return aqi
data['AQI']=data.apply(lambda
x:calculate_aqi(x['pm25i'],x['coi'],x['Temperature'],x['Humidity']),axis=1)
df= data[['pm25i','coi','Temperature','Humidity','AQI']]
df.head()
# Calculating Air Quality Index
def calculate_aqi(pm25i, coi, temperature, humidity):
aqi=0
if(pm25i>coi and pm25i>temperature and pm25i>humidity):
aqi=pm25i
if(coi>pm25i and coi>temperature and coi>humidity):
aqi=coi
return aqi
data['AQI']=data.apply(lambda
x:calculate_aqi(x['pm25i'],x['coi'],x['Temperature'],x['Humidity']),axis=1)
df= data[['pm25i','coi','Temperature','Humidity','AQI']]
df.head()
def AQI_Range(x):
if x<=50:
return "Good"
elif x>50 and x<=100:
return "Moderate"
elif x>100 and x<=200:
return "Poor"
elif x>200 and x<=300:
return "Unhealthy"
elif x>300 and x<=400:
return "Very Unhealthy"
elif x>400:
return "Hazardous"
df['AQI_Range']=data['AQI'].apply(AQI_Range)
df.head()
df['AQI_Range'].value_counts()
#Splitting dataset into dependent and independent column
X=df[['pm25i','coi','Temperature','Humidity']]
Y=df['AQI']
X_train,X_test,Y_train,Y_test=train_test_split(X,Y,test_size=0.2,random_state
=70)
print(X_train.shape,X_test.shape,Y_train.shape,Y_test.shape)
# RandomForestRegressor
RF=RandomForestRegressor().fit(X_train,Y_train)
#predicting train
Train_preds1=RF.predict(X_train)
#predicting test
Test_preds1=RF.predict(X_test)
RMSE_train=(np.sqrt(metrics.mean_squared_error(Y_train, Train_preds1)))
RMSE_test=(np.sqrt(metrics.mean_squared_error(Y_test, Test_preds1)))
print("RMSE TrainingData=", str(RMSE_train))
print("RMSE TestData=", str(RMSE_test))
print('-'*50)
print('RSquared value on train:',RF.score(X_train, Y_train))
print('RSquared value on test:',RF.score (X_test, Y_test))
# Logistic Regression
Train_preds2 =log_reg.predict(X_train2)
print("Model accuracy on train is: ",accuracy_score(Y_train2, Train_preds2))
Test_preds2 =log_reg.predict(X_test2)
print("Model accuracy on test is: ", accuracy_score(Y_test2,Test_preds2))
print('-'*50)
print("kappaScore is: ",metrics.cohen_kappa_score(Y_test2,Test_preds2))
## Testing with random values
log_reg.predict([[723.3,456.4,77.8,92.4]])
log_reg.predict([[200.3,46.4,77.8,92.4]]) 

```
## OUTPUT

## Pollutant ThingSpeak grapH 

![OP1](https://github.com/MIRUDHULA-DHANARAJ/Mini-Project-V/assets/94828147/b06b3943-eeb0-479a-9b6f-8c52ffefa92b)

## AQI Index AND AQI Range 

![OP2](https://github.com/MIRUDHULA-DHANARAJ/Mini-Project-V/assets/94828147/8dd7dab3-c3c0-4405-b057-695835e790fd)

## AQI Prediction 

![OP3](https://github.com/MIRUDHULA-DHANARAJ/Mini-Project-V/assets/94828147/f2faf5e0-1c11-4684-90c2-4409ecc4afd9)


## RESULT

The Indoor Air Quality (IAQ) prediction system offers real-time monitoring and visualization of key parameters like PM2.5, carbon monoxide (CO), temperature, and humidity, providing immediate feedback on indoor air quality. Users can analyze historical trends to track changes and identify patterns in air quality parameters. Leveraging a hybrid model, such as Random Forest and Logistic Regression, the system generates IAQ predictions classifying air quality as good, bad, or moderate. Evaluation metrics like accuracy, precision, recall, and F1 score gauge the model's performance. Additionally, system performance is assessed in terms of seamless data collection, transmission, processing speed, and responsiveness, ensuring reliability in real-world scenarios.

