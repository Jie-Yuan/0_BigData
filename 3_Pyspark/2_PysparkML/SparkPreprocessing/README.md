## 预处理模块
```python
class SparkPreprocessing(object):
    def __init__(self, df, _id='id', _label='label'):
        self.df = df
        self.id_label = [_id, _label]
        self.features_name = [i for i in df.columns if i not in self.id_label]
        __array = np.array(df.groupBy(_label).count().collect())
        self.Y_true = [j for i, j in __array if i == 1][0] + 0.
        self.N_true = [j for i, j in __array if i == 0][0] + 0.
        self.shape = self.Y_true + self.N_true, len(df.columns)
        self.df2array = lambda df: np.array(df.collect())[0]
        
    @classmethod
    def pipline(cls, df, na_prop_thresh=0.9, iv_thresh=0.1, _id='id', _label='label'):
        cls = cls(df, _id=_id, _label=_label)
        # distinct_count
        _array = cls.df2array(cls.df.select([countDistinct(i).name('_' + i) for i in cls.features_name]))
        col1 = [i for i, j in zip(cls.features_name, _array) if j != 1]
        cls.df = cls.df.select(col1 + cls.id_label)

        # na_prop
        _count = cls.shape[0]
        _array = cls.df2array(cls.df.select([(1 - count(i) / _count).name('_' + i) for i in col1]))
        col2 = [i for i, j in zip(col1, _array) if j < na_prop_thresh]
        cls.df = cls.df.select(col2 + cls.id_label)

        # iv
        y_i = ["sum({label}) OVER(PARTITION BY {feature}) as y_i_{feature}".format(label=cls.id_label[1], feature=i)
               for i in col2]
        n_i = ["sum(1-{label}) OVER(PARTITION BY {feature}) as n_i_{feature}".format(label=cls.id_label[1], feature=i)
               for i in col2]
        iv = [
            "sum((y_i_{colname}/{Y_true}-n_i_{colname}/{N_true})*log(y_i_{colname}/(n_i_{colname}+0.0001)/{y_n})) as iv_{colname}" \
                .format(colname=i, Y_true=cls.Y_true, N_true=cls.N_true, y_n=cls.Y_true / cls.N_true) for i in col2]
        _array = cls.df2array(cls.df.selectExpr(y_i + n_i).drop_duplicates().selectExpr(iv))
        _zip = sorted(zip(_array, col2), reverse=True)
        from pprint import pprint
        pprint(_zip)
        return cls.df.select([j for i, j in _zip if i > iv_thresh] + cls.id_label)
    
    def distinct_count(self):
        _array = self.df2array(self.df.select([countDistinct(i).name('_' + i) for i in self.features_name]))
        return self.df.select([i for i, j in zip(self.features_name, _array) if j != 1] + self.id_label)

    def na_prop(self, thresh=0.9):
        _count = self.shape[0]
        _array = self.df2array(self.df.select([(1 - count(i) / _count).name('_' + i) for i in self.features_name]))
        return self.df.select([i for i, j in zip(self.features_name, _array) if j < thresh] + self.id_label)

    def iv(self, thresh=0.1):
        y_i = ["sum({label}) OVER(PARTITION BY {feature}) as y_i_{feature}".format(label=self.id_label[1], feature=i)
               for i in self.features_name]
        n_i = ["sum(1-{label}) OVER(PARTITION BY {feature}) as n_i_{feature}".format(label=self.id_label[1], feature=i)
               for i in self.features_name]
        iv = ["sum((y_i_{colname}/{Y_true}-n_i_{colname}/{N_true})*log(y_i_{colname}/(n_i_{colname}+0.0001)/{y_n})) as iv_{colname}" \
                  .format(colname=i, Y_true=self.Y_true, N_true=self.N_true, y_n=self.Y_true / self.N_true) for i in self.features_name]
        _array = self.df2array(self.df.selectExpr(y_i + n_i).drop_duplicates().selectExpr(iv))
        _zip = sorted(zip(_array, self.features_name), reverse=True)
        from pprint import pprint
        pprint(_zip)
        return self.df.select([j for i, j in _zip if i > thresh] + self.id_label)

```
