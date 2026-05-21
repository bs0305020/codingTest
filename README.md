```code
import sys

n = int(input())
min_component = list(map(int, input().split()))

gradients = []
for i in range(n):
    gradients.append(list(map(int, input().split())))

min_cost = float('inf')
max_total_nutrients = -1
best_indices = []
selected = []

def dfs(idx, p, f, s, v, current_cost):
    global min_cost, max_total_nutrients, best_indices

    if current_cost > min_cost:
        return

    if idx == n:
        if p >= min_component[0] and f >= min_component[1] and s >= min_component[2] and v >= min_component[3]:
            total_nutrients = p + f + s + v
            if (current_cost < min_cost or
                    (current_cost == min_cost and total_nutrients > max_total_nutrients) or
                    (current_cost == min_cost and total_nutrients == max_total_nutrients and (
                            not best_indices or selected < best_indices))):
                min_cost = current_cost
                max_total_nutrients = total_nutrients
                best_indices = selected[:]
        return

    selected.append(idx + 1)
    dfs(idx + 1,
        p + gradients[idx][0],
        f + gradients[idx][1],
        s + gradients[idx][2],
        v + gradients[idx][3],
        current_cost + gradients[idx][4])

    selected.pop()
    dfs(idx + 1, p, f, s, v, current_cost)

dfs(0, 0, 0, 0, 0, 0)

if not best_indices:
    print(0)
else:
    print(*(best_indices))
```
