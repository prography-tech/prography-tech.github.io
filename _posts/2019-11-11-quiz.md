---
layout: post
title: "카카오 2020 공채 알고리즘 문제 풀이"
author: bluestragglr
date: 2019-11-11 00:00
tags: [algorithm, kakao]
image: 'https://kakao.github.io/files/career2020.jpg'
---


> 알고리즘 문제 풀이 by 신성환(github.com/blueStragglr)

---



#### [2020카카오공채] 문자열 압축

(https://programmers.co.kr/learn/courses/30/lessons/60057)



#### 문제 요약:

해당 문제는 임의의 string을 임의의 수의 substing으로 분해하여, 반복되는 substring을 압축함으로써 문자열을 짧게 압축하는 최적의 방법을 찾는 문제입니다. 압축은 아래와 같은 방법으로 수행합니다. 

> ababcdcdababcdcd의 경우 문자를 1개 단위로 자르면 전혀 압축되지 않지만, 2개 단위로 잘라서 압축한다면 2ab2cd2ab2cd로 표현할 수 있습니다. 다른 방법으로 8개 단위로 잘라서 압축한다면 2ababcdcd로 표현할 수 있으며, 이때가 가장 짧게 압축하여 표현할 수 있는 방법입니다.



#### 풀이 해설:

```python
import math

def solution(s):

    ##### 가장 짧은 단어를 저장할 변수와, 특정 길이의 substring으로 압축한 문자열을 저장할 변수 초기화
    shortest = s
    shortenWord = s
    

    ##### 1부터 string 길이 절반(홀수 길이일 경우 소수점 이하 올림)의 length를 갖는 subsrting으로
    ##### 압축한 결과를 비교하며 최소값을 저장하는 루프
    for i in range(1, math.ceil((len(s)+1)/2)):
    

	    	##### getShortenWord(s,i)는 문자열 s를 길이 i의 substring으로 분해한 뒤 압축한 결과를 반환
        shortenWord = getShortenWord(s,i)           
        

        ##### 압축 결과가 shortest에 저장된 string보다 짧은 경우 shortest를 갱신
        if(len(shortenWord) < len(shortest)):
            shortest=shortenWord
            
    return len(shortest) 
```

​     

```python
def getShortenWord(originalStr: str, partSize: int) -> str:
		##### 분해된 조각들을 담을 array 초기화
    partialArr = []
    
    ##### 결과를 저장할 string 초기화
    shortenWord = ''
    
    ##### string을 정해진 크기로 잘라서 저장하는 루프
    ##### 전체 string 길이를 조각의 크기로 나눈 뒤 소숫점 이하 올림
    for i in range(math.ceil(len(originalStr)/partSize)):
        ##### 읽기 편하게 줄을 바꾼 것으로, 실제로는 오작동할 수 있음. (원문은 띄어쓰기 x)
        ##### python의 parsing method를 이용하여 분해하여 저장
        partialArr.append(
        	originalStr[i*partSize:min((i+1)*partSize, len(originalStr))]
        )

    ##### buffer에는 직전에 확인한 substring의 정보를 담음
    buffer = ''
    
    ##### count에는 현재 substring이 반복된 횟수를 담음
    count = 1
    
    ##### 분해된 모든 조각에 대해서 압축 프로세스 실행
    for subStr in partialArr:
    
    		##### 이전에 나왔던 substring이 반복된 경우 count만 업데이트 하고 넘어감
        if(buffer == subStr):
            count += 1 
            
        ##### 이전에 나온 substring이 반복되지 않은 경우, 이전까지 반복된 정보를 shortenWord에 저장하고
        ##### 현재 substring에 대한 정보(buffer, count)를 업데이트 
        else:
            if(not(buffer=='')):
                if(count == 1):
                    count = ''
                shortenWord += str(count) + buffer
            count = 1
            buffer = subStr   
            
    ##### 마지막 substring은 위 루프에서 저장되지 않으므로 append를 종료하는 과정 수행
    ##### 마지막 substring이 반복된 것이 아닌 경우 substring(buffer)만 추가
    if(count==1):
        shortenWord += buffer
    ##### 반복된 substring인 경우 count와 함께 추가 
    else:
        shortenWord += str(count) + buffer  
        
    return shortenWord
```

---



#### 문제 이름: Search Insert Position

(https://leetcode.com/problems/search-insert-position/submissions/)



#### 문제 요약:

Sorting 된 int Array와 target(int)이 주어진다. target이 포함되어있다면 target의 위치를 반환하고, 포함되어 있지 않다면 추가 후 sorting했을 때의 위치를 반환하라.



#### 풀이 해설:

Sol 1. Brute Force Algorithm ~ O(n)

```python
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
    		##### index를 저장할 변수 선언
        index = 0
        for i in nums:
        		##### target보다 작은 element의 갯수를 count
            if(i < target):
                index += 1 
        ##### 갯수를 반환
        return index
```

 Sol 2. Binary Search Algorithm ~ O(log n)



---

#### 문제 이름: Max Water Container

(https://leetcode.com/problems/container-with-most-water/)



#### 문제 요약:

양의 정수가 담긴 Array가 주어진다. 해당 Array를 comb모양의 그래프로 그렸을 때, 가장 많은 물을 담을 수 있는 두 개의 값 쌍을 찾아 그 넓이를 반환하라. 

![image](https://user-images.githubusercontent.com/44422495/67058168-980ab780-f18e-11e9-81f6-a7087d5ae9cd.png)





#### 풀이 해설:

Sol 1. Brute Force Algorithm ~ O(n^2)

***TIME LIMIT EXCEEDED!!***

```
class Solution:
    def maxArea(self, height: List[int]) -> int:
    		##### Area의 최대값을 저장할 변수 선언
        maxAreaVal = 0;
        
        ##### Array 두 개로 matrix를 만들어 각각의 값 중 최대값을 탐색
        for i in range(len(height)):
            for j in range(i+1, len(height)):
                maxAreaVal = max(maxAreaVal, (min(height[i], height[j])*(j-i)))
        return maxAreaVal;
```

결과값은 올바르게 계산되었지만 펑하고 터졌습니다. 

![image](https://user-images.githubusercontent.com/44422495/67058307-2ed77400-f18f-11e9-83aa-c8cd54cd2bca.png)





 Sol 2. Two Pointer Search(Greedy Algorithm) ~ O(n)

```python
class Solution:
    def maxArea(self, height: List[int]) -> int:
    		##### 맨 앞과 맨 뒤에 pointer로 사용할 변수 선언
        frontIndex = 0
        backIndex = len(height) - 1
        
        ##### 최댓값을 저장할 변수와 임시 저장 공간 변수 초기화
        maxVal = 0;
        maxBuffer = 0;
        
        for i in range(len(height)):
        		##### 값을 계산해 봄 
            maxBuffer = min(height[frontIndex], height[backIndex]) 
												*(backIndex - frontIndex)
            ##### 둘 중 더 큰값을 저장
            maxVal = max(maxVal, maxBuffer)
            ##### 앞쪽과 뒷쪽 값 중 크기가 작은 쪽의 index를 변화시킴(앞쪽은 +1, 뒷쪽은 -1)
            ##### *해당 알고리즘의 핵심. 뒤에서 따로 증명.*
            if (height[frontIndex] < height[backIndex]):
                frontIndex += 1
            else: 
                backIndex -= 1
                
        ##### Loop에서 계산된 값들 중 최솟값을 반환
        return maxVal;
                
```



#### CLAIM ~ _앞쪽과 뒷쪽 값 중 크기가 작은 쪽의 index를 변화시키며 탐색_ 하면 올바르게 최댓값을 찾을 수 있는 것이 맞는가?

그림을 이용해 표현 해 보겠습니다. 길이 n의 array가 주어져 있을 때, `frontIndex = i, backIndex = j` 인 상황에서의 탐색은 다음과 같이 나타낼 수 있습니다.

![image](https://user-images.githubusercontent.com/44422495/67058303-2848fc80-f18f-11e9-931d-b4a5eac013a4.png)

이 경우, i를 변화시키며 탐색하게 됩니다. 이 방법을 통해 올바른 값을 찾을 수 있다는 것을 증명하기 위해서는 ***i를 이동하며 탐색했을 때 최대값 쌍을 누락하지 않는다*** 라는 것을 증명하면 됩니다. 해당 명제는 ***i를 이동하지 않고 탐색할 수 있는 모든 쌍 중에 최대값 쌍이 존재하지 않는다*** 라는 대우명제를 증명함으로써 증명할 수 있습니다. 

위 상황에서 i를 이동하지 않고 탐색하는 쌍은 `(i,i+1), (i,i+2), ... ,(i,j-1)` 입니다. 이 중 임의의 요소 `(i,k),   where i+1<k<j-1`의 넓이는 `min(h[i], h[k]) * (k-i)` 로 계산할 수 있습니다. 

이 때, `min(h[i], h[k]) < h[i]` 이므로, 임의의 요소에 대해 넓이 최대값은 `h[i](k-i)` 이 되며, 탐색할 쌍의 정의로부터 `k-i<j-i`이므로 ***i를 이동하지 않고 탐색할 수 있는 모든 쌍 중에는 `(i,j)` 쌍의 값보다 큰 값이 존재하지 않는다***는 것을 알 수 있습니다. 

즉, i를 이동하지 않고 탐색할 수 있는 모든 쌍에 의한 값은  `(i,j)` 보다 작으므로, 최대값을 존재하지 않는다는 결론을 얻을 수 있습니다.



두 알고리즘을 행렬을 이용해 비교 해 보면 조금 더 직관적인 이해를 할 수 있습니다.

 ![image](https://user-images.githubusercontent.com/44422495/67138288-e5b11e00-f27b-11e9-8d5c-63437811a6e3.png)

Brute Algorithm의 경우 모든 요소를 탐색하기 때문에 O(n^2)만큼의 결과값을 탐색해야 하는 반면, Two pointer algorithm은 명제 ***i를 이동하지 않고 탐색할 수 있는 모든 쌍 중에는 `(i,j)` 쌍의 값보다 큰 값이 존재하지 않는다*** 를 기반으로 한 행 혹은 한 열씩을 건너뛰기 때문에 한 열 혹은 한 행 중에 한개의 요소만 탐색하고 나머지를 배제할 수 있게 됩니다. 따라서 O(n) 시간 동안에 탐색을 마칠 수 있게 됩니다. 



---

#### Combination Sum

(https://leetcode.com/problems/combination-sum/)



#### 문제 요약:

해당 문제는 주어진 숫자 배열 내의 숫자들을 조합해 target의 값을 만들어낼 수 있는 모든 숫자쌍을 찾는 문제이다. 즉, 아래와 같은 input-ouput 쌍을 구하는 문제이다. 

> input: candidates = [2,3,6,7], target = 7,
> output: [ [7], [2,2,3] ]
>
> 즉, 2,3,6,7 중 임의의 숫자를 임의의 횟수만큼 사용하여 target을 만드는 방법을 탐색하는 문제입니다. 
>
> 사용 횟수에는 제한이 없습니다. 



#### 풀이 해설:

```python
class Solution:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        ##### 재귀함수를 실행하여 결과를 얻음.
        ##### 재귀함수에 필요한 변수가 기존 함수와 차이가 있어 별도 함수로 작성
        return getAvailableCombination(candidates, target, [])
        
def getAvailableCombination(candidates: List[int], targetRemain: int, stacked: List[int]) -> List[List[int]]:
  
  	##### 값을 깎아 나가며 0이 되면 반환하고, 0 이하가 되면 케이스를 종료해버리도록 재귀를 구성.
    if(targetRemain < 0):
        return [[]]
    if(targetRemain == 0):
        return [stacked]
    
    ##### 현재 loop에 담길 정답과, 임시로 스택을 쌓을 변수를 초기화
    answer = []
    tmpStack = stacked[:]
    tmpStorage = []
    
    
    for candidate in candidates:
        ##### input으로 들어온 stacked 변수가 비어있지 않고
        if(stacked!=[]):
            ##### candidate 요소가 stacked 변수의 마지막 값보다 작다면 그냥 패스.
            ##### 탐색한 요소를 중복해서 탐색하는 것을 방지하기 위함  
            if(candidate < stacked[-1]):
                continue
                
        ##### tmpStack 변수에 candidate를 추가하여, 새로운 재귀함수 실행 후 결과 저장
        tmpStack.append(candidate)
        tmpStorage = getAvailableCombination(candidates, targetRemain - candidate, tmpStack)
        
        ##### 반환받은 결과가 유효하다면 ([[]]가 아니라면) 정답에 추가
        if tmpStorage != [[]]:
            answer += tmpStorage
            
        ##### Shallow copy로 초기화. 
        tmpStack = stacked[:]
        
    return answer
   
```



해당 문제는 재귀함수를 이용하여 해결하였습니다. 피보나치 수열을 재귀함수로 해결하는 것과 비슷한 방식으로, target에서 candidate를 뺀 값으로 재귀함수를 다시 실행함으로써 문제를 해결하였습니다. 다만, 피보나치의 경우 항상 두 개의 재귀함수만 발생하므로 반복문 없이 재귀함수를 만들 수 있는 반면, 해당 문제의 경우 여러 가지 경우의 재귀함수를 모두 계산 해 주어야 하므로 상당한 공간복잡도(O(n^2logn))가 발생하게 됩니다. 

---

#### Word Ladder II

(https://leetcode.com/problems/word-ladder-ii/)



#### 문제 요약:

해당 문제는 주어진 string 배열 내에서 한 글자씩만 바꾸며 hopping하여 시작 단어에서 끝 단어에 도착할 수 있는 최단경로를 찾는 문제이다. 아래 예를 살펴보자.  

> input: 
>
> hot
> cog
> ["hit","hat","lot", "log","cog"]
>
> output: [[ "hot", "hit", "lot", "log", "cog"], [ "hot", "hat", "lot", "log", "cog"]]

즉, 위와 같이 한글자씩만 바꾸며 다른 단으로 넘어갔을 때, 최종 단어에 도착할 수 있는 가장 짧은 sequence만을 반환하는 문제이다. 여러 경로가 존재할 수 있으며, 경로가 존재하지 않는 경우에는 빈 array를 반환한다. 





#### 풀이 해설:

*조금 깁니다!

```python
class Solution:
    def findLadders(self, beginWord: str, endWord: str, wordList: List[str]) -> List[List[str]]:
      
        ##### 전처리 ~ Matrix 만들기
        ##### 단어간 이동 가능 여부를 쉽게 처리하기 위해 시작 단어를 맨 앞으로 가져오고 끝 단어를 맨 뒤로 보냅니다. 
        wordMatrix = []
        if beginWord in wordList:
            wordList.remove(beginWord)    
        wordList.insert(0, beginWord)
        if endWord not in wordList:
            return []
        wordList.remove(endWord)
        wordList.append(endWord)        
        
        ##### 단어 사이즈를 미리 변수로 저장
        strSize = len(wordList[0])
        
        ##### 각 단어마다 다른 char 갯수를 저장하는 matrix 생성. 
        ##### M(i,j) = i번째 단어와 j번째 단어의 다른 char 갯수.
        for i in range(len(wordList)):
            wordMatrix.append([])
            for j in range(len(wordList)):
                wordMatrix[i].append(0)
                for k in range(strSize):
                    if (wordList[i][k] != wordList[j][k]):
                        wordMatrix[i][j] += 1
                        
        ##### 가능한 propagation을 저장할 변수 초기화.
        ##### 첫 단어에서 하나도 이동할 수 있는 경우가 없는 경우는 바로 return. 
        availablePropagation = []
        availablePropagationBuffer = []
        for i in range(len(wordList)):
            if(wordMatrix[i][0] == 1):
                availablePropagation.append([i])
        if (availablePropagation == []):
            return []
            
        ##### 한 번 hoping에 바로 도달할 수 있는 경우에도, 이후 loop를 바로 지나가도록 설정해줌. 
        valueFounded = False
        for item in availablePropagation:
            if (wordList[item[0]] == endWord):
                valueFounded = True
                availablePropagation = [item]
                break
        
        ##### Matrix를 n번 hoping하면서 endword에 도달하는 경우 바로 종료. 
        ##### 계속해서 가능한 경로들을 쌓아나가는 방식으로 loop를 돌림. 
        for i in range(1, len(wordList)):
            if (valueFounded == True):
                break
            for propIndex in range(len(availablePropagation)):
                propItem = availablePropagation[propIndex]
                for j in range(len(wordList)):
                    if(wordMatrix[propItem[-1]][j]==1 and (j not in propItem) and j != 0):
                        availablePropagationBuffer.append(propItem + [j])
                        if (wordList[j] == endWord):
                            valueFounded = True
            availablePropagation = availablePropagationBuffer
            availablePropagationBuffer = []
            
        answer = []
        answerSet = []
        
        ##### 최단경로만을 남기고 나머지 탐색중이던 가능성들을 삭제. 
        for indexItem in availablePropagation:
            if(wordList[indexItem[-1]] == endWord):
                answer.append(beginWord)
                for wordIndex in indexItem:
                    answer.append(wordList[wordIndex])
            if(answer != []):
                answerSet.append(answer)
                answer = []
            
        return answerSet
```

해당 문제는 Matrix를 구성한 뒤, 가능한 pathway들을 모두 탐색하다 마지막 단어에 도달하는 경우 종료하도록 구성하였다. 탐색 자체는 O(n)에 마무리할 수 있지만 Matrix를 생성하는 탐색이 O(n^2)를 요구하게 된다. 









---

#### Trapping Rain Water (HARD)

(https://leetcode.com/problems/trapping-rain-water/)



#### 문제 요약:

해당 문제는 주어진 positive integer set을 block 형태로 쌓았을 때, trap되는 물의 총량을 구하는 문제이다. 아래와 같은 입력 예시를 생각해 보자. 

> Input: [0,1,0,2,1,0,1,3,2,1,2,1]
>
> 위와 같이 input이 주어지면, 아래와 같이 물이 담길 것으로 예상할 수 있다. 
>
> ![img](https://assets.leetcode.com/uploads/2018/10/22/rainwatertrap.png)
>
> output: 6

#### 풀이 해설:

#### Solution 1. Stack

##### Time: O(n), Space: O(n)

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        trapped = [0] * len(height)
        heightStack = []
        
        for i in range(len(height)):
            ##### Stack이 비어있다면 Stack에 추가
            if heightStack == []:
                heightStack.append([i,height[i]])
                
            ##### Stack이 차있다면
            else:
              	##### Stack의 첫 칸에는 Fragement에서 항상 가장 큰 값이 옴.
                ##### 만약 새로 탐색한 값이 이전에 있던 가장 큰 값보다 크다면 해당 fragment 전체에 물을 채우고
                ##### Stack을 갱신 (마지막으로 탐색한 요소만 남김)
                if heightStack[0][1] <= height[i]:
                    for j in range(heightStack[0][0] + 1, i):
                        trapped[j] = heightStack[0][1] - height[j] 
                    heightStack = [[i,height[i]]]
                else:  
                    ##### 바로 이전 요소보다 작은 값이라면 Stack에 추가함
                    if((heightStack[-1][1] > height[i]) and ((heightStack[-1][0] - i) == -1)):
                        heightStack.append([i,height[i]])
                    ##### 그렇지 않다면, 탐색값보다 큰 요소가 나올 때 까지 Backtracking
                    ##### 발견하면 탐색을 중단하고 물을 채운 뒤 Stack에서 물에 잠긴 요소를 제거
                    elif(height[i] > height[i-1]):   
                        for j in range(len(heightStack))[::-1]:
                            if (heightStack[j][1] >= height[i]):
                                for k in range(heightStack[j][0] + 1, i):
                                    if(trapped[k] < height[i] - height[k]):
                                        trapped[k] = height[i] - height[k]
                                del heightStack[j+1:]
                                break
        return (sum(trapped))

```



#### Solution 2. Dynamic Programming

홈페이지에 있는 사진으로 갈음합니다.

##### Time: O(n), Space: O(n)

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        leftMax = [0] * len(height)
        rightMax = [0] * len(height)
        maxValue = 0
        answer = 0
        for i in range(len(height)):
            maxValue = max(maxValue, height[i])
            leftMax[i] = maxValue

        maxValue = 0
        for i in range(len(height))[::-1]:
            maxValue = max(maxValue, height[i])
            rightMax[i] = maxValue
            answer += min(rightMax[i], leftMax[i]) - height[i]
        
        return answer

```

![Dynamic programming](https://leetcode.com/problems/trapping-rain-water/Figures/42/trapping_rain_water.png)



---

### Asteroid Collision

https://leetcode.com/problems/asteroid-collision/

#### 문제 요약:

해당 문제는 순서대로 위치한, 각기 다른 크기의 소행성들이 시간이 충분히 지난 후 어떻게 될지에 대한 문제이다. 주어지는 integer array는 소행성의 크기이며, 부호는 움직이는 방향이다. 작은 소행성이 큰 소행성과 부딪히면 작은 소행성이 소멸하며, 같은 크기의 소행성이 충돌시 두 개 모두 소멸한다.  

> Input: [5,10,-5]
>
> 5, 10의 소행성은 같은 방향으로 진행하지 않아 남아있고, -5의 소행성은 10의 소행성과 충돌하여 파괴되므로 결과적으로 5와 10만 남는다.
>
> output: [5,10]

#### 풀이 해설:

#### Solution 1. Stack

##### Time: O(n), Space: O(1)

Concept ~ Stack에 하나씩 소행성을 추가하며, 왼쪽(negative)으로 진행하는 소행성이 등장하는 경우 stack의 소행성들과 충돌하는 것으로 간주하여 폭발하는 소행성을 제거한다. 만일 왼쪽으로 진행하는 소행성이 stack에 왼쪽으로 진행하는 소행성만 남을 때 까지 소행성을 파괴한다면, 해당 소행성을 stack에 추가한다.  

```python
class Solution:
    def asteroidCollision(self, asteroids: List[int]) -> List[int]:
        answerStack = []
        for asteroid in asteroids:
            ##### Stack이 빈 경우, 소행성을 추가.
            if answerStack == []:
                answerStack.append(asteroid)
            else:
                #### 탐색한 소행성이 왼쪽으로 진행하고 있다면
                if asteroid < 0:
                  	##### Stack을 하나씩 탐색하며
                    for i in range(len(answerStack))[::-1]:
                      	##### 왼쪽으로 진행하는 소행성만 스택에 남았다면 그냥 추가하고
                        if (answerStack[i] < 0):
                            answerStack.append(asteroid)
                            break
                        ##### 그렇지 않다면
                        else:
                            ##### 탐색중인 소행성보다 작은 소행성을 하나씩 파괴하며
                            if (answerStack[i] + asteroid) < 0:
                                del answerStack[-1]
                                ##### 모두 파괴되었다면 Stack에 추가.
                                if i==0:
                                    answerStack.append(asteroid)
                                    
                          	##### 만약 크기가 같은 경우에는 둘 다 파괴하기만 하고 루프 종료.
                            elif (answerStack[i] + asteroid) == 0:
                                del answerStack[-1]
                                break
                                
                            ##### 왼쪽으로 진행하는 소행성이 더 작은 경우에는 바로 파괴하고 루프 종료.
                            else:
                                break
                else:                
                    answerStack.append(asteroid)

        return answerStack
                                
```






