  ## Marketing Data Predictive Analysis


``` r
responders_data_sheet <- read.csv(file="data.csv",header=TRUE)
```

Normality test for responders.


``` r
target_1_ages <- responders_data_sheet[responders_data_sheet[,7] == 1,2]
hist(target_1_ages)
```

<div align="center">
  <img src="https://github.com/user-attachments/assets/4838ad45-d769-4632-9b29-83d3dbc12c18" alt="unnamed-chunk-2-1">
</div>

``` r
shapiro.test(target_1_ages)
```

```
## 
## 	Shapiro-Wilk normality test
## 
## data:  target_1_ages
## W = 0.97781, p-value = 3.19e-11
```

``` r
qqnorm(target_1_ages)
qqline(target_1_ages, col = "red")
```
<div align="center">
  <img src="https://github.com/user-attachments/assets/7dce4876-f090-462d-9cf3-e9ce5a336f49" alt="unnamed-chunk-2-2">
</div>


reject null hypothesis that the sample is normally distributed.

Normality test for non responders.


``` r
target_0_ages <- responders_data_sheet[responders_data_sheet[,7] == 0,2]
hist(target_0_ages)
```
<div align="center">
  <img src="https://github.com/user-attachments/assets/4a2aeba6-6756-4463-822b-bf943235647d" alt="unnamed-chunk-3-1">
</div>



``` r
shapiro.test(target_0_ages)
```

```
## 
## 	Shapiro-Wilk normality test
## 
## data:  target_0_ages
## W = 0.93409, p-value < 2.2e-16
```

``` r
qqnorm(target_0_ages)
qqline(target_0_ages, col = "red")
```
<div align="center">
  <img src="https://github.com/user-attachments/assets/c2f0ed87-afa3-43ec-a769-752f7e6d614f" alt="unnamed-chunk-3-2">
</div>



reject the null hypothesis that the sample is normally distributed.

Non-parametric hypothesis test, HA: target_1 \> target_0


``` r
wilcox.test(target_1_ages, target_0_ages, alternative = "greater")
```

```
## 
## 	Wilcoxon rank sum test with continuity correction
## 
## data:  target_1_ages and target_0_ages
## W = 1915144, p-value < 2.2e-16
## alternative hypothesis: true location shift is greater than 0
```

reject the null hypothesis that the mean age of responders is equal to the mean age of non-responders.

In evaluation of a histogram, QQ-plot, and shapiro normality test; the ages of observations of responders and non responders were not normally distributed with confidence level \> (1 - 2.2exp(-16)). As the samples were not normally distributed a non-parametric hypotheis test was required, thus; a one-sided Wilcox rank sum test was performed for mean population age of responders to be greater than the mean population age of non responders samples. The resulting p-value of the Wilcox rank-sum test was such that with a confidence level \> (1 - 2.2exp(-16)), the mean population age of responders is greater than for non-responders.


``` r
data_income <- responders_data_sheet[responders_data_sheet[, 4] != "", ]

income_levels <- factor(data_income[, "income"],
                                    levels = c("Under $10k", "10-19,999", "20-29,999", "30-39,999",
                                   "40-49,999", "50-59,999", "60-69,999", "70-79,999",
                                   "80-89,999", "90-99,999", "100-149,999", "150 - 174,999",
                                   "175 - 199,999", "200 - 249,999", "250k+"),
                                    ordered = TRUE)

#income levels 1:15, level 1 being under 10k, level 15 being 250k+
data_income$income <- as.numeric(income_levels)

probt_income_men <- data_income[data_income[,5]=="M",]

probt_income_women <- data_income[data_income[,5]=="F",]

responders_data_sheet_men <- data_income[data_income[,5]=="M",]

responders_data_sheet_women <- data_income[data_income[,5]=="F",]
```

# Logistic Regression model


``` r
binary_classifier <- function(predictor, clean_data, color) {
  
  simple_logistic_model <- glm(data = clean_data,
                               target ~ predictor,
                               family = binomial())

  intercept_slope <- coef(simple_logistic_model)
  
  x_evaluation <- seq(min(predictor), max(predictor), length.out = 100)
  
  Prob_by_income <- function(x, intercept_slope) {
    log_odds <- intercept_slope[1] + intercept_slope[2] * x
    odds <- exp(log_odds)
    probability <- odds / (1 + odds)
    return(probability)
  }
  
  probabilities <- sapply(x_evaluation, function(x) Prob_by_income(x, intercept_slope))
  plot(x=x_evaluation,y=probabilities,col=color,xlim=range(min(x_evaluation - 5),max(x_evaluation+5)),ylim=range(0,.5),xlab="")
  legend("topleft", legend = c("Income", "Distance", "Age"),col = c("blue", "black", "red"), lty = 1, lwd = 2)
  min_prob <- min(probabilities)
  min_x <- x_evaluation[which.min(probabilities)]
  
  max_prob <- max(probabilities)
  max_x <- x_evaluation[which.max(probabilities)]
 
  delta_value <- as.numeric(tail(probabilities, 1) - head(probabilities, 1))
 
  delta_text <- paste0("Delta Prob %: ", round(delta_value*100))
  mtext(delta_text, side = 1, line = 3, col = "blue")
  
  points(min_x, min_prob, col = "black", pch = 19)
  
  text(min_x, min_prob - 0.02, labels = paste0("Min: ", round(min_prob, 3)), col = "blue")
  
  points(max_x, max_prob, col = "black", pch = 19)
  
  text(max_x, max_prob - 0.01, labels = paste0("Max: ", round(max_prob, 3)), col = "red")
  
  return(delta_value)
}
```

# probability of response by income, distance, and age


``` r
par(mfrow = c(1, 3))
#income
binary_classifier(data_income$income,data_income,c("blue"))
```

```
## [1] -0.2679233
```

``` r
#distance
binary_classifier(responders_data_sheet$dist,responders_data_sheet,c("black"))
```

```
## [1] -0.05432182
```

``` r
#age
binary_classifier(responders_data_sheet$age,responders_data_sheet,c("red"))
```
<div align="center">
  <img src="https://github.com/user-attachments/assets/f5e4dff9-ab56-4ea3-ad99-6e64d83c1f06" alt="unnamed-chunk-7-1">
</div>


```
## [1] 0.3144231
```

#probability of response stratefied by gender


``` r
predictors <- list(probt_income_men$income,probt_income_women$income,
                   responders_data_sheet_men$dist,responders_data_sheet_women$dist,
                   responders_data_sheet_men$age,responders_data_sheet_women$age) 

clean_data_list <- list(probt_income_men,probt_income_women,
                        responders_data_sheet_men,responders_data_sheet_women,
                        responders_data_sheet_men,responders_data_sheet_women)


par(mfrow = c(1,2))

colors <- c("#1E90FF","#87CEFA","#32CD32","#98FB98","#8A2BE2","#DDA0DD")

deltas <- numeric(6)
deltas <- mapply(binary_classifier, predictor = predictors, clean_data = clean_data_list, color = colors)
```

<div align="center">
  <img src="https://github.com/user-attachments/assets/94266666-19d6-4b36-be24-4b5b206fdf74" alt="unnamed-chunk-8-1">
</div>

<div align="center">
  <img src="https://github.com/user-attachments/assets/6fe326d3-1419-409f-baf2-13affa6980d4" alt="unnamed-chunk-8-2">
</div>

<div align="center">
  <img src="https://github.com/user-attachments/assets/0dcfccb6-1b56-43d8-8ea3-8e1c73b11dd2" alt="unnamed-chunk-8-3">
</div>

``` r
delta_mat <- matrix(data = deltas, nrow = 3, ncol = 2, byrow = TRUE,
                    dimnames = list(c("income", "distance", "age"), 
                                    c("Male", "Female")))

(delta_mat)
```

```
##                 Male      Female
## income   -0.26835258 -0.29127242
## distance -0.02602015 -0.06160512
## age       0.29746871  0.40568948
```

## build binary classifier

# clean data


``` r
responders_data_sheet$gender <- ifelse(responders_data_sheet$gender == "M", "F", "unknown")
responders_data_sheet$marital_status <- ifelse(responders_data_sheet$marital_status == "M", "S", "unknown")
responders_data_sheet$gender <- as.factor(responders_data_sheet$gender)
responders_data_sheet <- responders_data_sheet[responders_data_sheet[, 7] != "", ]
income_levels <- factor(responders_data_sheet[, "income"],
                                    levels = c("Under $10k", "10-19,999", "20-29,999", "30-39,999",
                                   "40-49,999", "50-59,999", "60-69,999", "70-79,999",
                                   "80-89,999", "90-99,999", "100-149,999", "150 - 174,999",
                                   "175 - 199,999", "200 - 249,999", "250k+"),
                                    ordered = TRUE)

responders_data_sheet$income <- as.numeric(income_levels)
responders_data_sheet$income[is.na(responders_data_sheet$income)] <- "unknown"
responders_data_sheet$marital_status <- as.factor(responders_data_sheet$marital_status)
responders_data_sheet$target <- factor(responders_data_sheet$target,levels = c(1,0), labels = c("response", "no_response"))
responders_data_sheet <- responders_data_sheet[,2:7]
clean_data <- responders_data_sheet
```

# Build and Test Accuracy by randomForest


```
## Loading required package: ggplot2
```

```
## Loading required package: lattice
```

```
## randomForest 4.7-1.2
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```
## 
## Attaching package: 'randomForest'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     margin
```

73% accuracy obtained

#evaluate probabilities of response for our data set and append probability column



#cumulative gains chart


``` r
lift_observation <- obs_with_prediction_prob[order(obs_with_prediction_prob[,7],decreasing = TRUE),]

baseline_observations <- obs_with_prediction_prob

totall_responces <- (sum(lift_observation[,6]=="response"))

lift_chart_cumulative_percentage <- function() {
  
        percent_contacted <- seq(0,1,by=.1)
        cumulative_resp_per = numeric(length(percent_contacted))
        cum_resp_per_baseline <- numeric(length(percent_contacted))
   
        for (i in 1:11) {
          
        cumulative_resp_per[i] =  sum(lift_observation[seq(0,percent_contacted[i]*nrow(lift_observation),by=1),6]=="response")/totall_responces*100
        cum_resp_per_baseline[i] =  sum(responders_data_sheet[seq(0,percent_contacted[i]*nrow(responders_data_sheet),by=1),6]=="response")/totall_responces*100
       
        }
        
  results_dataframe <- as.data.frame(cbind(percent_contacted,cumulative_resp_per,cum_resp_per_baseline))
  print(tail(results_dataframe,10))
  plot(x=percent_contacted,y=cumulative_resp_per,type="o",col="red",xlim=range(0,1),ylim=range(0,100))
  lines(x=percent_contacted,y=cum_resp_per_baseline,col="blue",type="o")
  
}



  lift_chart_lift_advantage <- function() {
    
    percent_contacted <- seq(.1,1,by=.1)
  cum_resp_lift <- numeric(length(percent_contacted))
  cum_resp_baseline <- numeric(length(percent_contacted))
  
  for (i in 1:10) {
  
  cum_resp_lift[i] =  sum(lift_observation[seq(1,percent_contacted[i]*nrow(lift_observation),by=1),6]=="response")/sum(responders_data_sheet[seq(1,percent_contacted[i]*nrow(responders_data_sheet),by=1),6]=="response")
  
  cum_resp_baseline[i] = sum(responders_data_sheet[seq(1,percent_contacted[i]*nrow(responders_data_sheet),by=1),6]=="response")/sum(responders_data_sheet[seq(1,percent_contacted[i]*nrow(responders_data_sheet),by=1),6]=="response")
  
  }

  LIFTCHART <- as.data.frame(cbind(percent_contacted,cum_resp_lift,cum_resp_baseline))
  print(LIFTCHART)
  plot(x=percent_contacted,y=cum_resp_lift,type="o",col="red",xlim=range(0,1),ylim=range(0,2.6),main="ratio of responce yeild by % of data set contacted")
  lines(x=percent_contacted,y=cum_resp_baseline,col="blue",type="o")
  
}


lift_chart_cumulative_percentage()
```

```
##    percent_contacted cumulative_resp_per cum_resp_per_baseline
## 2                0.1                38.2                  17.0
## 3                0.2                72.4                  28.2
## 4                0.3                88.3                  39.8
## 5                0.4                92.9                  48.5
## 6                0.5                95.4                  58.8
## 7                0.6                96.3                  67.3
## 8                0.7                97.9                  75.0
## 9                0.8                98.4                  83.6
## 10               0.9                99.4                  91.7
## 11               1.0               100.0                 100.0
```

<div align="center">
  <img src="https://github.com/user-attachments/assets/1685f432-a035-4820-b334-a9e454ddc448" alt="unnamed-chunk-12-1">
</div>


``` r
lift_chart_lift_advantage()
```

```
##    percent_contacted cum_resp_lift cum_resp_baseline
## 1                0.1      2.247059                 1
## 2                0.2      2.567376                 1
## 3                0.3      2.218593                 1
## 4                0.4      1.915464                 1
## 5                0.5      1.622449                 1
## 6                0.6      1.430906                 1
## 7                0.7      1.305333                 1
## 8                0.8      1.177033                 1
## 9                0.9      1.083969                 1
## 10               1.0      1.000000                 1
```

<div align="center">
  <img src="https://github.com/user-attachments/assets/3a5910f9-7bef-4413-8f59-a59ce73fe6f1" alt="unnamed-chunk-12-2">
</div>



In utilization of this model, only 30% of observations need to be contacted to yield 97.9% of the responses. Thus, evaluation of this model is recommended in selecting by utilizing this model by contacting only 30% of the observations, we yield 97.9% of the responses

The lift chart advantage is less at .1 then .2 because by chance the baseline did better at rows 1-800 than 1-400 proportionally. The chart is correct, the baseline and lift were calculated by the actual distribution of unmodified data set rather than theoretically is which responces and non-responces will be evenly distributed into subdivisions of the data set.

## Conclusion

In analysis of historical data, highly associated predictors of response were found; from these variables a model was built and outcomes of the models usage in future advertising campaigns were visualized. In order to utilize the categorical variables (gender and marital status) along with the continuous variables (age, income, and distance), the random Forest binary classifier algorithm was utilized and produced a predicted probability of response for each observation. The model was trained on 80% of the historical data and then used to predict response status (yes or no) on the remaing 20% (of which it did not know). The model obtained an accuracy of 73% in it's predictions of the actual outcome. 

To demonstraight the effectivness of this model, a cumulative gains chart was evaluated by sampling the observations with the highest predicted probability of response first (probability produced by the model), the result was that only 30% of individuals needed to be contacted to yield 97.9% of the total responses obtained in the entire the data set. With certainty, future advertising campaigns which select individuals for contact by predicted probability of response from this model will have significantly higher return investment.


