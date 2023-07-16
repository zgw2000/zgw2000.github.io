---
title: "Compute the betas of a portfolio of ETFs (exchange-traded funds)."
excerpt: "The project involves analyzing the performance of a portfolio composed of four ETFs and applying the Capital Asset Pricing Model (CAPM) to calculate portfolio beta. Initially, the portfolio demonstrates higher volatility compared to the benchmark, while subsequent adjustments to the ETF weights result in reduced volatility. This project aims to provide insights into portfolio management and risk assessment, aiding in informed investment strategies.<center><br/><img src='/images/capm3.png'></center>"
collection: portfolio
---
<a id="top"></a>

***Miro Confalone, Jacopo Giannetti, Federico Astolfi, Guowang Zeng***

## Form the portfolio
Form a portfolio of four (4) ETFs, taken from the website [`Yahoo Finance`](https://finance.yahoo.com), on a sample period from 2015 to 2021. Use the S&P500 fund as our market portfolio benchmark.

```shell
pip install yfinance
```

First of all we imported all the packages necessary for the execution of the program, including yfinance (yahoo finance), which allows us to connect to the yahoo database and extract the data of interest in the form of dataframe.

```python
import yfinance as yf
import pandas as pd
import matplotlib.pyplot as plt
import statsmodels.api as sm
import seaborn as sns
import numpy as np
```

First using the yfinance package we created a dataframe for each selected ticker plus a dataframe for the S&P500 benchmark, the dataframe created is composed of the following information: Open, High, Low, Close, Adj Close, Volume. The column that interests us for our purpose is that of the adjusted price, so i eliminate all the others using the concat method of pandas.

```python
#we create a list with the chosen tikers and with the benchmark tiker
tiker_string = ["^GSPC", "TECL", "QCLN", "MGK", "SOXL"]
df_list = []
#for each tiker we import a dataframe from yfinance and insert it in the df_list for tiker in tiker_string:
    data = yf.download(tiker, group_by="Tiker", period="5y")
    df_list.append(data)
#we abstract the single datafreme with only the "adj close" column
df_GSPC = pd.concat([df_list[0]])["Adj Close"]
df_TECL = pd.concat([df_list[1]])["Adj Close"]
df_QCLN = pd.concat([df_list[2]])["Adj Close"]
df_MGK = pd.concat([df_list[3]])["Adj Close"]
df_SOXL = pd.concat([df_list[4]])["Adj Close"]
# Show the general S&P500 price of the market.
df_GSPC.plot()
```
<center>![capm1](/images/capm1.png)</center>

```python
# Show the portfolio prices
df_TECL.plot()
df_QCLN.plot()
df_MGK.plot()
df_SOXL.plot()
```

<center>![capm2](/images/capm2.png)</center>

## Convert daily adjusted prices to monthly log returns 
The second step is to convert the adjusted price into the monthly log, to do this we use the resample method of panda which on the dataframe that have a datetime as index like ours convert it to the chosen value, in our case "M" that is monthly.
We have now converted prices to log returns using the ptc_change () method, which calculates the percentage change from the immediately preceding row by default.

```python
#we convert to monthly GSPC
returns_GSPC_M=df_GSPC.resample('M').ffill().pct_change()
returns_GSPC_M=returns_GSPC_M.dropna(axis=0)
#we convert to monthly ETFs
returns_TECL_M=df_TECL.resample('M').ffill().pct_change()
returns_QCLN_M=df_QCLN.resample('M').ffill().pct_change()
returns_MGK_M=df_MGK.resample('M').ffill().pct_change()
returns_SOXL_M=df_SOXL.resample('M').ffill().pct_change()
# Show the portfolio log return
returns_TECL_M.plot()
returns_QCLN_M.plot()
returns_MGK_M.plot()
returns_SOXL_M.plot()
```

<center>![capm3](/images/capm3.png)</center>

## Construct a portfolio with the weights we choose and calculate portfolio returns for each month using the ETF returns in our portfolio.
Then we created the returns of our portfolio which is made up of the four chosen ETFs all weighted at 25%:
+ Direxion Daily Technology Bull 3X Shares ETF
+ First Trust NASDAQ Clean Edge Green Energy Idx Fd ETF
+ Vanguard Mega Cap Growth Index Fund ETF
+ Direxion Daily Semiconductor Bull 3X Shares ETF

```python
#we average with 25,25,25,25
returns_avg = 0.25*returns_TECL_M+0.25*returns_QCLN_M+0.25*returns_MGK_M+0. ,→25*returns_SOXL_M
returns_avg=returns_avg.dropna(axis=0) #and we eliminate the null values
#we concat the benchmark with the average of the etfs
df=pd.concat([returns_GSPC_M, returns_avg], axis=1) df=df.dropna(axis=0) #and we eliminate the null values df.columns = ['GSPC', 'AVG'] #let's rename the columns
# Comare the returns of S&P500 and our weighted portfolio.
df.plot()
```
<center>![capm4](/images/capm4.png)</center>

## Use CAPM formula to estimate the beta of the portfolio. Show and comment results.​
Now, after creating a dataframe with 2 columns one for the benchmark and one for the portfolio return, we can create our CAPM model to calculate beta, to create the CAPM model we use Ordinary Least Squares, which constructs a linear regression by calculating beta by minimizing the sum of the squared errors of prediction.
The beta we find is 2.4537, this indicates that our portfolio is more volatile than the benchmark, more than double, a 1-fold movement in the market corresponds to a 2.4- fold movement in the portfolio.

```python
#and we create the model
x=df['GSPC']
y=df['AVG']
x_sm=sm.add_constant(x)
model=sm.OLS(y,x_sm)
results=model.fit()
results.summary()
```
<center>![capm5](/images/capm5.png)</center>
<center>![capm6](/images/capm6.png)</center>

```python
#Plot the regression.
sns.regplot(x='GSPC',y='AVG',data=df)
```

<center>![capm7](/images/capm7.png)</center>

## Change the weights and rerun the entire process to show how the portfolio betas change with the composition.
After we found the first beta, we changed the weights of the ETFs in the portfolio: TECL to 15%, QCLN to 20%, MGK to 50% and SOXL to 15%.
After that we have replicated all the steps made previously but with the returns of the new portfolio, and we calculated the beta again, the beta we got is 1.9033, which is less than that obtained with the previous portfolio.
<br/>
<br/>
The fact that the second beta is lower indicates that the second portfolio is less volatile than the first, but it is still more volatile than the benchmark, this is due to the fact that MGK's beta is lower than the others and since now our portfolio is composed half from MGK while before only by a quarter the average beta drops.

```python
#we repeat the same thing by changing the weights of the etfs, in 15,20,50,15
returns_avg_2 = 0.15*returns_TECL_M+0.2*returns_QCLN_M+0.5*returns_MGK_M+0. ,→15*returns_SOXL_M
returns_avg_2=returns_avg_2.dropna(axis=0)
df_2=pd.concat([returns_GSPC_M, returns_avg_2], axis=1)
df_2=df_2.dropna(axis=0)
df_2.columns = ['GSPC', 'AVG']
df_2.plot()
```

<center>![capm8](/images/capm8.png)</center>

```python
x_2=df_2['GSPC']
y_2=df_2['AVG']
x_sm_2=sm.add_constant(x_2)
model_2=sm.OLS(y_2,x_sm_2)
results_2=model_2.fit()
results_2.summary()
```

<center>![capm9](/images/capm9.png)</center>
<center>![capm10](/images/capm10.png)</center>

```python
sns.regplot(x='GSPC',y='AVG',data=df_2)
```

<center>![capm11](/images/capm11.png)</center>

[<center>↑ BACK TO TOP ↑</center>](#top)

  <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
  <script>
    $(document).ready(function() {
      $('a[href="#top"]').click(function() {
        $('html, body').animate({ scrollTop: 0 }, 'slow');
        return false;
      });
    });
  </script>
