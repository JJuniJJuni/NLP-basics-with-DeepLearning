- 특정 문장 내의 단어들의 임베딩 벡터들의 평균이 그 문장의 벡터가 될 수 있음을 설명
- 데이터나 코드에 대한 설명보다는 단어 벡터의 평균으로 얻을 수 있는 성능에 주목

# 1. 데이터 로드와 전처리

```python
import numpy as np
from tensorflow.keras.datasets import imdb
from tensorflow.keras.preprocessing.sequence import pad_sequences
```

- imdb.data_load()를 통해서 IMDB 리뷰 데이터를 다운로드
- 데이터를 로드할 때 파라미터로 num_words를 사용하면 이 데이터에서 등장 빈도 순위로 몇 번째에 해당하는 단어까지를 사용할 것인지를 의미
- vocab_size를 20,000으로 지정할 경우 등장 빈도 순위가 20,000등이 넘는 단어들은 데이터를 로드할 때 전부 제거 후 로드

```python
vocab_size = 20000

(X_train, y_train), (X_test, y_test) = imdb.load_data(num_words=vocab_size)
print('훈련용 리뷰 개수 :',len(X_train))
print('테스트용 리뷰 개수 :',len(X_test))

[output]
훈련용 리뷰 개수 : 25000
테스트용 리뷰 개수 : 25000
```

- 첫번째 리뷰와 첫번째 리뷰의 레이블을 출력

```python
print('훈련 데이터의 첫번째 샘플 :',X_train[0])
print('훈련 데이터의 첫번째 샘플의 레이블 :',y_train[0])

[output]
훈련 데이터의 첫번째 샘플 : [1, 14, 22, 16, 43, 530, 973, 1622, 1385, 65, 458, 4468, 66, 3941, 4, 173, 36, 256, 5, 25, 100, 43, 838, 112, 50, 670, 2, 9, 35, 480, 284, 5, 150, 4, 172, 112, 167, 2, 336, 385, 39, 4, 172, 4536, 1111, 17, 546, 38, 13, 447, 4, 192, 50, 16, 6, 147, 2025, 19, 14, 22, 4, 1920, 4613, 469, 4, 22, 71, 87, 12, 16, 43, 530, 38, 76, 15, 13, 1247, 4, 22, 17, 515, 17, 12, 16, 626, 18, 19193, 5, 62, 386, 12, 8, 316, 8, 106, 5, 4, 2223, 5244, 16, 480, 66, 3785, 33, 4, 130, 12, 16, 38, 619, 5, 25, 124, 51, 36, 135, 48, 25, 1415, 33, 6, 22, 12, 215, 28, 77, 52, 5, 14, 407, 16, 82, 10311, 8, 4, 107, 117, 5952, 15, 256, 4, 2, 7, 3766, 5, 723, 36, 71, 43, 530, 476, 26, 400, 317, 46, 7, 4, 12118, 1029, 13, 104, 88, 4, 381, 15, 297, 98, 32, 2071, 56, 26, 141, 6, 194, 7486, 18, 4, 226, 22, 21, 134, 476, 26, 480, 5, 144, 30, 5535, 18, 51, 36, 28, 224, 92, 25, 104, 4, 226, 65, 16, 38, 1334, 88, 12, 16, 283, 5, 16, 4472, 113, 103, 32, 15, 16, 5345, 19, 178, 32]
훈련 데이터의 첫번째 샘플의 레이블 : 1
```

- 정수 1이 출력되는데 이는 첫번째 리뷰가 긍정 리뷰임을 의미
- 각 리뷰의 평균 길이

```python
print('훈련용 리뷰의 평규 길이: {}'.format(np.mean(list(map(len, X_train)), dtype=int)))
print('테스트용 리뷰의 평균 길이: {}'.format(np.mean(list(map(len, X_test)), dtype=int)))

[output]
훈련용 리뷰의 평규 길이: 238
테스트용 리뷰의 평균 길이: 230
```

- 평균보다는 큰 수치인 400으로 패딩

```python
max_len = 400

X_train = pad_sequences(X_train, maxlen=max_len)
X_test = pad_sequences(X_test, maxlen=max_len)
print('X_train의 크기(shape) :', X_train.shape)
print('X_test의 크기(shape) :', X_test.shape)

[output]
X_train shape : (25000, 400)
X_test shape : (25000, 400)
```

# 2. 모델 설계하기

- 모델의 입력으로 사용하기 위한 모든 전처리
- GlobalAveragePooling1D()은 입력으로 들어오는 모든 벡터들의 평균을 구하는 역할
- Embedding() 다음에 GlobalAveragePooling1D()을 추가하면 해당 문장의 모든 단어 벡터들의 평균 벡터를 구한다
- 이진 분류를 수행해야 하므로 그 후에는 시그모이드 함수를 활성화 함수로 사용하는 뉴런 1개를 배치

```python
from tensorflow.keras.models import Sequential, load_model
from tensorflow.keras.layers import Dense, Embedding, GlobalAveragePooling1D
from tensorflow.keras.callbacks import EarlyStopping, ModelCheckpoint

embedding_dim = 64

model = Sequential()
model.add(Embedding(vocab_size, embedding_dim))

# 모든 단어 벡터의 평균을 구한다.
model.add(GlobalAveragePooling1D())
model.add(Dense(1, activation='sigmoid'))

es = EarlyStopping(monitor='val_loss', mode='min', verbose=1, patience=4)
mc = ModelCheckpoint('embedding_average_model.h5', monitor='val_acc', mode='max', verbose=1, save_best_only=True)

model.compile(loss='binary_crossentropy', optimizer='adam', metrics=['acc'])
model.fit(X_train, y_train, batch_size=32, epochs=10, callbacks=[es, mc], validation_split=0.2)
```

- 학습이 끝난 후에 테스트 데이터에 대해서 평가

```python
loaded_model = load_model('embedding_average_model.h5')
print("\n 테스트 정확도: %.4f" % (loaded_model.evaluate(X_test, y_test)[1]))

[output]
테스트 정확도: 0.8876
```

- 별 다른 신경망을 추가하지 않고 단어 벡터의 평균만으로도 88.76%라는 준수한 정확