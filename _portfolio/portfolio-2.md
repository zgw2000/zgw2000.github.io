---
title: "Judicial decision on ECtHR Dataset"
excerpt: "The goal of this project is building a system able to predict judicial decision with the intent of showing to what extent these are predictable. The dataset and the starting point are taken from Judicial decisions of the European Court of Human Rights: Looking into the crystal ball, a paper written by Medvedeva et al.
The dataset contains all english cases available on 09/2017 from both the Chamber and the Grand Chamber. The task consists in classifying violation or non-violation for 9 different articles of ECHR.
Their approach was basically to train 9 different binary classifiers using Support Vector Machines, one per article to assess if there was a violation or non-violation. 
The training set was built using the same number of random cases for the two classes. Many information were removed from the text of the articles, for example specific sections like the part containing the final decision that sometimes could be explicit or other informations not available until the trial ended. 
The test sets contain only one class. Notwithstanding this the task is still to build a model able to classify between the two classes rather than just identifying if it belong to that single class.
.<br/><img src='/images/ecthr.png'>"
collection: portfolio
---
<a id="top"></a>



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
