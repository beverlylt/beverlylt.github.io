---
title: "Framingham Study"
date: 2020-07-21
tags: [AB testing, data science, Python, hypothesis testing]
header:
  image: "/images/perceptron/percept.jpg"
excerpt: "A/B testing, Data Science, Python, Hypothesis test"
mathjax: "true"
---


# Framingham Study
#### Cholesterol and Heart Disease


```python
from datascience import *
import numpy as np

%matplotlib inline
import matplotlib.pyplot as plots
plots.style.use('fivethirtyeight')
np.set_printoptions(legacy='1.13')
```

In this project, we are going to examine whether there is an association between serum cholesterol (amount of cholesterol in someone's blood) and whether or not that person develops heart disease from the Framingham study.


```python
framingham = Table.read_table('framingham.csv')
framingham
```




<table border="1" class="dataframe">
    <thead>
        <tr>
            <th>AGE</th> <th>SYSBP</th> <th>DIABP</th> <th>TOTCHOL</th> <th>CURSMOKE</th> <th>DIABETES</th> <th>GLUCOSE</th> <th>DEATH</th> <th>ANYCHD</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>39  </td> <td>106  </td> <td>70   </td> <td>195    </td> <td>0       </td> <td>0       </td> <td>77     </td> <td>0    </td> <td>1     </td>
        </tr>
        <tr>
            <td>46  </td> <td>121  </td> <td>81   </td> <td>250    </td> <td>0       </td> <td>0       </td> <td>76     </td> <td>0    </td> <td>0     </td>
        </tr>
        <tr>
            <td>48  </td> <td>127.5</td> <td>80   </td> <td>245    </td> <td>1       </td> <td>0       </td> <td>70     </td> <td>0    </td> <td>0     </td>
        </tr>
        <tr>
            <td>61  </td> <td>150  </td> <td>95   </td> <td>225    </td> <td>1       </td> <td>0       </td> <td>103    </td> <td>1    </td> <td>0     </td>
        </tr>
        <tr>
            <td>46  </td> <td>130  </td> <td>84   </td> <td>285    </td> <td>1       </td> <td>0       </td> <td>85     </td> <td>0    </td> <td>0     </td>
        </tr>
        <tr>
            <td>43  </td> <td>180  </td> <td>110  </td> <td>228    </td> <td>0       </td> <td>0       </td> <td>99     </td> <td>0    </td> <td>1     </td>
        </tr>
        <tr>
            <td>63  </td> <td>138  </td> <td>71   </td> <td>205    </td> <td>0       </td> <td>0       </td> <td>85     </td> <td>0    </td> <td>1     </td>
        </tr>
        <tr>
            <td>45  </td> <td>100  </td> <td>71   </td> <td>313    </td> <td>1       </td> <td>0       </td> <td>78     </td> <td>0    </td> <td>0     </td>
        </tr>
        <tr>
            <td>52  </td> <td>141.5</td> <td>89   </td> <td>260    </td> <td>0       </td> <td>0       </td> <td>79     </td> <td>0    </td> <td>0     </td>
        </tr>
        <tr>
            <td>43  </td> <td>162  </td> <td>107  </td> <td>225    </td> <td>1       </td> <td>0       </td> <td>88     </td> <td>0    </td> <td>0     </td>
        </tr>
    </tbody>
</table>
<p>... (3832 rows omitted)</p>



We start of by formulating the following null and alternative hypotheses:

**Null Hypothesis**: The distribution of cholesterol levels in the population among those who suffer a heart disease is the same as the distribution of cholesterol levels among those who do not.

**Alternative Hypothesis**: The cholesterol levels of people in the population who suffer heart disease are higher, on average, than the cholesterol level of people who do not.

We will use *A/B testing* for this particular case.

**Our test statistic will be the following:** The difference between the mean of cholesterol levels among those who suffer a heart disease and the mean of cholesterol levels among those who do not. 

Our test statistic is as follows:


```python
def compute_test_statistic(tbl):
    return np.mean(tbl.where('ANYCHD', 1).column('TOTCHOL')) - np.mean(tbl.where('ANYCHD', 0).column('TOTCHOL'))
```


```python
observed_statistic = compute_test_statistic(framingham)
observed_statistic
```




    16.635919905689406



Now, we can conduct a hypothesis test. We will start by creating a function to simulate the test statistic under the null hypothesis, and then run the function 1000 times to better see the distribution under the null hypothesis.


```python
def simulate_framingham_null():
    shuffle = framingham.sample(with_replacement = False).column('ANYCHD')
    table_with_shuffled_label = framingham.select('TOTCHOL').with_column("ANYCHD", shuffle)
    return compute_test_statistic(table_with_shuffled_label)
```


```python
simulate_framingham_null()
```




    -2.6720282583320056



We will now simulate the test statistic for 1000 times.


```python
framingham_simulated_stats = make_array()

for i in np.arange(1000):
    framingham_simulated_stats = np.append(framingham_simulated_stats, simulate_framingham_null())
```


```python
Table().with_column('Simulated statistics', framingham_simulated_stats).hist()
plots.scatter(framingham_observed_statistic, 0, color='red', s=30)
```




    <matplotlib.collections.PathCollection at 0x101037310>




![png](images/Cholesterol_13_1.png)



```python
framingham_p_value = sum(framingham_simulated_stats>=observed_statistic)/len(framingham_simulated_stats)
framingham_p_value
```




    0.0



Since our p-value is 0.0 or less than 0.05, it strongly suggests against the null hypothesis, and hence we reject it. One of the key findings of the Framingham study was a strong association between cholesterol levels and heart disease, and we found it to be true from our own simulation.

However, we have to keep in mind here that though there may be a correlation, we cannot assume that also implies causation. There may be other factors that can contribute to heart disease that we did not consider in this test (since we only used cholestrol). For example, hereditary genes could also cause heart diseases.

