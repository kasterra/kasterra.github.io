---
title: 큰 수의 이항 계수 mod K 구하는 법
tags: 수학 이항계수
katex: True
layout: post
subtitle: 큰 수의 이항계수를 효율적으로 구하는 법
image: /images/thumbnails/이항계수.png
category: math
---

# 0. 도입

우선 이 글은 [영어 원글](https://fishi.devtail.io/weblog/2015/06/25/computing-large-binomial-coefficients-modulo-prime-non-prime/)을 번역해서 약간의 재구성을 거친 것임을 밝힙니다.

이 글에서는 고등학교 때 배웠던 이항 계수를 구하는 방법에 대한 내용을 설명할 것이고, 특히 숫자가 커져서, 일반적인 방법으로는 시간이 부족할때 대응하는 방법에 대해서도 설명할 것입니다.

# 본문

종종, 조합과 관련한 문제를 풀 때, 매우 큰 수 $$N$$,$$K$$ 에 관해서, $$\binom{N}{K} \mod M$$ 을 구하게 되는 경우가 있습니다.

일반적으로 이항계수는 $$\binom{N}{K} = \frac{N!M!}{(N-K)!} (0\le k\le n)$$ 와 같이 계산합니다. 여기서 생기는 문제는 팩토리얼은 너무나 빠르게 증가하여, 이러한 방식으로 계산하는 것은 오버플로를 일으킬 가능성이 높기 때문에 적합하지 못합니다.

이러한 이유로, 이런 계산을 하게 되는 문제는 $$\binom{N}{K} \mod M$$ 을 계산하도록 한다. 이번 포스팅에서 저는 $$M$$이 소수(prime number)일때와, 소수가 아닐때의 경우를 다루어 보고자 합니다.

# 1. 모든 M에 대한 단순한 접근법

좋은 소식이 있습니다. 모든 모듈로 $$M$$에 대해서, 쉽게 계산할 수 있는 방법이 있다는 것입니다.
나쁜 소식도 있습니다. 매우 큰 숫자에는 적합하지 않다는 것입니다.

그렇지만, 나중에 우리는 매우 큰 숫자들로 구성된 문제를, 더욱 작은 숫자들로 이루어진 부분 문제들로 분해 할 수 있는 방법을 배울 것이기 때문에, 여기서 다루는 기술들을 나중에 재사용 할 수 있습니다.

## 파스칼의 삼각형 사용하기

가장 쉬운(그렇지만 가장 제한된) 방법은, 파스칼의 삼각형을 미리 계산해 두어, 바텀-업 동적 계획법으로 문제를 푸는 것입니다. 파스칼의 삼각형은 참고표(원어:lookup table) 처럼 사용되는데, $$\binom{n}{k}$$의 값은 $$n$$번째 열의 $$k$$번째 행을 참조하면 됩니다.
파스칼의 삼각형을 미리 계산하는것은 매우 쉬운데, 새로운 값을 구할려면, 이전 행의 두 이웃한 칸의 값을 합치기만 하면 되기 때문 입니다.
$$(a+b)\mod m = (a\mod m) + (b\mod m)\mod m$$ 이라는 사실을 이용하면, 우리는 모듈로 연산을 적용한 파스칼의 삼각형을 오버플로 문제 없이 얻어낼 수 있습니다.

**코드 : 모든 m에 대해서 파스칼의 삼각형을 계산하기**

```python
#!/usr/bin/env python3

"""
이항계수 nCk % m 를 계산하기 위해 파스칼의 삼각형 계산하기
"""

# Calculating Pascal's Triangle until row 'n' with modulo 'mod'
# 바텀-업 동적계획법 접근법
# 시간복잡도:	O(n**2)
# 공간복잡도:	O(n**2)

def pascal_triangle(n,mod):
    # prepare first row in table
    pascal = []
    pascal.append([1,0])

    for i in range(1,n+1):
        # 표의 새 행을 초기화
        pascal.append([])
        pascal[i] = [0]*(i+2)
        pascal[i][0] = 1

        for j in range(1,i+1):
            # 현재의 값을 구하기 위해서 이전 행의 이웃한 두 값을 참조
            pascal[i][j] = (pascal[i-1][j] + pascal[i-1][j-1])%mod

    return pascal

if __name__ == '__main__':
    n = 1009 # number of rows in pascal table
    mod = 123456 # 소수가 아님

    # mod 123456 이 적용된 파스칼의 삼각형 계산하기
    pascal = pascal_triangle(n,mod)

    # (950 C 100) mod 123456
    print(pascal[950][100]) # 정답은 24942가 될것
```

삼각형을 계산하는것은 $$O(N^2)$$의 시간이 걸립니다. 우리가 모든 가능한 n개의 행들을 계산하기 떄문입니다. 한번 삼각형을 계산하고 나면, 다른 $$n$$과 $$k$$에 대해서 그냥 배열의 값만 참조하기만 하면 되기 떄문에, $$O(1)$$의 시간 복잡도가 걸립니다.
여기서 가장 큰 문제는, 공간복잡도 에서 옵니다. $$n$$이 적은 숫자라면 큰 문제가 되지 않지만, $$n$$이 매우 큰 숫자여서 $$n^2$$가 부담이 될 정도로 큰 숫자라면, 이러한 경우에 맞추어 더 좋은 접근법을 생각해 보아야 합니다.

공간 복잡도 문제를 해결하는 매우 쉬운 방법이 있습니다. 각 줄을 계산할 때는 바로 전 행의 정보만 있으면 계산해낼 수 있습니다. 이것은 우리가 현재의 열 정보만 저장하면 된다는 것을 의미하며, 공간 복잡도는 선형$$O(n)$$ 이 됨을 알 수 있습니다.

**코드 : 모든 m에 대해서, 파스칼의 삼각형 한줄 계산하기**

```python
#!/usr/bin/env python3

"""
Calculating a row in Pascal's Triangle to determine Binomial Coefficients
"""

# 파스칼의 삼각형의 'n'번째 열을, 이전 열만을 기억하는 방법으로 계산하기
# 바텀-업 동적계획법 접근법
# 시간복잡도:	O(n**2)
# 공간복잡도:	O(n)
def pascal_triangle_row(n,mod):
    #'n' 번째 행을 위한 공간 만들기
    pascal = [0]*(n+1)
    pascal[0] = 1

    for i in range(1,n+1):
        # 보조 배열 초기화
        tmp = [0]*(n+1)
        tmp[0] = 1

        for j in range(1,i+1):
            # 현재의 값을 구하기 위해서 이전 행의 이웃한 두 값을 참조
            tmp[j] = (pascal[j] + pascal[j-1])%mod

        pascal = tmp

    return pascal

if __name__ == '__main__':
    n = 950 # row of pascal table
    mod = 123456 # not a prime

    # mod 123456이 적용된 파스칼의 삼각형 950번째 행 계산하기
    pascal_row = pascal_triangle_row(n,mod)

    # (950 C 100) mod 123456
    print(pascal_row[100]) # 정답은 24942 일것
```

이 방법이 더 큰 이항계수들을 더 나은 공간 복잡도를 통해서 계산할 수 있게 하지만, 이 방법에도 한계점이 있습니다.

첫번째 접근에서는 파스칼의 삼각형을 $$O(n^2)$$로 계산하고, 이항계수를 $$O(1)$$의 시간으로 알 수 있었습니다. 하지만 이번 접근에서는 다른 $$n$$이 주어질 때마다, 다시 삼각형을 계산해야 합니다. 따라서 이번 접근은 같은 파스칼의 삼각형의 열에 속하지 않은 여러 이항계수를 계산할 때는 좋은 방법이 되지 못합니다.

# 2. m이 소수일때의 더 나은 접근법

이 글의 주제와 관련한 많은 문제에서, $$m$$이 소수일 때 $$\binom{n}{k} \mod m$$을 구하는 상황이 많이 주어집니다. $$m$$이 소수로 주어지는것은 단지 우연히 그렇게 된것은 아니고, 소수로 모듈러 연산을 하는것은 특정한 수학적 법칙들을 이용해서 간단히 계산할 수 있기 떄문입니다.

## 페르마의 소정리 사용하기

우리는 이항계수가 $$\frac{n!}{(k!(n-k)!} = \frac{n\cdot (n-1)\cdot\cdot \cdot \cdot\cdot(n-k+1)}{k!}$$ 로 계산할 수 있다는 것을 알고 있습니다. 이 식에서 우리는 $$\{(a \mod m) \cdot(b \mod m)\}\mod m = (a\cdot b)\mod m$$라는 것을 이용해, 분자를 쉽게 계산해 낼 수 있습니다. 분자를 분모로 나누기 위해, 우리는 분자에다가 분모의 곱셈의 역원을 곱해야 합니다. 다행히, $$m$$이 정수라면, 우리는 간단히 **페르마의 소정리**를 적용할 수 있습니다.
(페르마의 소정리 : $$p$$가 소수이고, $$a$$가 $$p$$의 배수가 아니면, $$a^{p-1} ≡ 1(\mod p)$$ 이다.)

이 정리를 통해, 우리는 p가 소수일 때, $$a^{-1} ≡ a^{p-2} (\mod p)$$ 라는 것을 간단히 알 수 있고, 분모의 곱셈의 역원을 $$(k!)^{m-2}$$ 에다가 모듈러 연산을 씌운 것으로 쉽게 알 수 있습니다.
($$a^{p-1} = a^{p-2}\times a ≡ 1$$이고, $$a^{p-2}$$가, $$a\times x ≡ 1(\mod p)$$를 만족하는 $$x$$가 되기 때문)

**코드 : $$m$$이 소수고 $$k<m$$일때, 페르마의 소정리를 이용해서 계산**

```python
#!/usr/bin/env python3

"""
 k < m 이고, m 이 소수일때 nCk mod m을 페르마의 소정리로 구하는 코드
두가지 버전:
1. 미리 팩토리얼과, 곱셈의 역원들을 O(n*logn)에 계산해 놓는 방법 --> 나중에 접근할 때는 O(1)
2. 필요할 떄마다 계산하기 --> 따로 참조하는건 없음 --> 계산 할 때마다 O(n)
"""

# 모듈러 지수승(Modular Exponentiation) : b^e % mod
# 파이썬의 내장 pow(b,e,mod)가 이 함수보다는 빠르다는것을 참조할것.
def mod_exp(b,e,mod):
    r = 1
    while e > 0:
        if (e&1) == 1:
            r = (r*b)%mod
        b = (b*b)%mod
        e >>= 1

    return r

# nCk mod p 를 계산하기 위해서 페르마의 소정리를 사용
# 주의사항 : p는 소수여야 하며, k < p 여야 함p
def fermat_binom(n,k,p):
    if k > n:
        return 0

    # 분자 계산하기
    num = 1
    for i in range(n,n-k,-1):
        num = (num*i)%p

    # 분모 계산하기
    denom = 1
    for i in range(1,k+1):
        denom = (denom*i)%p

    # 분자 * 분모^(p-2) (mod p)
    return (num * mod_exp(denom,p-2,p))%p

# 페르마의 소정리를 이용해서, 팩토리얼과 역원을 미리 계산하는 코드
# 주의사항 : p 가 소수이고, k < p 일떄만 작동함
def fermat_compute(n,p):
    facts = [0]*n
    invfacts = [0]*n

    facts[0] = 1
    invfacts[0] = 1
    for i in range(1,n):
    	# 팩토리얼들과, 그에 대응하는 곱셈의 역원을 계산
        facts[i] = (facts[i-1]*i)%p
        invfacts[i] = mod_exp(facts[i],p-2,p)

    return facts, invfacts

# 미리 계산된 팩토리얼들과 역원을 이용해서 이항계수를 계산하는 코드
def binom_pre_computed(facts, invfacts, n, k, p):
    # n! / (k!^(p-2) * (n-k)!^(p-2)) (mod p)
    return (facts[n] * ((invfacts[k]*invfacts[n-k] % p))) % p

if __name__ == '__main__':
    n = 1009 # 미리 계산한 팩토리얼들의 개수
    mod = 1000000007 # 소수

    # pre-compute factorials and inverses
    facts, invfacts = fermat_compute(n,mod)

    # print (950 C 100) mod 1000000007 (전처리를 이용한 코드)
    print(binom_pre_computed(facts,invfacts,950,100,mod)) # 정답은 640644226 일것

    # (950 C 100) mod 1000000007 (전처리 없는 코드)
    print(fermat_binom(950,100,mod)) # 정답은 640644226 일것
```

팩토리얼 $$n!$$을 계산하는 것은 $$O(n)$$ 의 시간이 걸리고, 곱셈의 역원을 구하는 데에는 $$O(log_{2}n)$$의 시간이 걸립니다. 우리가 두가지 버전을 구현했다는 것에 초점을 잠시 둬 봅시다.

첫번째 버전에서는, 팩토리얼들과 그에 대응하는 곱셈의 역원을 $$O(n)$$의 메모리를 이용해서 저장했고, 시간은 $$O(nlog_{2}n)$$ 이 걸렸습니다. 그리고 한번 계산만 해놓으면, 참고해서 값을 내놓는 것은 $$O(1)$$ 이 걸린다는 이점이 있지요.

두번째 버전에서는 특별한 전처리 없이, 매번마다 분모의 곱셈의 역원을 구했습니다. 따라서, 각각 이항계수를 구할 때 마다 $$O(n)$$의 시간이 걸렸지요.

우리가 미리 참고할 자료들을 전처리를 하든 하지 않든, n의 크기에는 크게 영향을 받지 않습니다. 만약 우리가 매우 큰 $$n$$에 대해서 문제를 풀어야 한다면, 곱셈의 역원 표를 미리 전처리 해놓는 것은 $$O(n)$$의 값이 매우 클 것이기 때문에 비효율적일 것입니다. 하지만 다른 일반적인 경우에서는, 전처리를 해놓으면 나중에 걸리는 시간을 크게 절약할 수 있을 것입니다. 추후에, 전처리 없이, $$n$$이 매우 클때 사용할 수 있는 알고리즘에 대해서도 소개할 것인데, 핵심이 되는 아이디어는 같기 때문에, 전처리를 통한 알고리즘을 약간 수정하는 것으로 만들어 질 수 있습니다.

위 코드의 제목에도 볼 수 있듯, 이 코드는 $$k < m$$ 일떄만 작동합니다. 그렇지 않으면 $$k!\mod m = 0$$이 되어, 곱셈의 역원을 정의할 수 없게 되기 때문이죠. 하지만 위의 코드를 약간 수정하는 것으로 이 문제를 해결할 수 있습니다.
분자와 분모를 인수분해 했을때의 m의 차수에 대해서 비교하는 것입니다. 분자의 차수는 분모의 차수보다 작거나 같기 때문에, 이를 이용하면 분자와 분모 모든곳에서 m에 대해서 약분을 해낼 수 있을 것입니다.

**코드 : $$m$$이 소수일때, 페르마의 소정리를 이용해서 계산**

```python
#!/usr/bin/env python3

"""
페르마의 소정리를 활용해 nCk mod m, 단 m은 소수 일때 O(n)의 시간으로 구하기
"""

# 모듈러 지수승(Modular Exponentiation) : b^e % mod
# 파이썬의 내장 pow(b,e,mod)가 이 함수보다는 빠르다는것을 참조할것.
def mod_exp(b,e,mod):
    r = 1
    while e > 0:
        if (e&1) == 1:
            r = (r*b)%mod
        b = (b*b)%mod
        e >>= 1

    return r



# n!에서 p의 차수를 구함(n! = p^a * ... 일때, 가능한 최대의 a를 구함)
def fact_exp(n,p):
    e = 0
    u = p
    t = n
    while u <= t:
        e += t//u
        u *= p

    return e

# 페르마의 소정리를 사용해 nCk mod p 를 구해보자
# 분자 분모를 p의 지수승으로 약분해서 계산
# 주의사항 : p가 반드시 소수여야 함
def fermat_binom_advanced(n,k,p):
    # 말이 되지 않는 입력을 거름
    num_degree = fact_exp(n,p) - fact_exp(n-k,p)
    den_degree = fact_exp(k,p)
    if num_degree > den_degree:
        return 0

    if k > n:
        return 0

    # 분자를 계산하고 p가 몇승만큼 들어있는지를 계산한다
    num = 1
    for i in range(n,n-k,-1):
        cur = i
        while cur%p == 0:
            cur //= p
        num = (num*cur)%p

    # 분모를 계산하고 p가 몇승만큼 들어있는지를 계산한다
    denom = 1
    for i in range(1,k+1):
        cur = i
        while cur%p == 0:
            cur //= p
        denom = (denom*cur)%p

    # 분자 * 분모^(p-2) (mod p)
    return (num * mod_exp(denom,p-2,p))%p

if __name__ == '__main__':
    mod = 1000000007 # prime

    # (950 choose 100) mod 1000000007
    print(fermat_binom_advanced(950,100,mod)) # should be 640644226
```

이제 이항계수를 모듈로 연산을 하는 수 $$p$$가 소수이기만 하면, $$O(N)$$의 시간 복잡도로 구할 수 있게 되었습니다. 우리는 다음 섹션에서 특정한 상황에서 시간 복잡도를 향상 시킬 수 있는 방법을 알아 볼 것입니다. 더 나아가서, 우리는 모듈로 연산을 하는 수 $$p$$가 소수가 아닌 상황에 관해서도 계산 하는 법에 대해서도 알아볼 것입니다. 하지만, 여기서 다루는 페르마의 소정리, 파스칼의 삼각형에 관한 내용은, 나중에 다룰 접근에서의 핵심 아이디어로 사용되기 때문에, 상당히 중요합니다. 즉 이 내용들을 이해하고 있지 않다면, 뒤의 내용도 이해할 수 없다는 뜻이니까 잘 이해를 하고 넘어가시길 바랍니다.

추가적으로 제가 문제를 주로 푸는 언어는 C++ 이기 때문의 위의 파이썬 코드를 C++로 옮겨서 구현해 보았습니다. 파이썬이 다룰 수 있는 정수의 범위는 무지막지하게 크기 때문에, C++등의 다른 언어로 코드를 옮길 때는 오버플로에 주의합시다. 사실 오버플로 문제 때문에 시간 쓴거 말고는 그냥 언어에 맞게 연산자 옮긴것 뿐이지만 아무튼요.

**추가사항 : 역자가 직접 적어본 C++ 코드**

```cpp
#include <iostream>
#include <vector>
#include <utility>

using namespace std;

typedef long long ll;
typedef unsigned long long ull;

//비재귀 적으로 (x^y)%p 를 O(log y)로 구하는 코드
ll power(ll x,ull y, ll p){
    ll res = 1;     // 결과 변수 초기화

    x = x % p; //x가 p 이상일 경우를 대비해서 하는 연산

    if (x == 0) return 0; // x 가 p의 배수일경우

    while (y > 0){
        //y가 홀수라면 결과에 x를 곱한다.
        if (y & 1)
            res = (res % p) * (x % p) % p;

        //이제 y는 반드시 짝수일 것
        y = y>>1; // y = y/2
        x = (x%p)*(x%p) % p;
    }
    return res;
}

// n!에서 p의 차수를 구하는 함수
int fact_exp(ll n, ll p){
	int e = 0;
    ll u = p;
    ll t = n;
    while(u <= t){
    	e += t / u;
        u *= p ;
    }
    return e;
}

//그때그떄마다 계산하는 O(n) 코드
//p는 소수, k < p 일때 정상작동
ll fermat_binom(ll n, ll k, ll p){
	if(k > n) return 0;

    //분자 계산
    ll num = 1;
    for(ll i = n; i > n - k;--i)
    	num = (num*i) % p;

    //분모 계산
    ll denom = 1;
    for(ll i = 1;i < k + 1;++i)
    	denom = (denom * i) % p;

    return (num * power(denom,p-2,p))%p;
}

//팩토리얼과 역원을 미리 계산하는 O(n * logn) 코드
// p는 소수 , k < p 일때 정상 작동
pair<vector<ll>,vector<ll> > fermat_compute(ll n, ll p){
	pair<vector<ll>, vector<ll> > ret
    = make_pair(vector<ll>(n,0),vector<ll>(n,0));
    vector<ll>& facts = ret.first;
    vector<ll>& invfacts = ret.second;

    facts[0] = 1;
    invfacts[0] = 1;
    for(ll i = 1 ;i < n;++i){
    	facts[i] = (facts[i - 1] * i) % p;
        invfacts[i] = power(facts[i], p - 2, p);
    }

    return ret;
}

ll binom_pre_computed(vector<ll> facts,vector<ll> invfacts,ll n,ll k,ll p){
	return (facts[n] * ((invfacts[k] * invfacts[n-k] % p))) %p;
}

ll fermat_binom_advanced(ll n,ll k,ll p){
	int num_degree = fact_exp(n,p) - fact_exp(n-k,p);
    int den_degree = fact_exp(k,p);

    if(num_degree > den_degree)
    	return 0;
    if(k > n)
    	return 0;

    //분자 계산
    ll num = 1;
    for(ll i = n;i > n - k;--i){
    	ll cur = i;
        while(cur % p == 0) cur /=p;
        num = (num * cur) % p;
    }

    //분모 계산
    ll denom = 1;
    for(ll i = 1;i < k + 1;++i){
    	ll cur = i;
        while(cur % p == 0) cur /= p;
        denom = (denom * cur) % p;
    }

    return(num * power(denom,p - 2,p))%p;
}
int main(void){
	ll n = 1009; // 미리 계산할 팩토리얼들의 개수
    ll mod = 1000000007;//소수

    pair<vector<ll>, vector<ll> > temp = fermat_compute(n,mod);
    vector<ll> facts = temp.first;
    vector<ll> invfacts = temp.second;

    cout <<"binom_pre_computed : "<<binom_pre_computed(facts,invfacts,950,100,mod) << '\n';
    cout <<"fermat_binom : "<< fermat_binom(950,100,mod) << '\n';
    cout << "fermat_binom_advanced : "<<fermat_binom_advanced(950,100,mod);
	return 0;
}

```

## 뤼카의 정리(루카스의 정리) 활용

[뤼카의 정리](https://bowbowbow.tistory.com/2)는 큰 수의 이항계수를, 작은 수들의 이항계수로 쪼갤 수 있도록 해줍니다. 주어진 문제를 분해하기 위해서는, $$n$$과 $$k$$를 $$m$$진법으로 나타낼 필요가 있습니다.

즉 $$n$$과 $$k$$를
$$n = n_{0}+n_{1}\times m^{1} + n_{2}\times m^{2} +\cdot\cdot\cdot+ n_{x}\times m^x$$,
$$k = k_{0}+ k_{1}\times m^{1} + k_{2}\times m^{2} + \cdot\cdot\cdot + k_{x}\times m^{x}$$의 꼴로 나타내야 합니다. 그다음 우리는 주어진 문제를 다음과 같은 부분문제로 쪼갤 수 있습니다.

$$\binom{n}{k} = \prod_{i=0}^{x} \binom{n_i}{k_i}(mod$$ $$m)$$

이 부분문제들은 위에서 했던 것 처럼 페르마의 소정리를 이용해서 풀어낼 수 있습니다.

```python
#!/usr/bin/env python3

"""
뤼카의 정리와, 페르마의 소정리를 이용해서 m이 소수일때, nCk mod m를 구하기
"""

# 모듈러 지수승(Modular Exponentiation) : b^e % mod
# 파이썬의 내장 pow(b,e,mod)가 이 함수보다는 빠르다는것을 참조할것.
def mod_exp(b,e,mod):
    r = 1
    while e > 0:
        if (e&1) == 1:
            r = (r*b)%mod
        b = (b*b)%mod
        e >>= 1

    return r

# n!에서 p의 차수를 구함(n! = p^a * ... 일때, 가능한 최대의 a를 구함)
def fact_exp(n,p):
    e = 0
    u = p
    t = n
    while u <= t:
        e += t//u
        u *= p

    return e

# 주어진 수 n을 b진법으로 변환함
# 가장 큰 자리수(Most significant Digit)의 숫자가 배열의 맨 오른쪽에 있음
def get_base_digits(n,b):
    d = []
    while n > 0:
        d.append(n % b)
        n  = n // b

    return d

# 페르마의 소정리를 이용해서 nCk mod p을 구함
# p의 지수승으로 분자, 분모를 약분
# 주의사항 : p가 반드시 소수여야함
def fermat_binom_advanced(n,k,p):
    # 지수승이 말이 되는지를 판별
    num_degree = fact_exp(n,p) - fact_exp(n-k,p)
    den_degree = fact_exp(k,p)
    if num_degree > den_degree:
        return 0

    if k > n:
        return 0

    # 분자에서 p의 지수를 파악
    num = 1
    for i in range(n,n-k,-1):
        cur = i
        while cur%p == 0:
            cur //= p
        num = (num*cur)%p

    # 분모에서도 같은 방법으로
    denom = 1
    for i in range(1,k+1):
        cur = i
        while cur%p == 0:
            cur //= p
        denom = (denom*cur)%p

    # 분자 * 분모^(p-2) (mod p)
    return (num * mod_exp(denom,p-2,p))%p

# 뤼카의 정리를 이용해, 주어진 문제를 더 작은 부분 문제로 쪼갬
# p가 소수라고 가정하고 문제를 풀어나감
def lucas_binom(n,k,p):
	# n과 k를 p진법으로 나타냄
    np = get_base_digits(n,p)
    kp = get_base_digits(k,p)

    # (nCk) = (n0 C k0)*(n1 C k1) ... (ni C ki) (mod p)
    binom = 1
    for i in range(len(np)-1,-1,-1):
        ni = np[i]
        ki = 0
        if i < len(kp):
            ki = kp[i]

        binom = (binom * fermat_binom_advanced(ni,ki,p)) % p

    return binom

if __name__ == '__main__':
    mod = 7 # 소수

    # (950 C 100) mod 7
    print(lucas_binom(950,100,mod)) # 정답은 2 일것

```

<i class="fas fa-info-circle"></i> <strong>정보</strong><br>
뤼카의 정리는 우리의 소수 $$p$$가 계산할 이항계수의 약수인지, 실제 계산을 하지 않고 알 수 있는 방법을 제공합니다. $$m$$진법으로 나타낸, $$n$$과 $$k$$를 비교해서, $$k$$의 어느 한 자리수가 $$i$$의 같은 자리의 자리수와 같고, $$k > n$$일때, $$\binom{n}{k} ≡ 0mod$$ $$p$$일 것입니다.
{:.info}

우리는 뤼카의 정리를 이용해서, 주어진 문제를 작은 부분문제들로 쪼개서 문제를 해결했습니다. 그리고 부분문제들을 풀기 위해, 아까 전의 섹션처럼 페르마의 소정리를 이용했습니다. 숫자가 작아진 만큼, 맨 처음에 다룬 파스칼의 삼각형 또한 이용해 볼 수 있을 것입니다. 또한, $$n$$과 $$k$$가 $$m$$보다 작을 때, 뤼카의 정리는 어떠한 이점도 불러오지 않을 것입니다. 왜냐하면 가장 작은 부분문제가 원래의 문제 그 자신일 것이기 때문입니다. 하지만, 그런 경우에는, $$n$$과 $$k$$를 m진법으로 나타내는 것에 매우 작은 오버헤드가 걸릴 것일 것이기에, 문제가 주어질때, 문제를 작은 문제로 분할 해보려는 시도는 일반적으로 좋은 시도입니다.

뤼카의 정리로 문제를 분해 한 다음, 페르마의 소정리나, 파스칼의 삼각형을 통해서 작은 부분문제들을 해결합시다.

<i class="fas fa-exclamation-triangle"></i> <strong>주의</strong><br>
이번 문제에서는 전처리를 하지 않았습니다. 매우 큰 단 하나의 n 에 대해서는 전처리를 하지 않는것이 효율적입니다. 일반적으로 뤼카의 정리로 문제를 쪼개면, lookup table로 쓸 메모리가 그렇게 크지 않아서, 여러 이항계수를 구하는 상황일때는 전처리를 하는것이 효율적입니다. 우리 문제상황에서는 n이 좀 많이 커서 $$O(n)$$의 공간을 잡아먹는데, 이 비용이 결코 싸지 않아서 따로 전처리를 하지 않은 것입니다. <br><br>
**하지만** 다른 일반적인 경우에서는 전처리를 하는것은 나중에 많은 시간을 절약시켜 주는 매우 좋은 일입니다. 문제 풀이에 참고하시기 바랍니다.
{:.warning}

이렇게 모듈로 연산 하는 수가 소수일때 효율적으로 계산하는 방법에 대해서 알아 보았습니다.

# 3. m이 소수가 아닐 경우

## m의 소인수 분해에 제곱 인수가 없을 경우(square-free)

만약에 모듈러 연산을 하는 수 m이 정수가 아니라면 어떻게 해야 할 까요? 질문의 답은 $$m$$의 소인수분해에 달렸습니다. 다음에 소개할 방법은 $$m$$의 소인수 분해 결과에 제곱 인수가 없을때(square-free)에 가능한 방법입니다. 간단히 예를 들어 봅시다. $$6 = 2\times 3$$이기 떄문에 6은 제곱 인수가 없는 수입니다. 반면, $$24 = 2^{3}\times 3$$이고, 소인수 $$2$$가 두번 이상 등장하기 때문에 제곱 인수가 없는 수가 아닙니다. 즉 square-free가 아니라는 뜻이죠.

$$m$$의 소인수분해 결과가 $$m=p_{0}\cdot p_{1} \cdot \cdot \cdot p_{r}$$이라고 해봅시다. $$m$$의 소인수, $$p_{i}$$에 대해서, 우리는 $$\binom{n}{k}mod$$ $$p_{i}$$를 우리가 이때까지 알아봤던 방식을 통해서 구해낼 수 있습니다. 여기서 계산해 낸 결과들을 각각 $$a_{0},a_{1},...,a_{r}$$이라고 합시다. [중국인 나머지 정리(Chinese Remainder Theorem, 줄여서 CRT)](https://j1w2k3.tistory.com/1340)라는 것을 이용하면, 우리는

$$x≡a_{0}\mod p_{0}$$, $$x≡a_{1}\mod p_{1} \cdot\cdot\cdot$$ , $$x≡a_{r} \mod p_{r}$$

합동식을 만족시키는 $$x$$를 찾을 수 있습니다.

<i class="fas fa-info-circle"></i> <strong>정보</strong><br>
중국인 나머지 정리를 링크 걸어놓은것을 보면 알 수 있겠지만, 중국인 나머지의 결론은 아래와 같습니다.<br><br>
$$n_1,n_2,\cdot\cdot\cdot,n_r$$을 $$i\neq j$$에 대해, $$n_i,n_j$$가 서로소인 양의 정수 일 때, 연립합동식
$$x≡a_{1}(\mod n_1)$$
$$x≡a_{2}(\mod n_2)$$
$$\cdot\cdot\cdot$$
$$x≡a_{r}(\mod n_r)$$은
$$\mod n_1 n_2 \cdot\cdot\cdot n_r$$ 에 대해서 유일한 공통 해를 가지고,<br>
해를 구하는 법은
$$n = n_{1}n_{2}\cdot\cdot\cdot n_{r}$$, $$N_{i} = \frac{n}{n_{i}}$$ 라고 정의하면
$$x = a_{1}N_{1}x_{1} + a_{2}N_{2}x_{2} + \cdot\cdot\cdot + a_{r}N_{r}x_{r}$$
로 나타낼 수 있다.
이때 $$x_{i}$$는, 각 선형 합동식의 특수해이고, 이는 확장 유클리드 알고리즘으로 구할 수 있다.
{:.info}

$$m_{i} = \frac{p_{0}p_{1}\cdot\cdot\cdot p_{r}}{p_{i}}$$ 라고 합시다. 우리는 $$m_{i}$$와 $$p_i$$가 서로소 라는 것을 당연히 알고 있습니다. 따라서, $$gcd(m_i,p_i) = 1$$이고, 이는 $$1=s_i \cdot m_i + t_i \cdot p_i$$라는 뜻 입니다. [확장 유클리드 알고리즘](https://joonas.tistory.com/25)을 이용하면, 우리는 위 등식을 만족하는 $$s_i$$와 $$t_i$$를 찾을 수 있습니다.

최종적으로 우리는 이 모든것을 조합해서
$$x=\sum_{i=0}^{r}(a_i\cdot m_i\cdot s_i)(mod$$ $$\prod_{i=0}^r m_i)$$라는 $$x$$의 값을 구하는 식을 얻을 수 있습니다.

여기에서 중국인 나머지 정리를 쓸 수 있는 이유는, 모듈러 연산을 하는 $$m$$의 소인수들, $$p_1,p_2,\cdot\cdot\cdot, p_r$$이 서로소이기 때문입니다. 아무튼 위의 아이디어를 파이썬 코드로 옮겨보면 아래와 같습니다.

```python
#!/usr/bin/env python3

"""
뤼카의 정리와 페르마의 소정리를 이용해서, nCk mod m을 계산한다.
m의 소인수 분해 결과는 square-free 이다.
중국인 나머지 정리를 이용하여, 소인수들의 합동식을 종합해서 결과를 낸다.
"""

# 모듈러 지수승(Modular Exponentiation) : b^e % mod
# 파이썬의 내장 pow(b,e,mod)가 이 함수보다는 빠르다는것을 참조할것.
def mod_exp(b,e,mod):
    r = 1
    while e > 0:
        if (e&1) == 1:
            r = (r*b)%mod
        b = (b*b)%mod
        e >>= 1

    return r


# n!에서 p의 차수를 구함(n! = p^a * ... 일때, 가능한 최대의 a를 구함)
def fact_exp(n,p):
    e = 0
    u = p
    t = n
    while u <= t:
        e += t//u
        u *= p

    return e

# 주어진 수 n을 b진법으로 변환함
# 가장 큰 자리수(Most significant Digit)의 숫자가 배열의 맨 오른쪽에 있음
def get_base_digits(n,b):
    d = []
    while n > 0:
        d.append(n % b)
        n  = n // b

    return d

# 확장된 유클리드 알고리즘
# ax + by = gcd(a,b) 일때, x,y를 계산한다.
# 여기서는 a,b가 서로소기 떄문에, gcd(a,b) = 1 이다.
def egcd(a, b):
    x,y, u,v = 0,1, 1,0
    while a != 0:
        q, r = b//a, b%a
        m, n = x-u*q, y-v*q
        b,a, x,y, u,v = a,r, u,v, m,n
    gcd = b
    return (x, y)

# 중국인 나머지 정리
# 주어진 합동식으로 하나의 답을 출력한다
def crt(congruences):
    #원래의 mod m 을 계산한다
    m = 1
    for congruence in congruences:
        m *= congruence[1]

    # 합동식들을 종합한다
    result = 0
    for congruence in congruences:
        s, t = egcd(m//congruence[1],congruence[1])
        result += (congruence[0]*s*m)//congruence[1]

    return result%m

# 페르마의 소정리를 이용해서 nCk mod p을 구함
# p의 지수승으로 분자, 분모를 약분
# 주의사항 : p가 반드시 소수여야함
def fermat_binom_advanced(n,k,p):
    # 차수가 말이 되는지를 판별
    num_degree = fact_exp(n,p) - fact_exp(n-k,p)
    den_degree = fact_exp(k,p)
    if num_degree > den_degree:
        return 0

    if k > n:
        return 0

    # 분자를 계산하고, p들을 약분해낸다.
    num = 1
    for i in range(n,n-k,-1):
        cur = i
        while cur%p == 0:
            cur //= p
        num = (num*cur)%p

    # 분모를 계산하고 p들을 약분해낸다
    denom = 1
    for i in range(1,k+1):
        cur = i
        while cur%p == 0:
            cur //= p
        denom = (denom*cur)%p

    # 분자 * 분모^(p-2) (mod p)
    return (num * mod_exp(denom,p-2,p))%p

# 뤼카의 정리를 이용하여 문제를 작은 부분 문제로 분할한다
# p 는 소수여야 한다
def lucas_binom(n,k,p):
    # get n and k in base p representation
    np = get_base_digits(n,p)
    kp = get_base_digits(k,p)

    # calculate (nCk) = (n0 choose k0)*(n1 choose k1) ... (ni choose ki) (mod p)
    binom = 1
    for i in range(len(np)-1,-1,-1):
        ni = np[i]
        ki = 0
        if i < len(kp):
            ki = kp[i]

        binom = (binom * fermat_binom_advanced(ni,ki,p)) % p

    return binom

# 주어진 소인수 m 에 대해서 nCk를 계산한다
# 소인수들은 m 에서 1의 차수를 가져야 한다
def binom(n,k,mod_facts):
    # 모든 소인수에 대해서 합동식을 작성한다
    congruences = []
    for p in mod_facts:
        # 합동식 리스트에 (binom,p) 를 추가한다
        congruences.append((lucas_binom(n,k,p),p))

    # 중국인 나머지 정리를 통해 합동식으로 하나의 답을 추린다
    return crt(congruences)

if __name__ == '__main__':
    mod_facts = [3,5,7,11] # prime factors of m = 1155

    # (8100 choose 4000) mod 1155
    print(binom(8100,4000,mod_facts)) # should be 924
```

<i class="fas fa-info-circle"></i> <strong>정보</strong><br>
큰 m에 대해서 소인수 분해를 하는것은 non-trivial 하고, 효율적인 알고리즘이 아직 발견되지 않았습니다. 하지만, m이 적당한 크기일때(일반적인 문제 상황에서는 대부분 그러합니다) m을 소인수 분해 하는것은 그리 어렵지 않습니다. 당신의 알고리즘을 직접 작성해도 되고, 온라인 상에 있는 계산기 들을 사용해서 소인수 분해한 값을 찾아야 합니다.
꼭 기억해야 할 것은 우리의 이번 접근에, 소인수들이 square-free 한 상황에서만 가능하다는 것입니다.
{:.info}

## m에 1보다 더 큰 차수를 가진 소인수가 있을 경우

마지막으로, 가능한 마지막 경우에 대해서 살펴보도록 하겠습니다. 소인수들의 차수가 1보다 더 클 경우 입니다.
그런 상황에서는 우리는 Andrew Granville의 [뤼카의 정리의 일반화(영어)](https://dms.umontreal.ca/~andrew/Binomial/genlucas.html)를 사용할 수 있습니다. 이 정리는 m의 소인수 분해 결과에 소수배의 차수를 가지고 있을 때, 계산을 할 수 있게 합니다. 이 정리를 적용하고, 우리는 중국인 나머지 정리를 이용해서, 부분문제들의 해답을 통해서 원래의 문제의 해답을 찾아낼 수 있습니다. 소수배의 차수를 계산할 수 있으면, 우리는 어떤 mod m 에 관해서도 계산을 할 수 있다는 것입니다. 원래 포스팅에도 구현체가 올라와 있지 않아, 언젠가 시간과 여유와 실력이 되는 날이 온다면 구현체를 올려보도록 하겠습니다.

# 4. 요 약

큰 수의 이항계수 $$\binom{n}{k} mod$$ $$m$$을 구하기 위해서는 아래의 수순을 밟아야 합니다.

-   $$m$$을 소인수 분해 하여 $$m = p_{0}^{e_{0}},...,p_{r}^{e_{r}}$$ 의 결과물을 얻어냅니다
-   (일반화된) 뤼카의 정리를 이용하여 각 $$\binom{n}{k}mod p_{i}^{e_{i}}$$를 부분문제들로 쪼갭니다
-   $$\binom{n}{k}mod p_{i}^{e_{i}}$$의 부분문제들을 페르마의 소정리나, 파스칼의 삼각형으로 풀어 냅니다
-   중국인 나머지의 정리를 이용해서 원래 문제의 답을 알아냅니다.

<i class="fas fa-info-circle"></i> <strong>정보</strong><br>
한가지 유념할 사항은, $$p_{i} > n$$ 이라면 부분문제로 쪼갤것이 없고, 그냥 그 문제를 페르마의 소정리나 파스칼의 삼각형을 이용해서 풀어내면 된다는 것입니다.
{:.info}

끝입니다. 이제 이것들을 이용해서 매우 큰 수의 이항계수 $$\mod m$$ 을 구할 수 있을 것입니다.

# 참고한 글들

모듈러 지수승(Modular Exponentiation) C++ 코드 : [링크](https://www.geeksforgeeks.org/modular-exponentiation-power-in-modular-arithmetic/)
<br>중국인의 나머지 정리 나무위키 문서 : [링크](https://namu.wiki/w/%EC%A4%91%EA%B5%AD%EC%9D%B8%EC%9D%98%20%EB%82%98%EB%A8%B8%EC%A7%80%20%EC%A0%95%EB%A6%AC)
<br>그 외 본 글에 링크한 모든 글들

읽어주셔서 감사합니다.
