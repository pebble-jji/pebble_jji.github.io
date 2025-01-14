---
layout : single
title : "2022-03-25-Hands on ML_Ch2 연습문제"
categories : 
  - homl
---
## 2장 연습문제  

1. SVM 회귀를 kernel = 'linear'(하이퍼파라미터 C를 바꿔가며)나 kernel = 'rbf'(하이퍼파라미터 C와 gamma를 바꿔가며) 등의 다양한 하이퍼 파라미터 설정으로 시도해보세요. 최상의 SVR모델은 무엇인가요?  

```python
from sklearn.svm import SVR

svm_reg = SVR()
svm_reg.fit(housing_prepared,housing_labels)

from sklearn.model_selection import GridSearchCV


param_grid = [
              {'kernel': ['linear'], 'C': [1,5,25,125,625,3125]},
              {'kernel': ['rbf'], 'C':  [1,5,25,125,625,3125],
                'gamma': [0.01,0.1,1,10]},]

svm_reg = SVR()

grid_search = GridSearchCV(svm_reg,param_grid, cv = 5,
                           scoring = 'neg_mean_squared_error',
                           return_train_score = True)

grid_search.fit(housing_prepared, housing_labels)
```

```python
GridSearchCV(cv=5, estimator=SVR(),
             param_grid=[{'C': [1, 5, 25, 125, 625, 3125],
                          'kernel': ['linear']},
                         {'C': [1, 5, 25, 125, 625, 3125],
                          'gamma': [0.01, 0.1, 1, 10], 'kernel': ['rbf']}],
             return_train_score=True, scoring='neg_mean_squared_error')
```

```python
neg_mse = grid_search.best_score_

grid_search.best_params_

{'C': 3125, 'gamma': 0.1, 'kernel': 'rbf'}
```

2. GridSearchCV를 RandomizedSearchCV로 바꿔보세요.

```python
from sklearn.model_selection import RandomizedSearchCV


param_grid = [
              {'kernel': ['linear'], 'C': [1,5,25,125,625,3125]},
              {'kernel': ['rbf'], 'C':  [1,5,25,125,625,3125],
                'gamma': [0.01,0.1,1,10]},]

svm_reg = SVR()

rand_search = RandomizedSearchCV(svm_reg,param_grid, cv = 5,
                           scoring = 'neg_mean_squared_error',
                           return_train_score = True)

rand_search.fit(housing_prepared, housing_labels)
```


```python
RandomizedSearchCV(cv=5, estimator=SVR(),
                   param_distributions=[{'C': [1, 5, 25, 125, 625, 3125],
                                         'kernel': ['linear']},
                                        {'C': [1, 5, 25, 125, 625, 3125],
                                         'gamma': [0.01, 0.1, 1, 10],
                                         'kernel': ['rbf']}],
                   return_train_score=True, scoring='neg_mean_squared_error')
```



```python
neg_mse = rand_search.best_score_
rand_search.best_params_
```




## 3. 분류

### 1. MNIST

#### Import Data

```python
from sklearn.datasets import fetch_openml # mnist 데이터 열기
mnist = fetch_openml('mnist_784', version = 1)
mnist.keys()
```


```python
dict_keys(['data', 'target', 'frame', 'categories', 'feature_names', 'target_names', 'DESCR', 'details', 'url'])
```



#### Data Shape

```python
# 데이터가 28*28 픽셀짜리 70000개

X,y = mnist['data'], mnist['target']

print(X.shape)

print(y.shape)
```

```python
(70000, 784)
(70000,)
```

#### 데이터 보기

```python
import matplotlib as mpl
import matplotlib.pyplot as plt

some_digit = X[0] # 0번 그림 호출
some_digit_image = some_digit.reshape(28,28) # 픽셀에 불이 들어오는지 아닌지 

plt.imshow(some_digit_image, cmap = 'binary') # 이미지로 반영
plt.axis('off')
plt.show()
```

![img](data:image/png;base64,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)

5인 것으로 보인다.  
y를 정수로 반환하기
```python
y = y.astype(np.uint8)
y[0]
```

타겟값도 5임을 알 수 있다.

#### Test Set 분리하기

```python
X_train, X_test, y_train, y_test = X[:60000], X[60000:], y[:60000], y[60000:]
```

이미 섞여있기 때문에 굳이 섞지 않는다.



### 2. 이진 분류기 훈련

확률적 경사하강법(SGD)  
랜덤하게 추출한 일부 데이터에 대해 가중치를 조절
- 속도 빠름
- 최적 해의 정확도 낮음

$W(t+1) = W(t) -\alpha \frac{\partial}{∂w} Cost(w)$
> - Learning Rate : $\alpha$
> - Gradient : $ \frac{\partial}{∂w} Cost(w) $
>
> 조금씩 앞으로 접근하게 하여 최적해에 다가가는 방식

```python
y_train_5 = (y_train == 5) # 5인지 아닌지 판단한 벡터
y_test_5 = (y_test == 5)
```

```python
from sklearn.linear_model import SGDClassifier

sgc = SGDClassifier(random_state = 42)  # 시드 유지
sgc.fit(X_train,y_train_5) # SGD 분류기로 훈련시킨 벡터

sgc.predict([some_digit]) # 샘플 데이터로 예측한 픽셀값
```

```python
array([ True]) # 얘는 5가 맞을거라고 예상함
```

정답이 맞다.

### 3. 성능 측정

```python
from sklearn.model_selection import StratifiedKFold # 클래스별 비율이 유지되도록 폴드를 만들기
from sklearn.base import clone # 매번 분류기를 복제

skfolds = StratifiedKFold(n_splits = 3, random_state = 42, shuffle = True) # 폴드 3개 k겹 교차검증

for train_index, test_index in skfolds.split(X_train,y_train_5) : 
  clone_clf = clone(sgc)
  X_train_folds = X_train[train_index]
  y_train_folds = y_train_5[train_index]
  X_test_folds = X_train[test_index]
  y_test_folds = y_train_5[test_index]

  clone_clf.fit(X_train_folds, y_train_folds)
  y_pred = clone_clf.predict(X_test_folds) # 테스트 세트로 예측한 값
  n_correct = sum(y_pred == y_test_folds) # 예측값과 타겟값이 맞는가
  print(n_correct / len(y_pred))
```

```python
0.9669 # 모델 1 : 정확도 96.7%
0.91625 # 모델 2 : 정확도 91.6%
0.96785 # 모델 3 : 정확도 96.8%
```

```python
from sklearn.model_selection import cross_val_score
cross_val_score(sgc,X_train,y_train_5,cv = 3,scoring = 'accuracy')
```

```python
array([0.95035, 0.96035, 0.9604 ])
```

```python
from sklearn.base import BaseEstimator

class Never5Classifier(BaseEstimator) : 
  def fit(self,X,y=None) :
    return self
  def predict(self,X) : 
    return np.zeros((len(X),1), dtype = bool)
```

### 2. 오차행렬
A인데 B라고 한게 몇개인지 센다.  

```python
from sklearn.model_selection import cross_val_predict

y_train_pred = cross_val_predict(sgc,X_train,y_train_5,cv = 3)
```

생각해보니까 품질관리 개념이다.
어떤 상품이 품질의 규격을 벗어나지 않는 것이 정밀도, 계속 똑같은 상품이 나올 확률을 재현율이라고 생각하면 된다.  

- 정밀도 =$\frac{TP}{TP+FP}$ : 양성 예측의 정확도  
- 재현율 =$\frac{TP}{TP+FN}$  양성들 중 예측이 맞는 비율    

- 정밀도/재현율 트레이드오프  
$전체 샘플 개수 = FP + FN$이므로 정밀도가 커진다는 것은 FP가 작아졌다는 것이고, 그러면 FN은 커질 수 밖에 없기 때문에 정밀도와 재현율의 증감은 반대이다.  

```python
from sklearn.metrics import confusion_matrix
confusion_matrix(y_train_5,y_train_pred)
```

```python
array([[53892,   687],
       [ 1891,  3530]])
```
[[**T**rue **N**egative,**F**alse **N**egative)],  
[**F**alse **P**ositive,**T**rue **P**ositive]]  

코로나 자가진단 키트를 생각하고 이해하면 쉽다.

```python
from sklearn.metrics import precision_score, recall_score

precision = precision_score(y_train_5,y_train_pred)
recall = recall_score(y_train_5,y_train_pred)

print(f'정밀도 : {precision}\n재현율 : {recall}')
```
```python
정밀도 : 0.8370879772350012
재현율 : 0.6511713705958311
```
```python
y_train_perfect_predictions = y_train_5 # 원래 타겟 데이터
confusion_matrix(y_train_5,y_train_perfect_predictions) # 당연히 다 맞지
```
```python
from sklearn.metrics import precision_score, recall_score

precision = precision_score(y_train_5,y_train_perfect_predictions)
recall = recall_score(y_train_5,y_train_perfect_predictions)

print(f'정밀도 : {precision}\n재현율 : {recall}') # 당연히 다 맞지
```
#### F1 Score  
정밀도와 재현율의 조화평균  
$F_1 = \frac{2}{\frac{TP+FP}{TP}+ \frac{TP+FN}{TP}}=\frac{2}{\frac{2TP+FP+FN}{TP}}=\frac{2TP}{2TP+FP+FN}$  


정밀도와 재현율이 비슷하면 F1스코어가 높음  
(적당한 선에서 합의보는것임)

![img](C:\Users\sarah\Downloads\Image-1.jpg)

```python
from sklearn.metrics import f1_score
f1_score(y_train_5,y_train_pred)
```
```python
0.7325171197343846
```

결정함수 : 샘플 각각의 점수를 판단하고 그 점수가 임곗값보다 크면 True 출력

```python
y_scores = sgc.decision_function([some_digit]) # 샘플 픽셀들의 점수 벡터
threshold = 0 # 임계값
y_some_digit_pred = (y_scores > threshold) # 임계값 넘는 애들만 True

y_some_digit_pred
```
```python
array([ True])
```
```python
y_scores = sgc.decision_function([some_digit]) # 샘플 픽셀들의 점수 벡터
threshold = 8000 # 임계값
y_some_digit_pred = (y_scores > threshold) # 임계값 넘는 애들만 True

y_some_digit_pred
```
```python
array([False])
```
모든 샘플의 결정점수를 구한다.
```python
y_scores = cross_val_predict(sgc, X_train, y_train_5, cv=3, method = 'decision_function') # 결정 점수 반환

y_scores
```
```python
array([  1200.93051237, -26883.79202424, -33072.03475406, ...,
        13272.12718981,  -7258.47203373, -16877.50840447])
```
```python
from sklearn.metrics import precision_recall_curve

precisions, recalls, thresholds = precision_recall_curve(y_train_5,y_scores)
```
```python
def plot_precision_recall_vs_threshold(precisions,recalls,thresholds):
  plt.plot(thresholds,precisions[:-1],'b--',label = '정밀도')
  plt.plot(thresholds,recalls[:-1],'g-',label = '재현율')

plot_precision_recall_vs_threshold(precisions,recalls,thresholds)
plt.show()
```
![img](data:image/png;base64,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)

```python
threshold_90_precision = thresholds[np.argmax(precisions >= 0.9)]
threshold_90_precision
```
```python
3370.0194991439557
```
```python
y_train_pred_90 = (y_scores >= threshold_90_precision)
y_train_pred_90
```
```python
array([False, False, False, ...,  True, False, False])
```
```python
print(precision_score(y_train_5,y_train_pred_90))
print(recall_score(y_train_5,y_train_pred_90))
```
```python
0.9000345901072293
0.4799852425751706
```

적당한 선에서 합의를 본 것을 볼 수 있다.

이를 그래프로 그리는 코드는 다음과 같다.

```python
plt.plot(recalls[:-1],precisions[:-1],'#6495ED')
plt.annotate('proper threshold',(0.4799852425751706,0.9000345901072293),\
             (0.4799852425751706+0.07,0.9000345901072293+0.07),arrowprops ={'color' : '#437C17'})
plt.show()
```

![img](data:image/png;base64,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)

#### ROC 곡선 

Receiver Operating Charatericstic   

$FPR = \frac{FP}{FP+TN}$   

$FPR = 1-TNR$    

TNR : 특이도 

\- 재현율 =$\frac{TP}{TP+FN}$  

ROC : 재현율(민감도)에 대한 1-특이도 그래프  

+) 트레이드오프가 존재할 수 밖에 없음  

```python
from sklearn.metrics import roc_curve

fpr,tpr,thresholds = roc_curve(y_train_5,y_scores)

def plot_roc_curve(fpr,tpr,label = None) :
  plt.plot(fpr,tpr,linewidth = 2, label = label)
  plt.plot([0,1],[0,1],'k--')

plot_roc_curve(fpr,tpr)
plt.show()
```

![img](data:image/png;base64,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)

#### 분류기 : 그래프 아래 면적을 계산.

분류기 면적 = 1 : 완벽 

분류기 면적 = 0.5 : 랜덤 분류



```python
from sklearn.metrics import roc_auc_score

roc_auc_score(y_train_5,y_scores)
```

```python
0.9604938554008616
```

```python
from sklearn.ensemble import RandomForestClassifier
forest = RandomForestClassifier(random_state = 42)
y_probas_forest = cross_val_predict(forest, X_train, y_train_5, cv = 3, method = 'predict_proba')

y_scores_forest = y_probas_forest[:,1]
fpr_forest, tpr_forest,thresholds_forest = roc_curve(y_train_5,y_scores_forest)

plot_roc_curve(fpr,tpr,label = 'SGD')
plot_roc_curve(fpr_forest,tpr_forest, label='Forest')
plt.legend()
plt.show()
```

![img](data:image/png;base64,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)

```python
roc_auc_score(y_train_5,y_scores_forest)
```

```python
0.9983436731328145
```
