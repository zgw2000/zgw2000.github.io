---
title: "Compute the betas of a portfolio of ETFs (exchange-traded funds)."
excerpt: "The project involves analyzing the performance of a portfolio composed of four ETFs and applying the Capital Asset Pricing Model (CAPM) to calculate portfolio beta. Initially, the portfolio demonstrates higher volatility compared to the benchmark, while subsequent adjustments to the ETF weights result in reduced volatility. This project aims to provide insights into portfolio management and risk assessment, aiding in informed investment strategies.<br/><img src='/images/capm3.png'>"
collection: portfolio
---
<a id="top"></a>

***Miro Confalone, Jacopo Giannetti, Federico Astolfi, Guowang Zeng***

## Form the portfolio
Form a portfolio of four (4) ETFs, taken from the website ["Yahoo Finance"](https://finance.yahoo.com), on a sample period from 2015 to 2021. Use the S&P500 fund as your market portfolio benchmark.

First of all we imported all the packages necessary for the execution of the program, including yfinance (yahoo finance), which allows you to connect to the yahoo database and extract the data of interest in the form of dataframe.
<br/>
<br/>
First using the yfinance package we created a dataframe for each selected ticker plus a dataframe for the S&P500 benchmark, the dataframe created is composed of the following information: Open, High, Low, Close, Adj Close, Volume.
The column that interests us for our purpose is that of the adjusted price, so i eliminate all the others using the concat method of pandas.
 


## Convert daily adjusted prices to monthly log returns 
The second step is to convert the adjusted price into the monthly log, to do this we use the resample method of panda which on the dataframe that have a datetime as index like ours convert it to the chosen value, in our case "M" that is monthly.
We have now converted prices to log returns using the ptc_change () method, which calculates the percentage change from the immediately preceding row by default.


## Construct a portfolio with the weights you choose and calculate portfolio returns for each month using the ETF returns in your portfolio.
Then we created the returns of our portfolio which is made up of the four chosen ETFs all weighted at 25%:
+ Direxion Daily Technology Bull 3X Shares ETF
+ First Trust NASDAQ Clean Edge Green Energy Idx Fd ETF
+ Vanguard Mega Cap Growth Index Fund ETF
+ Direxion Daily Semiconductor Bull 3X Shares ETF
  
## Use CAPM formula to estimate the beta of the portfolio. Show and comment results.​
Now, after creating a dataframe with 2 columns one for the benchmark and one for the portfolio return, we can create our CAPM model to calculate beta, to create the CAPM model we use Ordinary Least Squares, which constructs a linear regression by calculating beta by minimizing the sum of the squared errors of prediction.
The beta we find is 2.4537, this indicates that our portfolio is more volatile than the benchmark, more than double, a 1-fold movement in the market corresponds to a 2.4- fold movement in the portfolio.

## Change the weights: change your weights and rerun the entire process to show how the portfolio betas change with the composition. Comments your results.
After we found the first beta, we changed the weights of the ETFs in the portfolio: TECL to 15%, QCLN to 20%, MGK to 50% and SOXL to 15%.
After that we have replicated all the steps made previously but with the returns of the new portfolio, and we calculated the beta again, the beta we got is 1.9033, which is less than that obtained with the previous portfolio.
<br/>
<br/>
The fact that the second beta is lower indicates that the second portfolio is less volatile than the first, but it is still more volatile than the benchmark, this is due to the fact that MGK's beta is lower than the others and since now our portfolio is composed half from MGK while before only by a quarter the average beta drops.









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
