# aocast
# ADAPTATIVE OPERATOR FORECAST
 
The purpose of this lib is to train and evaluate a state space reconstruction adaptive models for time-series prediction.  Using DEAP GA as optimization engine to calibrate the hyperparameters of the model

# EXAMPLE 1: HOW TO RUN AN ADAPTIVE OPERATOR
First you have to download 
numpy, pandas, matplotlib, sklearn, DEAP
```
>pip install --upgrade numpy
>pip install --upgrade scipy
>pip install --upgrade pandas
>pip install --upgrade matplotlib
>pip install --upgrade sklearn
>pip install --upgrade DEAP
```
and aocast
```
>pip install --upgrade aocast

```
# RUN A SIMPLE EXAMPLE
```

import aocast as ac
import numpy as np
import pandas as pd


#CREATE ADAPTATIVE OPERATOR
ao=ac.AdaptativeOperator()


# =============================================================================
# Simple sin data
# =============================================================================
t = np.linspace((-np.pi), (np.pi), 1000).reshape(-1,1)
y=np.sin(t*100)
x1=np.sin(t*0.1)

window=100
lc=np.asarray([2, 1])

delta=2
crossValidations=100

target,simulation=ao.predict(y,x1,lc,window,delta,crossValidations)
ao.plotPerformance(target,simulation,delta)

```
# RUN A FORECAST EXAMPLE
```

import aocast as ac
import numpy as np
import pandas as pd


#CREATE ADAPTATIVE OPERATOR
ao=ac.AdaptativeOperator()

# =============================================================================
# Load the hydrological data
# =============================================================================
rootDir="https://raw.githubusercontent.com/felipeardilac/adaptcast/master/"
file="FLOWS_MAG.csv"
#file="LEVEL.csv"
url=rootDir+file

data_df =pd.read_csv(url, index_col=0, parse_dates=True)

##Fill the missing the data
dataInterp=ao.interpolate(data_df.values)
data= dataInterp


targetIndex=9

inputData= data
inputData= np.delete(data, [targetIndex], axis=1)

target_name=list(data_df)[targetIndex]        
print("TARGET : ",target_name)
targetData= data[:,targetIndex].reshape(-1,1)



# =============================================================================
# Define the parameters
# =============================================================================
# Window of the adaptive operator
minWindow=12*4
maxWindow=12*4+6*10
windowLimits=np.array([minWindow,maxWindow])
#Forecast horizon
maxLag=5

numberCrossVal=12*3
#Forecast horizon
delta=2

ngen=15
popSize=500

# =============================================================================
# Run a single step forecast
# =============================================================================

#FIT ADAPTATIVE OPERATOR
best_windowSize,best_lagConf=ao.fit(targetData,inputData,maxLag,windowLimits,delta,numberCrossVal,ngen,popSize)


#APPLY OPERATOR
target,simulation=ao.predict(targetData,inputData,best_lagConf,best_windowSize,delta,numberCrossVal)

#PLOT PERFORMANCE
ao.plotPerformance(target,simulation,delta)

# =============================================================================
# Run several steps ahead forecasts
# =============================================================================
stepsAhead=3

forecasts,error=ao.predictSteps(targetData,inputData,stepsAhead,maxLag,windowLimits,numberCrossVal,ngen,popSize)

ao.plotSteps(targetData,forecasts,error)
