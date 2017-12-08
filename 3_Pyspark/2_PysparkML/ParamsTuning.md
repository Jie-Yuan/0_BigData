```python
class SparkML(object):
    def __init__(self):
        pass

    @staticmethod
    def cv(df, clf, _id='id', _label='label'):
        CrossValidator_ = CrossValidator(estimator=clf[0],
                                         estimatorParamMaps=clf[1],
                                         evaluator=SparkML.evaluator(),
                                         numFolds=3,
                                         seed=0)
        cvModel = CrossValidator_.fit(SparkML.vectorAssembler(df, _id, _label))
        return cvModel

    @staticmethod
    def evaluator(metricName="areaUnderROC"):
        evaluator_ = BinaryClassificationEvaluator(rawPredictionCol="rawPrediction",
                                                   labelCol="label",
                                                   metricName=metricName)
        return evaluator_

    @staticmethod
    def vectorAssembler(df, _id, _label):
        df = df.withColumnRenamed(_label, 'label').withColumnRenamed(_id, '_id')
        X_name = filter(lambda x: x != _id and x != _label, df.columns)
        df = VectorAssembler(inputCols=X_name, outputCol='features') \
            .transform(df) \
            .select('_id', 'features', 'label')
        return df


class CvModel(object):
    def __init__(self):
        pass

    @staticmethod
    def rf():
        rf = RandomForestClassifier()
        estimatorParamMaps_ = ParamGridBuilder().addGrid(rf.maxBins, [24]) \
            .addGrid(rf.maxDepth, [4]) \
            .addGrid(rf.impurity, ["entropy"]) \
            .addGrid(rf.numTrees, [15]) \
            .build()
        return rf, estimatorParamMaps_
```
