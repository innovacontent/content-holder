---
Layout:Post
useMath: true
Title:"Information Theory in The Financial Markets"
Date:2022-01-14
Categories:Research
Description:"Information theory is the mathematical treatment of the concepts, parameters and rules governing the transmission of messages through communication systems"
---

# **Information Theory in The Financial Markets**

## Introduction

I'll start by the strict definition of the term information theory and thereafter give a simple definition.  
Information theory is the mathematical treatment of the concepts, parameters and rules governing the transmission of messages through communication systems.
to simplify this I'd put it this Information theory is concern on transmitting information in a noisy channel.
The concept was founded by Claude Shannon.

### some key term defination.
I'd like us to define some time so we can get the ball rolling as we will need them as we go


*Information*  -  is the decrease in ambiguity regarding a phenomenon, an increment to our knowledge when we observe a specific event.

*Uncertainty* -  is something which is possible but is unknown

*Entropy* - denoted as `(Hx)` is a measure of the amount of uncertainty associated with a variable X when only its distribution is known i.e It is a measure of the amount of information we hope to learn from an outcome (observations), and we can understand it as a measure of uniformity

The more uniform the distribution, the higher the uncertainty and the higher the entropy. This definition is important because entropy will ultimately be used to measure the randomness of a series.

#### Why do e need to measure the randomness of a series in the financial markets.

The stock markets reflects the interaction of many agents buying or selling a particular index. Influenced by their beliefs and the economic situation, the agents may feel inclined to buy or sell under markets with clear trends or act more erratically in Time of great uncertainty. Therefore, by quantifying the level of randomness of the stock markets we can obtain insights on the general behavior of the participants.

#### Some Illustration
its time to dive into action and leave talks

so assume we have two variables

A = 01010101010101010101

B = 01101000110111100010

While it is true that the two series have been produced randomly with the same probability and have the same values of mean and variance, it is easy to continue the pattern described in series A, but we would have to work harder to be able to predict the next number in series B. 

##### illustration in julia
````markdown
Some rendered text...

    // This is code by way of four leading spaces
    // or a leading tab
    using Statistics
    using Distributions

    A = [0,1,0,1,0,1,0,1,0,1,0,1,0,1,0,1,0,1,0,1]
    B = [0,1,1,0,1,0,0,0,1,1,0,1,1,1,1,0,0,0,1,0]
    
    ##run the risk metrics
    std(A) 
    std(B)
    var(A)
    var(B)

````

Series A is constructed by alternating zeros and ones, while series B does not have a clear pattern.
if we run the matics of the starndadard deviation and variances well know measure of risk we get ```0.51298917 and 0.263157``` respectively we could go on and run  kurtosis and the outcome ewould be equal despite the clear randomness we see hence The analysis of randomness is, therefore, an essential limitation of the methods of moment statistics and cannot be used to guarantee the randomness of a series mathematically.

The classical probabilistic approach of analyzing the moments of different orders is not analyzing the randomness of a series but the randomness of the generating process of the series.

How do I mean by different random generating order,
If we shuffle any of the previous two series we would obtain the same values using moment statistics since the calculations are independent of the organization, their terms are of the form his expression:  
 ∑2_𝑖(𝑥_𝑖−𝑥 ̅ )^𝑎 , where a is the order of the calculated moment.

since entropy quantifies the amount of information, it also measures the degree of randomness in the data series. It would be desirable that one of the characteristics when measuring the randomness of a series were to establish a hierarchy of degrees of randomness, a quantification of the level of randomness so that the question would not be whether a data series is random or not, but how random it is.

Information Theory was designed as a theory to analyze the process of sending messages through a noisy channel and understanding how to reconstruct the message with a low probability of error.
The analysis of communications by telegraph led them to present a formula to measure information of the type ```𝐻=𝑛 logS```, where S is the number of possible symbols and n the number of them transmitted.
Shannon extended this formulation to bypass that restriction, proposing entropy as the measure of information of a single random variable.

If  X can take the values ```{𝑥_(1∙ ∙ ∙ ) ├ 𝑥_𝑛 }┤``` and ```p(x)``` is the probability associated with those values given  ```𝑥∈𝑋``` , entropy is defined as:
                ```𝐻(𝑋)=−∑_(𝑥∈𝑋)▒〖𝑝(𝑥)  log⁡𝑝(𝑥) 〗```
entropy is based on the probability of occurrence of each symbol, i.e., entropy is not a function of the values of the series themselves but a function of their probabilities.

###### illustrationusing kenyan financial markets data

wehn running this example we are interested in on the returns so we will start by wrriting a return function.i am assuming a technical knowledge in finance hence wont go into the definations of return

```
function returnCalculation(inputs::Array{Float64,1})
  todaysprice = inputs
  previous = lag(inputs)
  assetret =  (todaysprice./previous) .- 1 
   return assetret
end
```
i'll skip the reading in of the file as the data used here is proprietory 

so we write the shanons entropy model 
```
function ShanonEntropy(Data)
    X = Data
    len = length(X)
    #assign probabilities greater than zero is market up and lower than is market down.
    p_up   =   sum(X .>= 0) /len
    p_down = sum(X .< 0) / len 
    total = 0 
    if p_up > 0
      total = total + p_up * log(2,p_up)
    end
    if p_down > 0
      total = total + p_down * log(2,p_down)
    end
    return(-total)
end
```
the securities i am using are 

* Kenya Commercial Bank Ltd Ord 1.00
* British American Tobacco Kenya Ltd Ord 10.00
* Nairobi Securities Exchange Ltd Ord 4.00  
          
```
assets = ["Kenya Commercial Bank Ltd Ord 1.00",
          "British American Tobacco Kenya Ltd Ord 10.00",
          "Nairobi Securities Exchange Ltd Ord 4.00"]

SecuritiesReturn = 
begin
    @pipe NSEMarketData |>
    select(_, Not([:Low,:High])) |>
    filter(x -> x.MarketDate >= Date("2015-01-01", "y-m-d") && x.MarketDate <= Date("2020-12-31", "y-m-d"), _) |>
    filter(x -> x.Name in assets, _) |>
    unstack(_, :Name, :MarketPrice) |>
    rename(_, :MarketDate => :Date) |>
    Impute.locf |>
    transform(_,:2 .=> ByRow(Float64),renamecols = false)|>
    transform(_,:3 .=> ByRow(Float64),renamecols = false)|>
    transform(_, [:2,:3] .=> returnCalculation, renamecols=false)|>
    dropmissing(_)
end
```
what happens in the above code bloack straightforward we selct the date range we want to use and form the data into a usable form and finally calculate returns from our return function.
the drop missing is so we can be able to drop the initial row since when calculating return we will have to omit the first row as it will be missing.

Now we move ahead and calculate the the 20day shanons moving entropy to see which amoung our selected stocks are as random.

first before i move to the shanons 20 day shanons entropy lets look at the genral overrall entropy for the period.
```
ShanonEntropy(IndexReturns.NseAllshare)
ShanonEntropy(SecuritiesReturn[!,:2])
ShanonEntropy(SecuritiesReturn[!,:3])
```
and now here is the shanons 20 day moving entropy you can change this and play around with this to see how it changes.

```
#Calculte the 20day moving average of the shanons Entropy
MarketDate =  SecuritiesReturn[!,:1] 
EntropyKCB = rolling(ShanonEntropy, SecuritiesReturn[!,:2], 20) 
EntropyBAT = rolling(ShanonEntropy, SecuritiesReturn[!,:3], 20) 
lowerLmt1 = length(MarketDate) - 1 
uppeLmt1 =  length(EntropyKCB) 
deleteat!(MarketDate,uppeLmt1:lowerLmt1)
```
now we are going to create a dataframe for this so that we can be able to plot this 
```
#Transform the out put into a data frame
DfSecuritiesEntropy = DataFrame(Date = MarketDate,
                             KCBEntropy = vec(EntropyKCB),
                             BATEntropy = vec(EntropyBAT))

#Plot the out put against the time 
plot2 = plot(DfSecuritiesEntropy[!,:1],DfSecuritiesEntropy[!,:KCBEntropy],
title  = "KCB Share Rolling 20 Day Shannon Entropy vs Time",
xlabel = "Year",
ylabel = "Rolling 20 Day Shannon Entropy")
#theme  = :juno)

#Plot BAT moving 20 shanons Entropy
plot3 = plot(DfSecuritiesEntropy[!,:1],DfSecuritiesEntropy[!,:BATEntropy],
title  = "BAT Share Rolling 20 Day Shannon Entropy vs Time",
xlabel = "Year",
ylabel = "Rolling 20 Day Shannon Entropy")
#theme  = :juno)

plot(plot2, plot3, layout = (2, 1), legend = false,size = (900, 600))

````
          
##### some concluding thoughts.
Note . With Shannon’s definition, events with high or low probability do not contribute much to the value of the measure, as shown in the graph for values of p = 0 or p = 1. 
In conclusion:
The question of evaluating the randomness of a series of data is, therefore, a matter of characterizing the system as stochastic or deterministic.
More precisely, to which extent the data behaves as stochastic and how much determinism exists; we would be talking about degrees of randomness. If there is no underlying deterministic process when analyzing a particular series of data, it will imply a verification of the strictest random movement. However, if there are cycles, trends, and patterns, then the data series would not be totally random. It will be necessary to quantify the degrees of randomness.


