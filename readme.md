## pyfixest

This is a draft package (highly experimental!) for a Python clone of the excellent [fixest](https://github.com/lrberge/fixest) package. 

Fixed effects are projected out via the [PyHDFE](https://github.com/jeffgortmaker/pyhdfe) package. 

```python
import pandas as pd
import numpy as np
from pyfixest.api import feols
import statsmodels.formula.api as sm

# create data
np.random.seed(123)
N = 100000
k = 4
G = 25
X = np.random.normal(0, 1, N * k).reshape((N,k))
X = pd.DataFrame(X)
X[1] = np.random.choice(list(range(0, 50)), N, True)
X[2] = np.random.choice(list(range(0, 1000)), N, True)
X[3] = np.random.choice(list(range(0, 1000)), N, True)

beta = np.random.normal(0,1,k)
beta[0] = 0.005
u = np.random.normal(0,1,N)
Y = 1 + X @ beta + u
cluster = np.random.choice(list(range(0,G)), N)

Y = pd.DataFrame(Y)
Y.rename(columns = {0:'Y'}, inplace = True)
X = pd.DataFrame(X)

data = pd.concat([Y, X], axis = 1)
data.rename(columns = {0:'X1', 1:'X2', 2:'X3', 3:'X4'}, inplace = True)
data['X4'] = data['X4'].astype('category')
data['X3'] = data['X3'].astype('category')
data['X2'] = data['X2'].astype('category')
data['group_id'] = cluster
data['Y2'] = data.Y + np.random.normal(0, 1, N)


feols('Y ~ X1 | X2 + X3 + X4', 'hetero', data)
#   colnames      coef        se    tstat    pvalue
# 0       X1  0.001469  0.003159  0.46505  0.641896
feols('Y ~ X1 + X2 + X3 + X4', 'iid', data)
#   colnames      coef        se    tstat    pvalue
# 2048         X1    0.001469  0.003159     0.465050  6.418959e-01
sm.ols('Y ~ X1 + X2 + X3 + X4', data).fit().summary()
#   colnames      coef        se    tstat    pvalue
# X1            0.0015      0.003      0.460      0.645    

# cluster robust inference: 
feols(fml = 'Y ~ X1', vcov = {'CRV1':'group_id'}, data = data)
#     colnames        coef        se       tstat    pvalue
# 0  Intercept -577.090042  1.072007 -538.326702  0.000000
# 1         X1    1.389563  1.002708    1.385810  0.165805
feols(fml = 'Y ~ X1', vcov = {'CRV3':'group_id'}, data = data)
#     colnames        coef        se       tstat    pvalue
# 0  Intercept -577.090042  1.139483 -506.449086  0.000000
# 1         X1    1.389563  1.066219    1.303261  0.192486
```

## Multiple Estimations

Currently supported: multiple dependent variables: 

```python
feols(fml = 'Y + Y2 ~ X1', vcov = {'CRV3':'group_id'}, data = data)
# [  depvar   colnames        coef        se       tstat    pvalue
# 0      Y  Intercept -577.090042  1.139483 -506.449086  0.000000
# 1      Y         X1    1.389563  1.066219    1.303261  0.192486,   depvar   colnames        coef        se       tstat    pvalue
# 0     Y2  Intercept -577.092077  1.139712 -506.348923  0.000000
# 1     Y2         X1    1.386202  1.066095    1.300261  0.193511]
```

Support for more [fixest formula-sugar](https://cran.r-project.org/web/packages/fixest/vignettes/multiple_estimations.html) is work in progress.

