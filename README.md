# LogicPuzzles

## Sum and product in Hybrid Epistemic Logic

### Language
$\varphi::=x\ |\ p\ |\ \neg\phi\ |\ \phi\land\varphi\ |\ \Diamond_A\varphi\ |\ \Diamond_B\varphi\ |\ \downarrow_x\varphi$

### Frame

$\mathcal{F} = (W,R_A,R_B)$ where

$W=$ { $(a,b)\ |\ a,b\in\mathbb{N},1<a<b, a+b\le 100$ }

$(a,b)R_A(c,d)$ iff $a+b = c+d$

$(a,b)R_B(c,d)$ iff $ab = cd$

### Frame Check
Find a couple $(a,b)$ where $\mathcal{F},(a,b)\vDash \varphi_1\land\psi\land\chi$ where

$\varphi_1:=(\downarrow_x\Diamond_A\neg x)\land (\Box_A\downarrow_x\Diamond_B\neg x)$

$\varphi_2:=(\downarrow_x\Diamond_A\neg x)\land (\Diamond_A\downarrow_x\Box_B x)$

$\psi:=\downarrow_x\Box_B(\neg x\rightarrow\varphi_2)$

$\chi:=\downarrow_x\Box_A(\neg x\rightarrow\neg \psi)$

其中 $\varphi_1$ 为真使 $A$ 能够断言 $B$ 不知道答案。 $\psi$ 为真使 $B$ 能够在 $A$ 断言后知道答案。 $\chi$ 为真使 $A$ 能够在 $B$ 知道答案后知道答案。

### Code Demo
```python
# 100以内质数列表
primes = {
    2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37,
    41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97
}

# 答案
result = []

# 乘积相同的二元组，即非自反的B关系
def m_pair(a, b):
    if a < 2 or b < 2:
        return False  # 不考虑小于2的因数

    product = a * b
    factor_pairs = set()

    # 枚举所有可能的 x × y = product 且 x <= y 且 x, y ≥ 2
    for x in range(2, int(product ** 0.5) + 1):
        if product % x == 0:
            y = product // x
            if y >= 2:
                factor_pairs.add(tuple(sorted((x, y))))
    # 去除自反的B关系
    factor_pairs.discard(tuple(sorted((a, b))))
    return factor_pairs

# 判断A是否能够知道B不知道，返回true表示A能够知道B不知道
# 即所有的A关系到达的点上（不包括自己），是否都有非自反的B关系
def A_know_B_dont_know(a,b):
    s = a + b
    # 枚举所有 x + y = s 且 2 <= x <= y <= 99
    valid = True
    for x in range(2, s // 2 + 1):
        y = s - x
        if x > y or y > 99:
            continue
        if (x, y) == (a, b):
            continue
        if len(m_pair(x, y)) == 0:
            valid = False
            break

    return valid

# 判断B是否能够在A确信B不知道的宣告后，B能够知道答案，返回true表示能够
# 即B能通达的非自反点上，都要A_know_B_dont_know是false，这样B就能确定唯一的一个A_know_B_dont_know为true的二元组，确定为答案
def B_know_the_answer(a, b):
    
    valid = True
    for mp in m_pair(a, b):
        x = mp[0]
        if x >= 100:
            continue
        y = mp[1]
        if y >= 100:
            continue
        if A_know_B_dont_know(x, y) == True:
            valid = False
            break
    return valid

# 主循环，遍历模型中所有可能的二元组(a, b)
for a in range(2, 100):
    for b in range(a, 100):
        s = a + b
        if s > 100:
            continue
        
        # 如果是B自反点，B立刻知道答案
        if len(m_pair(a, b)) == 0:
            continue
        
        # A知道B不知道，并且A告诉B后B知道答案
        if A_know_B_dont_know(a,b) and B_know_the_answer(a,b):

            # 最后要保证，B告诉A知道答案后，A也知道答案
            # 即，在A能通达的除了(a,b)以外的点上，B都不能够在A告诉B之后知道答案，只能在(a,b)上B_know_the_answer为真
            valid = True
            for x in range(2, s // 2 + 1):
                y = s - x
                if x > y or y > 99:
                    continue
                if (x, y) == (a, b):
                    continue
                if B_know_the_answer(x, y) == True:
                    valid = False
            if valid:
                result.append((a, b))

print("满足条件的二元组有：")
for pair in result:
    print(pair)
```
