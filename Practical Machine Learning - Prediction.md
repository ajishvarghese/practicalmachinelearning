## Practical Machine Learning - Prediction Project
========================================================
The goal of the project is to use Weight Lifing Exercise data gathered from accelerometers on the belt, forearm, arm, and dumbell of 6 participants to classify barbell lifts into 5 different types of doing the exercise correctly or incorrectly. To achieve this goal, I will build a prediction model using the random forests algorithm which is widely recognised for its high accuracy for these type of classification problems.
 
## Approach
========================================================
### Dataset Exploration and Cleaning 
The first step is to examine the training and testing (prediction) datasets. 

```r
datatr<-read.csv("pml-training.csv")
datatst<-read.csv("pml-testing.csv")
```
The datasets have 19622 and 20 observations of 160 variables.


```r
dim(datatr)
```

```
## [1] 19622   160
```

```r
dim(datatst)
```

```
## [1]  20 160
```
Both have the same variables except for the last one, the classe response variable in the training set but problem_id in the prediction set.

```r
which(!names(datatr)==names(datatst))
```

```
## [1] 160
```

```r
names(datatr)[160]
```

```
## [1] "classe"
```

```r
names(datatst)[160]
```

```
## [1] "problem_id"
```
Using the summary function on the datasets indicate a many sparsely populated columns with the occurences of 'NA', blanks and '#DIV/0!' (which should be converted to NA for analysis). The sparseness is examined to determine its extent and the further action warranted.  

```r
fullcoltr<-apply(datatr,2,function(x) sum(!is.na(x)))
nacoltr<-apply(datatr,2,function(x) sum(is.na(x)))
fullrowtr<-apply(datatr,1,function(x) sum(!is.na(x)))
narowtr<-apply(datatr,1,function(x) sum(is.na(x)))
unique(fullcoltr)
```

```
## [1] 19622   406
```

```r
unique(nacoltr)
```

```
## [1]     0 19216
```

```r
unique(fullrowtr)
```

```
## [1]  93 160
```

```r
unique(narowtr) 
```

```
## [1] 67  0
```
67 variables in the training set are NA for all observations, the other observations are fully populated except for 406 cases.

```r
fullcoltst<-apply(datatst,2,function(x) sum(!is.na(x)))
nacoltst<-apply(datatst,2,function(x) sum(is.na(x)))
fullrowtst<-apply(datatst,1,function(x) sum(!is.na(x)))
narowtst<-apply(datatst,1,function(x) sum(is.na(x)))
unique(fullcoltst)
```

```
## [1] 20  0
```

```r
unique(nacoltst)
```

```
## [1]  0 20
```

```r
unique(fullrowtst)
```

```
## [1] 60
```

```r
unique(narowtst)
```

```
## [1] 100
```
The prediction set has 100/160 NA columns for the 20 observations. Therefore, the choice of predictor columns can be narrowed down to the fully populated columns present in the prediction (and training) sets.  

```r
data<-read.csv("pml-training.csv",na.strings = c("NA","","#DIV/0!"))
p_data<-read.csv("pml-testing.csv",na.strings = c("NA","","#DIV/0!"))
fullcoldata<-apply(data,2,function(x) sum(!is.na(x)))
colsel<-which(fullcoldata==nrow(data))
fullcolpdata<-apply(p_data,2,function(x) sum(!is.na(x)))
colselp<-which(fullcolpdata==nrow(p_data))
```
Again, we see that both sets have 60 fully populated variables including the last one (classe/problem_id) that is different between the two data sets. Also, the first seven variables are not related to the accelerometer data and excluded as predictors.

```r
length(colsel)
```

```
## [1] 60
```

```r
sum(colsel==colselp)
```

```
## [1] 60
```

```r
sum(names(colsel)==names(colselp))
```

```
## [1] 59
```

```r
names(colsel)[1:7]
```

```
## [1] "X"                    "user_name"            "raw_timestamp_part_1"
## [4] "raw_timestamp_part_2" "cvtd_timestamp"       "new_window"          
## [7] "num_window"
```

```r
colsel<-colsel[-c(1:7)]
```
The datasets can now be subset to get the data sets to be used for model building and prediction.

```r
compldata<-data[,colsel]
complp_data<-p_data[,colsel]
```
### Model building 
The caret package is used for data slicing into training and building the random forest classification model including cross-validation. The training data is subset into two data sets (60/40, with seed set for reproducibility) into a new 'training' set to train the model and an independent 'test' set to provide an unbiased validation of the model and an estimate of the out of sample error.

```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
set.seed(1011)
inTrain = createDataPartition(compldata$classe, p = 0.6)[[1]]
training = compldata[ inTrain,]
testing = compldata[-inTrain,]
```
The data set has 52 predictor variables whose interrelationships (such as collinearity) and predictive potential are not known.Random forests is able to handle such cases with a high degree of accuracy. Moreover, it automatically calculates the variable importance in prediction and includes cross-validation during the forest building progress, calculating the out of bag (oob) error rate. Further pre-processing of data is not necessary in this method.

```r
modFitrf <- train(classe ~ .,data=training,method="rf",prox=FALSE)
```

```
## Loading required package: randomForest
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
```

```r
modFitrf
```

```
## Random Forest 
## 
## 11776 samples
##    52 predictor
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Bootstrapped (25 reps) 
## 
## Summary of sample sizes: 11776, 11776, 11776, 11776, 11776, 11776, ... 
## 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy   Kappa      Accuracy SD  Kappa SD   
##    2    0.9846881  0.9806284  0.002383020  0.003001932
##   27    0.9864338  0.9828370  0.002473017  0.003126635
##   52    0.9770509  0.9709670  0.004810608  0.006091112
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was mtry = 27.
```
### Model evaluation
The model used all 52 predictors and an mtry of 27 predictors in the random forest, with an overall Kappa of 0.9864. Applying the trained model to predicting the 'classe' on the testing set gives an out of sample error of just under 0.78%. The plot of variable importance is also obtained from the random forest as seen below.


```r
prf <- predict(modFitrf,testing)
ooserr <- sum(testing$classe!=prf)/nrow(testing)
paste("Out of sample error: ", ooserr)
```

```
## [1] "Out of sample error:  0.00777466224827938"
```

```r
plot(varImp(modFitrf), main="Variable ImportancePlot", ylab="Variables")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png) 
