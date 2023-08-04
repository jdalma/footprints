
<!-- TOC -->

- [**알고리즘을 왜 하고, 얼마나 깊게 해야할까**](#알고리즘을-왜-하고-얼마나-깊게-해야할까)
- [**`Binary Search` 이분,이진 탐색**](#binary-search-이분이진-탐색)
  - [Basic Template](#basic-template)
- [**그래프**](#그래프)
  - [**인접 리스트**](#인접-리스트)
  - [**인접 행렬**](#인접-행렬)
  - [**그래프 탐색**](#그래프-탐색)
    - [깊이 우선 탐색 (DFS)](#깊이-우선-탐색-dfs)
    - [넓이 우선 탐색 (BFS)](#넓이-우선-탐색-bfs)
    - [양방향 탐색](#양방향-탐색)
- [**트리**](#트리)
  - [**이진 트리 (Binary Tree)**](#이진-트리-binary-tree)
  - [**이진 탐색 트리 (Binary Search Tree)**](#이진-탐색-트리-binary-search-tree)
  - [**완전 이진 트리 (Complete Binary Search)**](#완전-이진-트리-complete-binary-search)
  - [**전 이진 트리 (Full Binary Tree)**](#전-이진-트리-full-binary-tree)
  - [**포화 이진 트리 (Perfect Binary Tree)**](#포화-이진-트리-perfect-binary-tree)
  - [**이진 트리 순회**](#이진-트리-순회)
    - [중위 순회 (in-order traversal)](#중위-순회-in-order-traversal)
    - [전위 순회 (pre-order traversal)](#전위-순회-pre-order-traversal)
    - [후위 순회 (post-order traversal)](#후위-순회-post-order-traversal)
  - [**이진 힙 (최소 힙)**](#이진-힙-최소-힙)
- [균형 탐색 트리](#균형-탐색-트리)
  - [`2-3` 탐색 트리](#2-3-탐색-트리)
  - [레드 블랙 트리](#레드-블랙-트리)
- [**트라이 Trie (접두사 트리)**](#트라이-trie-접두사-트리)
- [**TreeSet**](#treeset)
- [우선순위 큐 **2.4.3 참조**](#우선순위-큐-243-참조)
  - [**힙**](#힙)
    - [**상향식 힙 복구(swim)**](#상향식-힙-복구swim)
    - [**하향식 힙 복구(sink)**](#하향식-힙-복구sink)
- [**StringBuilder**](#stringbuilder)
- [**재귀 Recursive 와 동적 프로그래밍 Dynamic Programming**](#재귀-recursive-와-동적-프로그래밍-dynamic-programming)
  - [**상향식 접근법 (bottom-up approach)**](#상향식-접근법-bottom-up-approach)
  - [**하향식 접근법 (top-down approach)**](#하향식-접근법-top-down-approach)
  - [**반반 접근법 (half-and-half approach)**](#반반-접근법-half-and-half-approach)
  - [**동적계획법 \& 메모이제이션**](#동적계획법--메모이제이션)
    - [Recursive Fibo](#recursive-fibo)
    - [하향식 동적 프로그래밍 (메모이제이션)](#하향식-동적-프로그래밍-메모이제이션)
    - [상향식 동적 프로그래밍](#상향식-동적-프로그래밍)
- [**최단경로 문제**](#최단경로-문제)
  - [0-1BFS](#0-1bfs)
  - [다익스트라 알고리즘](#다익스트라-알고리즘)
  - [벨만-포드 알고리즘](#벨만-포드-알고리즘)
    - [음수간선](#음수간선)
  - [SPFA (Shortest Path Faster Algorithm) - `벨만-포드 알고리즘 보완`](#spfa-shortest-path-faster-algorithm---벨만-포드-알고리즘-보완)
- [**최소 신장 트리 `Minimum Spanning Tree`**](#최소-신장-트리-minimum-spanning-tree)
  - [**`Kruskal Algorithm` 크루스칼 알고리즘**](#kruskal-algorithm-크루스칼-알고리즘)
  - [**`Prim` 프림 알고리즘**](#prim-프림-알고리즘)
  - [✋ `Kruskal` , `Prim` 차이점](#-kruskal--prim-차이점)
- [**`LIS` 최장 증가 부분 수열 알고리즘**](#lis-최장-증가-부분-수열-알고리즘)
- [**`Backtracking` 백트래킹 특징**](#backtracking-백트래킹-특징)
- [**Kadane’s Algorithm**](#kadanes-algorithm)
- [**Floyd's Cycle Finding Algorithm**](#floyds-cycle-finding-algorithm)
- [오일러 회로 `Eulerian Circuit`](#오일러-회로-eulerian-circuit)
- [오일러 경로 `Eulerian Trail`](#오일러-경로-eulerian-trail)

<!-- /TOC -->

# **알고리즘을 왜 하고, 얼마나 깊게 해야할까**

> 컴퓨터를 이용해 문제를 풀어야할 때 같으 문제라도 여러 가지 해결 방법이 있을 수 있다.  
> 문제의 크기가 작다면 올바른 답을 내기만 하면 어떤 방법을 사용하든 크게 관계될 것이 없지만,  
> 문제의 크기가 크다면 **시간과 공간을 효율적으로 이용하는 방법을 찾는것이 중요해진다.**  
> 알고리즘을 공부하는 주된 이유는 그 사고 체계와 접근 방법이 같은 문제를 풀더라도  
> **훨씬 더 적은 자원으로도 해결할 수 있도록 심지어 해결 불가능한 것을 가능하게 만들어주기 때문이다.**  
> 어떤 작업에 합당한 알고리즘을 배우는 것뿐만 아니라 **알고리즘적 방법론들이 가진 성능 차이를 비교할 수 있는 안목을 가질 수 있다.**  
> - 알고리즘 개정 4판
  
> 문제를 많이 푼다면 구현 실력이나, 자신이 아는 요소(여기서는 그게 알고리즘이 되겠죠)를 적재적소에 활용하는 능력을 키울 수 있는 점은 분명하지만,  
> 이 알고리즘들이 모든 컴퓨터 분야에 필수인 것은 절대 아니고, 오히려 실무에서는 몰라도 큰 상관이 없을 때가 많습니다.  
> 고급 알고리즘이 꼭 좋은 것도 아닙니다.  
> 실무와 개발에 있어서는 유지보수성과 협업을 신경쓰는 것도 굉장히 중요한데요.  
> 성능은 좋지만 그렇게 필수적이지는 않았는데도 구현이 난해한 알고리즘을 너무 남발하다 보면,  
> 시간이 지나고 자신의 코드를 되돌아볼 때나 다른 사람과 협업해야 할 때,  
> 다른 사람이 이어서 작업해야 할 때 등 상당히 골치 아파지는 경우도 있을 것입니다.  
> (물론, 항상 좋은 것이 아니랬지 성능이 무조건적으로 우선인 곳에서는 잘 사용할 때도 있습니다.)  
> (리눅스의 스케쥴러가 레드블랙트리로 구현되어 있는 것이 좋은 예)  
> 어느 정도 가벼이 취미로 하기에 재밌기도 하고, 이때는 취미를 하면서 코딩/구현 실력도 늘릴 수 있다는 점에서 컴퓨터분야 종사자의 취미생활로써는 굉장히 도움이 많이 되지만,  
> 주 목적이 되는 일은 흔치 않을 것입니다.  
> 덤으로 제가 모든 개발자가 기본소양(?) 정도로 갖추면 좋겠을 지식은 기초 자료구조들 및 그리디, DP 정도가 뭔지를 아는 것이 끝입니다.  
> 또 어느때나 물어보면 즉시 구현이 가능할 정도도 아니고, 아는 것만으로 충분하다고 봅니다.  
> - [출처](https://blog.naver.com/kks227)
  
  
- **[Cracking The Coding Interview 6th Edition In LeetCode](https://leetcode.com/discuss/general-discussion/1152824/cracking-the-coding-interview-6th-edition-in-leetcode#4)**
- **알고리즘 개정 4판 (빨간책)**
  - [예제 코드](https://algs4.cs.princeton.edu/code/)
  - [`coursera` 알고리즘 1부](https://www.coursera.org/learn/algorithms-part1)
  - [`coursera` 알고리즘 2부](https://www.coursera.org/learn/algorithms-part2)

***

# **`Binary Search` 이분,이진 탐색**
- 이진 탐색 알고리즘은 **정렬된 원소 리스트를 받아 리스트에 원하는 원소가 있을경우 그 원소의 위치를 반환**, 없을경우 null을 반환함.
- 시간복잡도는 O(log n)으로 매우 빠른편.

![](imgs/binary-search.png)

- [https://velog.io/@ming/](https://velog.io/@ming/%EC%9D%B4%EB%B6%84%ED%83%90%EC%83%89Binary-Search)

## Basic Template

- 배열의 단일 인덱스에 접근하여 검색할 때 사용 된다.
- Initial Condition : `left = 0, right = length-1`
- Termination : `left > right`
- Searching Left : `right = mid-1`
- Searching Right : `left = mid+1`


```java
int binarySearch(int[] nums, int target){
  if(nums == null || nums.length == 0)
    return -1;

  int left = 0, right = nums.length - 1;
  while(left <= right){
    // Prevent (left + right) overflow
    int mid = left + (right - left) / 2;
    if(nums[mid] == target){ return mid; }
    else if(nums[mid] < target) { left = mid + 1; }
    else { right = mid - 1; }
  }

  // End Condition: left > right
  return -1;
}
```


***

# **그래프**
- **노드와 그 노드를 연결하는 간선(edge)를 하나로 모아 놓은 것이다.**
- 방향성이 있을 수도 있고 없을 수도 있다.
- **여러 개의 고립된 부분 그래프 (isolated subgraphs)로 구성될 수 있다.**
- 모든 정점간에 경로가 존재하는 그래프는 **연걸 그래프**라고 부른다.
- 그래프에는 사이클이 존재할 수도 있고 존재하지 않을 수도 있다.

## **인접 리스트**
- 그래프를 표현할 때 사용되는 가장 일반적인 방법이다.

![](imgs/TreeAndGraphAdjacencyList.png)


## **인접 행렬**
- `N*N` 행렬 로써 `행렬[i][j]`에 정보가 존재한다면 **i에서 j로의 간선**이 있다는 뜻이다.
- 무방향 그래프를 인접 행렬로 표현한다면 이 행렬은 **대칭 행렬(symmetric matrix)**이 된다.

![](imgs/TreeAndGraphAdjacencyMatrix.png)

## **그래프 탐색**

### 깊이 우선 탐색 (DFS)
- **루트 노드 (혹은 다른 임의의 노드)에서 시작해서 다음 분기로 넘어가기 전에 해당 분기를 완벽하게 탐색하는 방법**
- 모든 노드를 방문하고자 할 때 더 선호되는 편이다.
- [백준(DFS , 백트래킹) - N과 M](https://www.acmicpc.net/workbook/view/9372)
- **[Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)**

### 넓이 우선 탐색 (BFS)
- 노드 사이의 최단 경로 혹은 임의의 경로를 찾고 싶을 때 사용한다.
- 큐(Queue)를 사용하여 구현한다.

- **[Find if Path Exists in Graph](https://leetcode.com/problems/find-if-path-exists-in-graph/)**

### 양방향 탐색
- **출발지와 도착지 사이에 최단 경로를 찾을 때 사용된다.**
- 기본적으로 , 출발지와 도착지 두 노드에서 동시에 너비 우선 탐색을 사용한 뒤 , 두 탐색 지점이 충돌하는 경우에 경로를 찾는 방식이다.

![](imgs/TreeAndGraph8.png)

> 추가로 읽을 거리 - 위상정렬 , 다익스트라 , AVL , 레드블랙트리

# **트리**
- 노드로 이루어져있고 그래프의 한 종류인 자료구조이다.
- 트리는 하나의 루트 노드를 갖는다.
  - *꼭 가질 필요는 없지만 프로그래밍 면접에서 사용하는 트리에선 맞는 말이다.*
- 루트노드는 0개 이상의 자식 노드를 갖고 있다.
- 그 자식 노드 또한 0개 이상의 자식 노드를 갖고 있고 , 이는 반복적으로 정의된다.
- **사이클이 존재할 수 없다.**
- 노드들은 특정 순서로 나열될 수도 있고 그렇지 않을 수도 있다.
- 각 노드는 부모 노드로의 연결이 있을 수도 있고 없을 수도 있다.

## **이진 트리 (Binary Tree)**
- **이진 트리는 각 노드가 최대 두 개의 자식을 갖는 트리**를 말한다.
- [구간 합 구하기](https://www.acmicpc.net/problem/2042), [구간 곱 구히기](https://www.acmicpc.net/problem/11505), [최소값과 최대값](https://www.acmicpc.net/problem/2357) 이 세그먼트 트리 문제들은 이진트리에 대한 응용이다.

## **이진 탐색 트리 (Binary Search Tree)**
- **`모든 왼쪽 자식들 <= n < 모든 오른쪽 자식들` 속성은 모든 노드 n에 대해서 반드시 참이어야 한다.**
- 부등식의 경우에 대해서는 바로 아래 자식뿐만 아니라 **내 밑에 있는 모든 자식 노드들에 대해서 참이어야 한다.**

> 같은 값을 처리하는 방식에 따라 이진 탐색 트리는 약간씩 정의가 달라질 수 있다.  
> 어떤 곳에서는 중복된 값을 가지면 안 된다고 나오고 , 또 다른 곳에서는 중복된 값은 오른 쪽 혹은 양쪽 어느 곳이든 존재할 수 있다고 나온다.

![](imgs/TreeAndGraph1.png)

- 모든 노드에 대해서 그 **왼쪽 자식들의 값이 현재 노드 값보다 작거나 같도록 하고** , **오른쪽 자식들의 값은 현재 노드의 값보다 반드시 커야 한다**.

## **완전 이진 트리 (Complete Binary Search)**
- **트리의 모든 높이에서 노드가 꽉 차 있는 이진트리를 말한다.**
- 마지막 레벨은 꽉 차 있지 않아도 되지만 **왼쪽에서 오른쪽으로 채워져야 한다.**

![](imgs/TreeAndGraph2.png)

## **전 이진 트리 (Full Binary Tree)**
- **모든 노드의 자식이 없거나 정확히 두 개 있는 경우를 말한다.**
- 즉, *자식이 하나만 있는 노드가 존재해서는 안된다.*

![](imgs/TreeAndGraph3.png)

## **포화 이진 트리 (Perfect Binary Tree)**
- **전 이진 트리**이면서 **완전 이진 트리**인 경우를 말한다.
- **모든 말단 노드는 같은 높이에 있어야 하며 , 마지막 단계에서 노드의 개수가 최대가 되어야 한다.**
- 노드의 개수는 정확히 **2<sup>k-1</sup>(k는 트리의 높이)** 이다.

## **이진 트리 순회**

### 중위 순회 (in-order traversal)
- **왼쪽 가지 , 현재 노드 , 오른쪽 가지** 순서로 노드를 방문하는 것이다.
- **이진 탐색 트리를 이 방식으로 순회한다면 오름차순으로 방문하게 된다.**
- *가장 빈번하게 사용된다.*

### 전위 순회 (pre-order traversal)
- **자식 노드보다 현재 노드를 먼저 방문하는 방법을 말한다.**
- 가장 먼저 방문하게 될 노드는 언제나 루트이다.

### 후위 순회 (post-order traversal)
- **모든 자식 노드들을 먼저 방문한 뒤 마지막에 현재 노드를 방문하는 방법을 말한다.**
- 후위 순회에서 가장 마지막에 방문하게 될 노드는 언제나 루트이다.

![](imgs/TreeAndGraphTraversal.png)


## **이진 힙 (최소 힙)**
- **최대 힙은 원소가 내림차순으로 정렬되어 있다는 점만 다를 뿐 , 최소 힙과 완전히 같다.**
- 최소 힙은 트리의 마지막 단계에서 오른쪽 부분을 뺀 나머지 부분이 가득 채워져 있다는 점에서 완전 이진 트리이다.
- **각 노드의 원소가 자식들의 원소보다 작다는 특성이 있다.**

![](imgs/TreeAndGraph4.png)

- **삽입 (Insert)**
  - **원소를 삽입할 때는 언제나 트리의 밑바닥에서부터 삽입을 시작한다.**
  - *완전 트리의 속성에 위배되지 않게 새로운 원소는 밑바닥 가장 오른쪽 위치로 삽입된다.*
  - 새로 삽입된 원소가 제대로 된 자리를 찾을 때 까지 부모 노드와 교환해 나간다.

![](imgs/TreeAndGraph5.png)

- **최소 원소 뽑아내기 (Extract_Min)**
  - 최소 원소는 가장 위에 놓이기 때문에 최소 원소를 찾기란 쉬운 일이다.
  - **이 최소 값을 어떻게 힙에서 제거하느냐가 까다로운 일이다.**
  1. 최소 원소를 제거한 후에 힙에 있는 가장 마지막 원소(밑바닥 가장 왼쪽에 위치한 원소)와 교환한다.
  2. 최소 힙의 성질을 만족하도록 , 해당 노드를 자식 노드와 교환해 나감으로 써 밑으로 내보낸다. (*자식들 중 최소 값을 선택한다.*)

![](imgs/TreeAndGraph7.png)

> ![](imgs/TreeAndGraph6.png)
> - [출처 및 최대 힙](https://gmlwjd9405.github.io/2018/05/10/data-structure-heap.html)

# 균형 탐색 트리

## `2-3` 탐색 트리

## 레드 블랙 트리

# **트라이 Trie (접두사 트리)**
- **각 노드에 문자를 저장하는 자료구조 이다.**
- 따라서 트리를 아래쪽으로 순회하면 단어가 하나 나온다.
- **접두사를 빠르게 찾아보기 위한 아주 흔한 방식이다.**
- NULL 노드라고도 불리우는 **`*`노드** 는 종종 단어의 끝을 나타낸다.
  - **`*`노드** 의 실제 구현은 특별한 종류의 자식 노드로 표현될 수도 있다.
  - 아니면 **`*`노드** 의 **부모 노드 안에 boolean flag 새로 정의함으로써 단어의 끝을 표현할 수도 있다.**

![](imgs/TreeAndGraphTrie.png)

- **유효한 단어 집합을 이용하는 많은 문제들은 트라이를 통해 최적화 할 수 있다.**

***

# **TreeSet**
- 객체를 중복해서 저장할 수 없고 저장 순서가 유지되지 않는다는 **Set**의 성질을 그대로 가지고 있다.
- **이진 탐색 트리** 구조로 되어 있다.
- 추가와 삭제에는 시간이 조금 더 걸리지만 **정렬,검색**에 높은 성능을 보이는 자료구조 이다.
- 생성자의 매개변수로 Comparator객체를 입력하여 정렬 방법을 임의로 지정해 줄 수도 있다.
- **레드-블랙트리**로 구현되어 있다.
  - 부모노드보다 작은 값을 가지는 노드는 왼쪽 자식으로 ,
  - 큰 값을 가지는 노드는 오른쪽 자식으로 배치하여 균형을 맞춘다.

![](imgs/redblackTree.png)

- [문제 - K번째 큰 수](https://jdalma.github.io/docs/algorithm/javaAlgorithm/section4/#treeset-k%EB%B2%88%EC%A7%B8-%ED%81%B0-%EC%88%98-%EC%8B%A4%ED%8C%A8)

***

# 우선순위 큐 **2.4.3 참조** 

우선순위 큐는 **추상 데이터 타입의 전형적인 예**이다.  
기본적인 큐나 스택의 사용 방법과 비슷하지만, 우선순위에 따른 동작을 효율적으로 구현하는 것은 좀 더 어렵다.  
**스택/큐 구현과 대비해 우선순위 큐를 구현하는 것의 중요한 차이는 최악 조건에서 삽입 작업 또는 최대 키 값 항목 삭제 작업이 로그 시간 성능을 보이는 것이다.**  
- 배열을 이용한 기초적인 구현은 정렬을 삽입에 하거나 값을 꺼낼 때 하던 결국은 `N`이지만, 우선순위 큐로는 `logN`이 걸린다.
- [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) 이 문제를 아래와 같은 방식으로 풀면
  - Runtime 346ms Beats 98.78%of users with Kotlin
  - Memory 49.84mb Beats 93.90%of users with Kotlin
  - 위와 같은 성능이 나온다.

우선순위 큐의 기본 동작들을 효율적으로 할 수 있게 해주는 힙에 대해 알아보자.  

## **힙**

> `N`개의 키를 가진 우선순위 큐에서, 힙의 삽입 작업은 `1 + logN`이하의 비교 연산을 소요하고, 최대 키 값 항목 삭제 작업은 `2 logn`이하의 비교 연산을 소요한다.

- **`Complete Binary Tree` 완전 이진 트리 이다.**
- **모든 노드에 저장된 값들은 자식 노드들의 것보다 크거나 같다.**
- **일종의 반정렬 상태(느슨한 정렬 상태) 를 유지한다.**
  - 큰 값이 상위 레벨에 있고 작은 값이 하위 레벨에 있다는 정도
  - 간단히 말하면 부모 노드의 키 값이 자식 노드의 키 값보다 항상 큰(작은) 이진 트리를 말한다.
- **힙을 저장하는 표준적인 자료구조는 배열** 이다.
  - 구현을 쉽게 하기 위하여 배열의 첫 번째 인덱스인 0은 사용되지 않는다.
  - 특정 위치의 노드 번호는 새로운 노드가 추가되어도 변하지 않는다.
  - 예를 들어 루트 노드의 오른쪽 노드의 번호는 항상 3이다.
- **힙에서의 부모 노드와 자식 노드의 관계**
  - **왼쪽 자식의 인덱스 = `(부모의 인덱스) * 2`**
  - **오른쪽 자식의 인덱스 = `(부모의 인덱스) * 2 + 1`**
  - **부모의 인덱스 = `(자식의 인덱스) / 2`**

![](imgs/heap.png)

[출처](https://gmlwjd9405.github.io/2018/05/10/data-structure-heap.html)

- **배열 크기 조정**
  - 삽입과 삭제에서 배열 사이즈를 확인하며 자동으로 조정하는 버전을 만들 수 있다.
  - 로그 시간 성능 경곗값은 우선순위 큐의 크기가 임의적이고 배열의 크기가 변경되는 것은 **분할 상쇄 성능**이 된다. [2.4.22 참조](https://github.com/reneargento/algorithms-sedgewick-wayne/blob/master/src/chapter2/section4/Exercise22_ArrayResizing.java)

### **상향식 힙 복구(swim)**

**배열의 끝부분에 새로운 원소를 추가(삽입)할 때 발생한다.**  
어떤 노드가 부모 노드보다 커지면서 힙의 순서가 어긋난다면 그 노드를 부모 노도와 교환하여 힙을 복구할 수 있다.  
교환 후에도 새로운 부모와 비교해서 자식 노드가 더 크다면 같은 방식으로 교환해 나간다. (거슬러올라간다.)  

```java
private void swim(int k) {
    while(k > 1 && this.less(k / 2, k)) {
        this.exch(k / 2, k);
        k /= 2;
    }
}
```

### **하향식 힙 복구(sink)**

**힙의 최상위에서 원소를 꺼내고(최대 또는 최소 항목 삭제) 힙의 바닥에 있는 항목을 힙의 꼭대기로 옮긴 후에 발생한다.**   
어떤 노드가 두 자식 노드들 중 어느 하나보다 작아서 힙 순서가 위반되었다면 그 노드를 자식 노드들 중에서 큰 노드와 교환함으로써 순서를 바로 잡을 수 있다.  
교환 후에도 자식 노드들 보다 작다면 똑같이 교환해나가면서 순서를 바로 잡는다.  

```java
private void sink(int k) {
    while(true) {
        if (2 * k <= this.n) {
            int j = 2 * k;
            if (j < this.n && this.less(j, j + 1)) {
                ++j;
            }
            if (this.less(k, j)) {
                this.exch(k, j);
                k = j;
                continue;
            }
        }
        return;
    }
}
```

***

# **StringBuilder**

- **[문자열(String) 객체가 저장되는 String Pool에 대하여](https://dololak.tistory.com/718)**
- **[String은 항상 StringBuilder로 변환될까?](https://siyoon210.tistory.com/160)**

> ✋ `String.intern()`

```java
    public static void main (String[] args) {
    	String a = "테스트";
    	String b = new String("테스트");
    	
    	System.out.println("a equals b : " + a.equals(b));
    	System.out.println("a == b : " + (a == b));

    	b = b.intern();
    	System.out.println("a equals b : " + a.equals(b));
    	System.out.println("a == b : " + (a == b));
    	
//    	a equals b 	: true
//    	a == b 		: false
//    	a equals b 	: true
//    	a == b 		: true    	
    }
```

***

# **재귀 Recursive 와 동적 프로그래밍 Dynamic Programming**
- 재귀적 해법은 , **부분 문제(subproblem)**에 대한 해법을 통해 완성된다.
  1. 단순히 `f(n-1)`에 대한 해답에 무언가를 더하거나, 제거하거나,
  2. 그 해답을 변경하여 `f(n)`을 계산해내거나,
  3. 데이터를 반으로 나눠 각각에 대해서 문제를 푼 뒤 이 둘을 **병합 (merge)**하기도 한다.
- 주어진 문제를 부분문제로 나누는 3가지 방법
  - 상향식 (bottom-up)
  - 하향식 (top-down)
  - 반반 (half-half)

## **상향식 접근법 (bottom-up approach)**
- 가장 직관적인 경우가 많다.
- **이 접근법은 우선 간단한 경우들에 대한 풀이법을 발견하는 것으로부터 시작한다.**
- 리스트를 예로 들어보면,
  1. 처음에는 원소 하나를 갖는 리스트로부터 시작한다.
  2. 다음에는 원소를 두 개가 들어 있는 리스트에 대한 풀이법을 찾고,
  3. 그 다음에는 세 개 원소를 갖는 리스트에 대한 풀이법을 찾는다.
  4. 이런 식으로 계속해 나간다.
- 📌 **이 접근법의 핵심은 , 이전에 풀었던 사례를 확장하여 다음 풀이를 찾는다는 점이다.**

## **하향식 접근법 (top-down approach)**
- 해당 접근법은 덜 명확해서 복잡해 보일 수 있다.
- 이러한 문제들은 **어떻게 하면 `N`에 대한 문제를 부분 문제로 나눌 수 있을지 생각해 봐야 한다.**
- 나뉜 부분 문제의 경우가 서로 겹치지 않도록 주의한다.

## **반반 접근법 (half-and-half approach)**
- 위에 소개된 2개의 접근법 외에 데이터를 절반으로 나누는 방법도 종종 유용하다.
- **이진 탐색**은 **반반 접근법**을 이용한 탐색 방법이다.
- **병합 정렬** 또한 **반반 접근법**을 이용한 정렬 방법이다.
  - 배열 절반을 각각 정렬한 뒤 이들을 하나로 병합한다.

## **동적계획법 & 메모이제이션**
- **동적 프로그래밍은 거의 대부분 재귀적 알고리즘과 반복적으로 호출되는 부분문제를 찾아내는 것이 관건이다.**
- 이를 찾은 뒤에는 나중을 위해 현재 결과를 캐시에 저장해 놓으면 된다.

### Recursive Fibo

![](imgs/Fibo.png)

```java
int fibo(int i){
    if (i == 0) return 0;
    if (i == 1) return 1;
    return fibo(i - 1) + fibo(i - 2);
}
```

- 트리의 말단 노드는 기본 경우인 `fibo(1)`아니면 `fibo(0)`인 것을 알 수 있다.
- **각 호출에 소요되는 시간이 `O(1)`이므로 트리의 전체 노드의 개수와 수행 시간은 같다.**
- **O(2<sup>n</sup>)** 개의 노드를 갖게 되며 총 호출 횟수가 수행 시간이 된다.

### 하향식 동적 프로그래밍 (메모이제이션)
- **`fibo(i)`를 계산할 때 마다 이 결과를 캐시에 저장하고 나중에는 저장된 값을 사용하는 것이 좋다.**

```java
public int fibo(int i){
    if (i == 0) return 0;
    else if (i == 1) return 1;
    else if (memo[i] != 0) return memo[i];
    return memo[i] = fibo(i - 1) + fibo(i - 2);
}
```

![](imgs/memoFibo.png)

- **사각형 부분은 캐시값을 그대로 사용한 부분이며 , `O(N)` 수행 시간이 걸린다.**


### 상향식 동적 프로그래밍

```java
public int fibo(int n){
    if(n == 0) return 0;
    else if(n == 1) return 1;
    int[] memo = new int[n];
    memo[0] = 0;
    memo[1] = 1;
    for(int i = 2 ; i < n ; i++){
        memo[i] = memo[i - 1] + memo[i - 2];
    }
    return memo[n - 1] + memo[n - 2];
}
```

- `fibo(0)` 과 `fibo(1)`을 계산하고 , 이 둘을 이용해(이전의 결과를 이용해) 위로 점차 올라간다.
- `memo[i]`는 `memo[i + 1]` 과 `memo[i + 2]`를 계산할 때만 사용될 뿐 , 그 뒤에는 전혀 사용되지 않는다.
  - 따라서 `memo` 배열 말고 변수 몇 개를 사용해서 풀 수도 있다.


```java
public int fibo(int n){
    if(n == 0) return 0;
    int a = 0;
    int b = 1;
    for(int i = 2 ; i < n ; i++){
        int c = a + b;
        a = b;
        b = c;
    }
    return a + b;
}
```

- 피보나치 수열의 마지막 숫자 두 개를 `a`와 `b`변수에 저장하도록 바꾼 결과다.

***

# **최단경로 문제**
- 최단 경로 문제란 두 노드를 잇는 가장 짧은 경로를 찾는 문제입니다. 
- **가중치가 있는 그래프(Weighted Graph)에서는 엣지 가중치의 합이 최소가 되도록 하는 경로를 찾으려는 것이 목적입니다.**
- 최단 경로 문제엔 다음과 같은 변종들이 존재합니다.
- **단일 출발(single-source) 최단경로** : 단일 노드 v에서 출발하여 그래프 내의 모든 다른 노드에 도착하는 가장 짧은 경로를 찾는 문제.
- **단일 도착(single-destination) 최단경로** : 모든 노드들로부터 출발하여 그래프 내의 한 단일 노드 v로 도착하는 가장 짧은 경로를 찾는 문제. 그래프 내의 노드들을 거꾸로 뒤집으면 단일 출발 최단경로문제와 동일.
- **단일 쌍(single-pair) 최단 경로** : 주어진 꼭지점 u와 v에 대해 u에서 v까지의 최단 경로를 찾는 문제.
- **전체 쌍(all-pair) 최단 경로** : 그래프 내 모든 노드 쌍들 사이의 최단 경로를 찾는 문제.

> ✋ 
> - **플로이드-와샬 알고리즘**은 **전체 쌍 최단 경로**문제에 널리 쓰인다.
> - **다익스트라 알고리즘** , **벨만-포드 알고리즘** , **SPFA**는 **단일 출발 최단경로**문제를 푸는데 적합합니다. 하지만 여기에서 **조금 더 응용하면 나머지 세 문제도 풀 수 있습니다.**

- **출처 [ratsgo.github.io](https://ratsgo.github.io/data%20structure&algorithm/2017/11/25/shortestpath/)**

## [0-1BFS](https://justicehui.github.io/medium-algorithm/2018/08/30/01BFS/)
- [벽 타기](https://www.acmicpc.net/problem/23563)
- [숨바꼭질 3](https://www.acmicpc.net/problem/13549)

## 다익스트라 알고리즘
- 그래프의 간선에 가중치를 부여돼어있을 때 , `현재 위치에서 목표 위치까지 최단 경로`는?(**단일 출발 최단경로**문제) **다익스트라 알고리즘**
- 📌 **사이클이 있을수도 있는 가중 방향 그래프에서 두 지점간의 최단 경로를 찾는 방법이다.**
- **참고 예제**
  - [백준 - 최단경로](https://jdalma.github.io/docs/algorithm/2021y10m/#%EB%B0%B1%EC%A4%80-%EB%8B%A4%EC%9D%B5%EC%8A%A4%ED%8A%B8%EB%9D%BC---%EA%B8%B0%EC%B4%88%EB%AC%B8%EC%A0%9C-%EC%B5%9C%EB%8B%A8%EA%B1%B0%EB%A6%AC)
  - [백준 - 숨바꼭질3](https://jdalma.github.io/docs/algorithm/2021y09m/#-%EB%B0%B1%EC%A4%80-bfs-%EC%88%A8%EB%B0%94%EA%BC%AD%EC%A7%883)
- **동작 원리**
  1. `s`에서 시작한다．
  2. `s`의 유출 간선의 개수만큼 우리 자신을 복제한 뒤 해당 간선을 걸어간다.`(s, x)`
  의 가중치가 `5`라면 `x`에 도달하는 데 실제로 **5분**이 걸린다는 뜻이다．
  3. **노드에 도착하면 이전에 누가 방문했었는지 확인한다. 만약 방문했었다면 거기
  서 멈춘다.** `s`에서 시작한 다른 누군가가 이미 우리보다 빨리 도착했기 때문에
  자동으로 현재 경로는 다른 경로보다 더 빠를 수 없게 된다. **아무도 도착한 적
  이 없다면 , 다시 우리 자신을 복제한 뒤 가능한 모든 경로로 나아간다.**
  4. 먼저 `t`에 도착하는사람이 이긴다.

![](imgs/dijkstra.png)

<!-- |    |**a**|**b**|**c**|**d**|**e**|**f**|**g**|**h**|**i**|
|----|---|---|---|---|---|---|---|---|---|
|**a**|0|5|3||2|||||
|**b**||0||2||||||
|**c**||1|0|1||||||
|**d**|1|||0|||2|1||
|**e**|1||||0|||4|7|
|**f**||3||||0|1|||
|**g**|||3||||0||2|
|**h**|||2||||2|0||
|**i**|||||||||0| -->


- `path_weight[node]` : 시작 지점에서 `node`로의 최단 경로의 길이가 적혀있다.
  - `path_weight[0]`만 **0**으로 초기화 되어 있고 , 나머지는 모두 무한대 값으로 초기화 된다.
- `previous[node]` : 현재까지의 최단 경로가 주어졌을 때 , 각 노드를 방문하기 이전 노드의 정보가 적혀있다.
- `remaining` : **모든 노드에 대한 우선순위 큐 이다.**
  - 각 노드의 우선순위는 `path_weight`에 의해 정의된다.
- **`remaining`이 빌 때까지 노드를 꺼내 와서 다음을 수행한다.**
    1. `remaining`에서 `path_weight`가 가장 작은 노드를 선택한다.
       - 이노드를 `n`이라 하자.
    2. 📌 **인접한 노드들에 대해서 `path_weight[x]`(현재까지의 `a`에서 `x`로의 최단 경로)와 `path_weight[n] + edge_weight[(n , x)]`를 비교한다.**
        - `a`에서 `x`로 가는 현재까지의 경로보다 더 짧은 경로가 존재한다면 , `path_weight`와 `previouse`를 갱신한다.
    3. `remaining`에서 `n`을 삭제한다.
    4. `remaining`이 비면 `path_weight`에는 `a`에서 각각의 노드로의 최단 경로의 거리가 들어있게 된다.
- **위의 `previous`를 쫓아가면서 최단 경로를 재구성할 수 있다.**
    1. `n`의 첫번째 값은 `a`가 된다. 
    2. 인접한 노드 `(b , c , e)`를 보고 `path_weight (각각 5 , 3 , 2)` 와 `previous(a)`를 갱신하고 `remaining`에서 `a`를 삭제한다.
    3. 그 다음으로 작은 노드인 `e`를 선택한다. `path_weight[e]`를 2로 갱신했었다. 
    4. `e`의 인접한 노드는 `h`,`i`이므로 `path_weight (각각 6 , 2)`와 `previous`를 갱신한다. 
       - 6은 `path_weight[e](2) + (e , h)(4)`의 결과이다.
    5. 그 다음으로 작은 노드는 3인 `c`이다. 인전합 노드는 `b`와 `d`이다. 
    6. `path_weight[d]`는 무한대 값이므로 4로 갱신한다. `path_weight[c] + weight(c , d)`, `path_weight[b]`는 이전에 5였지만 , `path_weight[c] + weight(c , b) (3+1=4)`가 5보다 작으므로 `path_weight[b]`는 4로 갱신되고 `previous`는 `c`로 갱신된다.
       - `a`에서 `b`로 가는 경로가 `c`를 통해서 가는 경로로 개선되었다는 뜻이다.
- 아래의 다이어그램은 `path_weight`(왼쪽)와 `previous`(오른쪽)가 변하는 것을 단계별로 보여준다.


![](imgs/dijkstraDiagram.png)

## 벨만-포드 알고리즘
- **출처 [ratsgo.github.io](https://ratsgo.github.io/data%20structure&algorithm/2017/11/27/bellmanford/)**
- **단일 출발 최단경로**문제에 적합하며 , 가중치가 **음수**일 때도 사용 가능하다.
  - *하지만 , 다익스트라 알고리즘에 비해 느리므로 , **가중치가 모두 양수일 경우에는 다익스트라 알고리즘을 사용한다.***
- **동작 원리**

![](imgs/bellmanFord2.png)

- [벨만-포드 참고](https://coder-in-war.tistory.com/entry/%EA%B0%9C%EB%85%90-38-%EB%B2%A8%EB%A7%8C%ED%8F%AC%EB%93%9CBellman-Ford-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98)
  - `(vertex - 1) * edge`만큼만 확인하는 이유
  - `INF`일 때 `contiune`하는 이유
  - `costArr`이 `long`인 이유
  

### 음수간선

![](imgs/bellmanFord.png)

> - 위 그림에서 `c`,`d` 그리고 `e`,`f`가 사이클을 이루고 있는 걸 확인할 수 있습니다. 
> - `c`,`d`의 경우 사이클을 돌 수록 거리가 커져서 최단경로를 구할 때 문제가 되지 않습니다. 
> - 반면 **`e`,`f`의 경우 사이클을 돌면 돌수록 그 거리가 작아져 벨만-포드 알고리즘으로 최단경로를 구하는 것 자체가 의미가 없어집니다.**
> - 따라서 그래프 모든 엣지에 대해 **edge relaxation (변 경감)**을 시작노드를 제외한 전체 노드수 만큼 반복 수행한 뒤, 
> - **마지막으로 그래프 모든 엣지에 대해 edge relaxation을 1번 수행**해 줍니다. 
> - 📌 이때 한번이라도 업데이트가 일어난다면 위와 같은 **negative cycle**이 존재한다는 뜻이 되어서 결과를 구할 수 없다는 의미의 **false를 반환하고 함수를 종료하게 됩니다.** 

## SPFA (Shortest Path Faster Algorithm) - `벨만-포드 알고리즘 보완`
- [**MCMF**(Minimum Cost Maximum Flow)](https://www.crocus.co.kr/1090)에서 자주 쓰인다.
- **벨만-포드**와 같은 아이디어지만 차이점은
  - **벨만-포드**는 <span style="color:red; font-weight:bold;">모든 간선에 대해 업데이트를 진행</span>
  - **SPFA**는 <span style="color:red; font-weight:bold;">바뀐 정점과 연결된 간선에 대해서만 업데이트를 진행</span>
- **이를 위해 바뀐 정점은 큐를 이용해서 관리하고, 큐에 해당 정점이 있는지 없는지는 배열을 이용해서 체크한다.**


***

# **최소 신장 트리 `Minimum Spanning Tree`**
- [출처](https://velog.io/@fldfls/최소-신장-트리-MST-크루스칼-프림-알고리즘)
- **간선의 가중치의 합이 최소**
- 모든 노드가 연결
- 트리의 속성을 만족 (사이클이 존재하지 않는다)

## **`Kruskal Algorithm` 크루스칼 알고리즘**
- 그리디 알고리즘의 일종이다.
  - 그래프 간선들을 **가중치의 오름차순**으로 정렬해 놓은 뒤 , 사이클을 형성하지 않는 선에서 정렬된 순서대로 간선을 선택한다.
  - 연결하려는 2개의 노드가 서로 연결되지 않은 상태라면 , 2개의 노드를 서로 연결한다.
- **`Union & Find 활용` 사이클 판단하기**
  - **Union-Find 란?**
    - Disjoint Set (서로소 집합) 을 표현하는 자료구조
    - **서로 다른 두 집합을 병합하는 Union 연산**, **집합 원소가 어떤 집합에 속해있는지 찾는 Find 연산**을 지원하기에 이러한 이름이 붙었다.
    - [해당 알고리즘을 트리 구조로 구현하는 이유](https://gmlwjd9405.github.io/2018/08/31/algorithm-union-find.html)

## **`Prim` 프림 알고리즘**
- **시작 정점을 기준으로 가장 작은 간선과 연결된 정점을 선택하며 신장 트리를 확장 시키는 알고리즘**
- `정점 선택을 기반으로 하는 알고리즘`
  - 임의의 간선을 선택
  - **선택한 간선의 정점으로부터 가장 낮은 가중치를 갖는 정점을 선택**


## ✋ `Kruskal` , `Prim` 차이점
- **프림**은 **정점 위주**의 알고리즘 , **크루스칼**은 **간선 위주**의 알고리즘
- **프림**은 시작점을 정하고 , 시작점에서 가까운 정점을 선택하면서 트리로 구성하므로 그 과정에서 사이클을 이루지 않는다.
  - 최소 거라의 정점을 찾는 부분에서 자료구조의 성능에 영향을 받는다.
- **크루스칼**은 시작점을 따로 정하지 않고 최소 비용의 간선을 차례로 대입하면서 트리를 구성하기 때문에 **사이클이 이루어지는지 항상 확인 해야한다.**
  - 간선을 기준으로 정렬하는 과정이 오래 걸린다.
- **간선의 개수가 작은 경우**에는 **크루스칼**
- **간선의 개수가 많은 경우**에는 **프림**
- [참고 문제1](https://jdalma.github.io/docs/javaAlgorithm/section9/#-원더랜드-)
- [참고 문제2](https://jdalma.github.io/docs/algorithm/2022y02m/#프로그래머스-최소신장트리-섬-연결하기)

***

# **`LIS` 최장 증가 부분 수열 알고리즘**
- 원소가 n개인 배열의 일부 원소를 골라내서 만든 부분 수열 중,
- 각 원소가 이전 원소보다 크다는 조건을 만족하고, 그 길이가 최대인 부분 수열을 **최장 증가 부분 수열**이라고 합니다.

- 예를 들어, { 6, **2**, **5**, 1, **7**, 4, **8**, 3} 이라는 배열이 있을 경우, LIS는 **{ 2, 5, 7, 8 }** 이 됩니다.
- { 2, 5 }, { 2, 7 } 등 증가하는 부분 수열은 많지만 그 중에서 가장 긴 것이 { 2, 5, 7, 8 } 입니다.
-  일반적으로 최장 증가 부분 수열의 길이가 얼마인지 푸는 간편한 방법은 **DP**를 이용하는 것입니다.
    - 시간복잡도를 개선하기 위하여 LIS를 구성할 때 이분탐색을 활용한다.

[출처](https://chanhuiseok.github.io/posts/algo-49/)

***

# **`Backtracking` 백트래킹 특징**
- 백트래킹은 말 그대로 **역추적**을 의미한다
- 문제 해결을 위한 모든 경우의 수를 따져보기 위해서 일반적으로 활용되는 기법 중 하나이다.
  - `(완전 검색 (Exhaustive search), 실제로 모든 케이스를 다 직접 확인한다는 의미는 아니다)`
- 백트래킹은 우선 어떤 하나의 가능한 케이스를 확인하고, 가능하지 않다면 다시 Back하고,
- 다시 다른 가능성있는 케이스를 확인하면서 Solution이 도출될 때까지 이런 과정이 계속적으로 반복되도록 구현하게 된다.
- 따라서 **일반적으로 백트래킹은 알고리즘의 구조 특성상 재귀함수를 사용하여 구현**된다.
- 백트래킹에 있어 중요한 것은 문제의 요구사항에 따라서, 구현된 알고리즘의 연산량이 제한 시간을 넘어버리게 되는 경우가 발생한다는 것이다.
- 무작정 전 검색을 수행하는 데에는 한계가 발생할 수 밖에 없다.
- 따라서, 더 이상 탐색할 필요가 없는 후보군에 대해서는 재귀 호출을 더 이상 하지 않고 **가지치기 (Pruning 또는 Branch and bound) 하는 것이 매우 중요**합니다.
- 굳이 체크할 필요가 없다고 판단되는 후보군들을 적절히 제외시켜서, 연산 시간은 줄이면서 완전 검색이 수행되도록 하는 것이 백트래킹의 핵심이라고 할 수 있다.
- [N Queen 문제 최적화 (Backtracking)](https://jayce-with.tistory.com/17)

[출처 Jayce's Blog](https://jayce-with.tistory.com/16)

***

# **[Kadane’s Algorithm](https://sustainable-dev.tistory.com/23)**

***

# **Floyd's Cycle Finding Algorithm**

- [참고 문제](https://jdalma.github.io/docs/leetCodeStudyPlan/dataStructure1/#linked-list-linked-list-cycle)

![](imgs/floydFindCycle.png)

- `Fast`가 `null`이거나 `Fast.next`가 `null`이면 사이클이 없는 것
- `Slow`와 `Fast`가 같다면 사이클이 발생한 것이다.

```
1.
S - 1
F - 1

2.
S - 2
F - 3

3.
S - 3
F - 5

4.
S - 4
F - 3

5.
S - 5
F - 5
```

***

# 오일러 회로 `Eulerian Circuit`
- **그래프의 모든 간선을 정확히 한 번씩 방문하고 시작점으로 돌아오는 경로를 말한다**
  - *정점은 여러번 방문해도 상관없다*

![](imgs/eulerianCircuit.png)

- 위의 그림에서 여러 순서들 중 `{A , B , C , E , F , C , D , A}`는 오일러 회로이다.
- **연결 그래프의 모든 정점의 차수가 짝수이면 오일러 회로가 존재한다.**
- 어떤 연결 그래프가 오일러 회로를 가지려면 각 정점의 차수가 모두 짝수여야 한다.
  - 만약, *홀수 차수를 가진 정점이 존재한다면 오일러 회로는 존재하지 않는다.*

# 오일러 경로 `Eulerian Trail`
- **그래프의 모든 간선을 정확히 한 번씩 방문하는 경로를 말한다.**
  - *오일러 회로와는 다르게 시작점으로 돌아올 필요가 없다.*

![](imgs/eulerianTrail.png)

- 위의 그림에서는 오일러 회로가 존재할 수 없다.
  - *`E`와 `F`의 차수가 홀수이기 때문이다*
- **여러 개의 경로 중 `{F , C , B , A , D , C , E}` 가 그 중 오일러 경로의 하나이다.**
- **연결 그래프에서 홀수 차수를 가진 정점이 정확히 0개 또는 2개 존재한다면 , 오일러 경로가 존재한다.** 📌 
