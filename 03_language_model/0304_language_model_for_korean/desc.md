- 영어나 기타 언어에 비해서 한국어는 언어 모델로 다음 단어를 예측하기가 훨씬 까다롭다
# 1. 한국어는 어순이 중요하지 않다.
- 이전 단어가 주어졌을때, 다음 단어가 나타날 확률을 구해야하는데 어순이 중요하지 않다는 것은 다음 단어로 어떤 단어든 등장할 수 있다는 의미
```
1. 나는 운동을 합니다 체육관에서.  
2. 나는 체육관에서 운동을 합니다.  
3. 체육관에서 운동을 합니다.  
4. 나는 운동을 체육관에서 합니다.
```
# 2. 한국어는 교착어이다.
- 띄어쓰기 단위인 어절 단위로 토큰화를 할 경우에는 문장에서 발생가능한 단어의 수가 굉장히 늘어난다
- 가령 '그녀'라는 단어 하나만 해도 그녀가, 그녀를, 그녀의, 그녀와, 그녀로, 그녀께서, 그녀처럼 등과 같이 다양한 경우가 존재
- 한국어에서는 토큰화를 통해 접사나 조사 등을 분리하는 것은 중요한 작업
# 3. 한국어는 띄어쓰기가 제대로 지켜지지 않는다.
- 자연어 처리를 하는 것에 있어서 한국어 코퍼스는 띄어쓰기가 제대로 지켜지지 않는 경우가 많다
- 토큰이 제대로 분리 되지 않는채 훈련 데이터로 사용된다면 언어 모델은 제대로 동작 X