# IBM-Attrition-HR-Analytics
---
title: "IBM Attrition"
output:
  html_document: default
  word_document: default
---

```{r}
hrdata<- read.csv("WA_Fn-UseC_-HR-Employee-Attrition.csv")

str(hrdata)


```

```{r}
library(dplyr)

library(corrplot)
library(caTools)

```
Check for missing values,and other abnormalities & replacing output as int for time being
```{r}
sapply(hrdata,function(inp){return(sum(is.na(inp)))})
names(hrdata)[1]="Age"
hrdata$Attrition<-as.integer(hrdata$Attrition)
hrdata$Attrition<-hrdata$Attrition-2
hrdata$Attrition<-hrdata$Attrition*-1
hrdata$Attrition<-as.integer(hrdata$Attrition)
```
No missing values,
truncate the output of unique to study to find constant variables and find whether 
the class of variables are correct


```{r}
for(i in 1:35)
{if(length(unique(hrdata[,i]))<=10)
  { print(names(hrdata[i]))
  print(unique(hrdata[,i])) }}
```
Remove columns having constant values:
```{r}
hrdata<- select(hrdata,-EmployeeCount,-StandardHours,-Over18)
length(hrdata)
```
Environment satis,job involvement,job level,job satis,relationship satisfaction,stockoptionlevel,work life
should be factors instead of integers.
 
```{r}

hrdata$EnvironmentSatisfaction<-as.factor(hrdata$EnvironmentSatisfaction)
hrdata$JobInvolvement<-as.factor(hrdata$JobInvolvement)
hrdata$JobLevel<-as.factor(hrdata$JobLevel)
hrdata$JobSatisfaction<-as.factor(hrdata$JobSatisfaction)
hrdata$RelationshipSatisfaction<-as.factor(hrdata$RelationshipSatisfaction)
hrdata$StockOptionLevel<-as.factor(hrdata$StockOptionLevel)
hrdata$WorkLifeBalance<-as.factor(hrdata$WorkLifeBalance)
str(hrdata)


```

Finding the correlations for numeric values:
```{r}
ints<-c()

for(i in 1:32)
 {if(is.integer(hrdata[,i]))
{
ints<-c(ints,i)}}
corrplot(cor(hrdata[,ints]),method = "pie",number.font = 1)
cor(hrdata[,ints],hrdata$Attrition)
#for(i in 1:32)
#{print(names(hrdata[i])) 
 # print(cor(hrdata$Attrition,as.numeric(hrdata[,i])))}

```
Thus Attrition is positively correlated to Age,Monthly Income,Total Working Years,YearsAtCompany and YearsWithCurrentManager, and no significant negative correlation is found.


Chi-Square Analysis to find the significant variables:

```{r}
for(i in 1:length(hrdata))
  {if(class(hrdata[,i])!="integer")
   { tbl<-0
  tbl=table(hrdata$Attrition,hrdata[,i])
print(names(hrdata[i]))
print(chisq.test(tbl))}}
```
Comparision of expected attrition and actual attrition in each categories:

```{r}
pooled_prop=nrow(hrdata[hrdata$Attrition==1,])/nrow(hrdata)


for(k in 1:32)
{
if(is.factor(hrdata[,k]))
{props<-c()
prop_vector<-c()
nam<-c()  
  for(i in 1:length(unique(hrdata[,k])))
  {
    props<-nrow(hrdata[hrdata$Attrition==1 & hrdata[,k]==levels(hrdata[,k])[i],])/nrow(hrdata[hrdata[,k]==levels(hrdata[,k])[i],])
prop_vector<- c(prop_vector,props)  
   

se<-sqrt((pooled_prop*(1-pooled_prop))/nrow(hrdata)+(props*(1-props))/nrow(hrdata[hrdata[,k]==levels(hrdata[,k])[i],]))

}
prop_vector1<-c()
prop_vector1<-c(pooled_prop,prop_vector)
#print(names(hrdata[k]))
#print(result)
#if(result=="significant")
#{ 
#print(ifelse(z<z1,sprintf("Significantly Min Attrition in %s is in Category %i",names(hrdata[k]),i),sprintf("Significantly Max Attrition in %s is in Category %i",names(hrdata[k]),i)))
if(k>2)
{x<-barplot(prop_vector1,xlab = "Categories/Levels",ylab="Attrition",names.arg=c("Expected",levels(hrdata[,k])),main = names(hrdata[k]),axisnames=T,ylim = c(0,1),width = 1.5,cex.names = .6,space=0.7,col="blue")}}}

```


Splitting:
```{r}
hrdata$Attrition<-as.factor(hrdata$Attrition)
split<-sample.split(hrdata,SplitRatio = 0.8)
train<-subset(hrdata,split==TRUE)
testing<-subset(hrdata,split==FALSE)
```

```{r}
logistic<- glm(Attrition~.,data=train,family=binomial)
prob_pred<-predict(logistic,newdata=testing,type='response')
l_pred<-ifelse(prob_pred>0.5,1,0)
cm<- table(testing$Attrition,l_pred)
cm
accuracy<- (27+261)/(322)
accuracy
```


```{r}
logistic<- glm(train$Attrition~BusinessTravel+EnvironmentSatisfaction+MaritalStatus+JobInvolvement+JobRole+OverTime+WorkLifeBalance+Age+MonthlyIncome+TotalWorkingYears+YearsAtCompany+YearsWithCurrManager,data=train,family=binomial)

prob_pred1<-predict(logistic,newdata = testing,type = 'response')
l_pred<-ifelse(prob_pred1>0.5,1,0)
cm<- table(testing$Attrition,l_pred)
cm
accuracy<-(10+262)/nrow(testing)
accuracy
```



```{r}
library(rattle)
library(rpart.plot)
library(RColorBrewer)
decision_tree<- rpart(train$Attrition~.,data=train,method='class')
fancyRpartPlot(decision_tree)
decision<-predict(decision_tree,newdata=testing,type='class')
cm<-table(testing$Attrition,decision)
cm
accuracy<- (12+256)/nrow(testing)
accuracy
```
```{r}
library(randomForest)
RF<- randomForest(Attrition ~ .,data=train, importance=T,ntree=2000)
ensemble<- predict(RF,newdata=testing)
cm<-table(testing$Attrition,ensemble)
cm
accuracy<- (267+6)/nrow(testing)
accuracy
print(importance(RF,type = 2))
varImpPlot(RF)
```

```{r}
library(e1071)
model_svm<- svm(Attrition~.,train)
supportvm<-predict(model_svm,testing)
cm<- table(testing$Attrition,supportvm)
cm
accuracy<-(269+1)/nrow(testing)
accuracy
```

```{r}
model_nb<- naiveBayes(Attrition~.,train)
naiveb<-predict(model_nb,testing)
cm<- table(testing$Attrition,naiveb)
cm
accuracy<-(38+210)/nrow(testing)
accuracy
```
