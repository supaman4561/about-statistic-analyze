# about-statistic-analyze
統計分析についての備忘録.  
主に2群間についての統計処理について.
付随してpythonのコードも載せておく.

## 対応のあるt検定(2群間)
データに対応がある場合に使用. 主に事前テストと事後テストの平均値の差を比較をするときに使う.

```python
import numpy as np
from scipy import stats

# sample datas
A = np.array([0.7, -1.6, -0.2, -1.2, -0.1, 3.4, 3.7, 0.8, 0.0, 2.0])
B = np.array([1.9, 0.8, 1.1, 0.1, -0.1, 4.4, 5.5, 1.6, 4.6, 3.4])

stats.ttest_rel(A, B)
```
[stats.ttest_rel()](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.ttest_rel.html)

効果量を知りたいときは[Cohen's d](#cohens-d).

## 対応のないt検定(2群間)
データに対応が無い場合に使用. 主にA群とB群の試験の結果を比較するのに使う.
まず, 等分散かどうかをF検定で確認する.

```python
def ftest(x1, x2):
    x1_var = np.var(x1, ddof=1)  # x1の不偏分散
    x2_var = np.var(x2, ddof=2)  # x2の不偏分散
    x1_df = len(x1) - 1  # x1の自由度
    x2_df = len(x2) - 1  # x2の自由度
    f = x1_var / x2_var  # F比
    one_sided_pval1 = stats.f.cdf(f, x1_df, x2_df)  # 片側検定のp値 1
    one_sided_pval2 = stats.f.sf(f, x1_df, x2_df)   # 片側検定のp値 2
    two_sided_pval = min(one_sided_pval1, one_sided_pval2) * 2  # 両側検定のp値

    print('F:       ', round(f, 3))
    print('p-value: ', round(two_sided_pval, 3))
```
p > 0.05 : 等分散であることが示唆されているので[Studentのt検定](#studentのt検定)に進む.  
p <= 0.05 : 2群間の分散に差があるので, [Welchのt検定](#welchのt検定)に進む.

### Studentのt検定
```python
stats.ttest_ind(A, B)
```

### Welchのt検定
```python
stats.ttest_ind(A, B, equal_var=False)
```
ここまで等分散か検定する→等平均か検定するという流れを書いたが, 
昨今は2群間の対応の無いt検定の場合, 等分散かどうかに関わらずデフォルトで
Welchのt検定をしようという流れがあるらしい. 

## Cohen's d
効果量. 一般的に0.2: 小, 0.5: 中, 0.8: 大.

```python
def cohens_d(x1, x2):
    n1 = len(x1)
    n2 = len(x2)
    x1_mean = x1.mean()
    x2_mean = x2.mean()
    s1 = x1.std()
    s2 = x2.std()
    s = np.sqrt((n1*np.square(s１)+n2*np.square(s2))/(n1+n2))
    d = np.abs(x1_mean-x2_mean)/s
    return d
```


## 参考元
[【Python】2群間での統計検定手法まとめ - Qiita](https://qiita.com/suaaa7/items/745ac1ca0a8d6753cf60)  
[コーエンのdをpythonで求める。 - Kaggle Note](https://kagglenote.com/misc/cohens-d/)
