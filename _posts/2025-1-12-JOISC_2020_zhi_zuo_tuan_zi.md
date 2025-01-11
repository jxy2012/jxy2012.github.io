---
layout: post
title: "恩情课文之JOISC2020制作团子卡我一天的恩情还不完"
date:   2025-1-12
tags: [Medium]
comments: true
author: jxy2012
---

[题目链接](https://uoj.ac/problem/511)

这题浪费了我一天。。。

显然是一个一般图的最大独立集问题。

就是把几串共用了相同位置的丸子连边，表示它们不能同时存在，然后就做完了。

***But***，这是个 ***NP-Hard*** 问题！！！

## Part 1 模拟退火
没事，既然是 NP-Hard 问题，数据范围只有 $500$，而且还是提交答案题，就用我们无敌的模拟退火，感觉 $2h$ 就能过掉（记住这句话）。

于是，开始狂打代码。

打着打着就已经过去了 $3h$。

没事，再过 $0.5h$ 就能过掉。

展示一下我 $8KB$ 的代码：

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
const ll mod = 1e9 + 7;
const int N = 505;
const int INF = 0x3f3f3f3f;
mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());
int random(int l, int r) {
    return uniform_int_distribution<int>(l, r)(rng);
}
const int N = 1005;
char s[N];
vector<int> V[N][N], adj[4 * N * N];
int col[N][N], ans[N][N], n, m, tot;
#define ls (u << 1)
#define rs (u << 1 | 1)
int sum[4 * N * N << 2];
void pushup(int u) {
    sum[u] = sum[ls] + sum[rs];
}
void build(int u, int l, int r) {
    if (l == r) {
        sum[u] = 1;
        return;
    }
    int mid = l + r >> 1;
    build(ls, l, mid);
    build(rs, mid + 1, r);
    pushup(u);
}
void update(int u, int l, int r, int pos, int dlt) {
    if (l == r) {
        sum[u] += dlt;
        return;
    }
    int mid = l + r >> 1;
    if (pos <= mid)
        update(ls, l, mid, pos, dlt);
    else
        update(rs, mid + 1, r, pos, dlt);
    pushup(u);
}
int query(int u, int l, int r, int rnk) {
    if (l == 1 && r == tot) assert(rnk <= r - l + 1);
    if (l == r) {
        assert(rnk == 1);
        return l;
    }
    int mid = l + r >> 1;
    if (sum[ls] >= rnk)
        return query(ls, l, mid, rnk);
    else
        return query(rs, mid + 1, r, rnk - sum[ls]);
}

int dot, vis[4 * N * N];
int ans_dot, ans_vis[4 * N * N];

void updans() {
    ans_dot = dot;
    for (int i = 1; i <= tot; i++) ans_vis[i] = vis[i];
}
void SA() {
    for (int i = 1; i <= tot; i++) vis[i] = 0;
    double T = 1000, alpha = 0.99994;
    build(1, 1, tot);
    dot = 0;
    while (T >= 1e-6) {
        for (int times = 1; times <= 5; times++) {
            //      if (dot == tot) {
            //        puts("wtf");
            //      }
            int rnk = random(1, tot - dot);
            assert(dot >= 0);
            //      printf("tot = %d, dot = %d, rnk = %d\n", tot, dot, rnk);
            int u = query(1, 1, tot, rnk);
            assert(!vis[u]);
            //      printf("%d\n", u);
            int adds = 1;
            for (auto v : adj[u])
                if (vis[v]) {
                    adds--;
                }
            //      printf("adds = %d\n", adds);
            int ok = 0;
            if (adds > 0 || exp(10.0 * adds / T) > rng() / 4294967296.0) {
                vis[u] = 1, update(1, 1, tot, u, -1);
                for (auto v : adj[u])
                    if (vis[v]) {
                        vis[v] = 0, update(1, 1, tot, v, 1);
                    }
                dot += adds;
                if (dot > ans_dot) updans();
            }
        }
        T *= alpha;
    }
}

int main() {
    freopen("01.in", "r", stdin);
    freopen("01.out", "w", stdout);
    scanf("%d%d", &n, &m);
    n += 2, m += 2;
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            col[i][j] = ans[i][j] = 2;
        }
    }
    for (int i = 2; i < n; i++) {
        scanf("%s", s + 2);
        for (int j = 2; j < m; j++) {
            if (s[j] == 'P') col[i][j] = ans[i][j] = 0;
            if (s[j] == 'G') col[i][j] = ans[i][j] = 1;
        }
    }
    for (int i = 2; i < n; i++) {
        for (int j = 2; j < m; j++) {
            if (col[i][j] == 2) {
                if ((col[i - 1][j] ^ col[i + 1][j]) == 1) {
                    tot++;
                    V[i - 1][j].pb(tot);
                    V[i + 1][j].pb(tot);
                }
                if ((col[i][j - 1] ^ col[i][j + 1]) == 1) {
                    tot++;
                    V[i][j - 1].pb(tot);
                    V[i][j + 1].pb(tot);
                }
                if ((col[i - 1][j - 1] ^ col[i + 1][j + 1]) == 1) {
                    tot++;
                    V[i - 1][j - 1].pb(tot);
                    V[i + 1][j + 1].pb(tot);
                }
                if ((col[i - 1][j + 1] ^ col[i + 1][j - 1]) == 1) {
                    tot++;
                    V[i - 1][j + 1].pb(tot);
                    V[i + 1][j - 1].pb(tot);
                }
            }
        }
    }
    cerr << "tot = " << tot << '\n';
    for (int i = 2; i < n; i++) {
        for (int j = 2; j < m; j++) {
            int sz = V[i][j].size();
            for (int x = 0; x < sz; x++) {
                for (int y = 0; y < x; y++) {
                    int u = V[i][j][x], v = V[i][j][y];
                    adj[u].pb(v), adj[v].pb(u);
                }
            }
        }
    }
    SA();
    cerr << "ans_dot = " << ans_dot << '\n';
    tot = 0;
    for (int i = 2; i < n; i++) {
        for (int j = 2; j < m; j++) {
            if (col[i][j] == 2) {
                if ((col[i - 1][j] ^ col[i + 1][j]) == 1) {
                    tot++;
                    if (ans_vis[tot]) ans[i][j] = 3;
                }
                if ((col[i][j - 1] ^ col[i][j + 1]) == 1) {
                    tot++;
                    if (ans_vis[tot]) ans[i][j] = 4;
                }
                if ((col[i - 1][j - 1] ^ col[i + 1][j + 1]) == 1) {
                    tot++;
                    if (ans_vis[tot]) ans[i][j] = 5;
                }
                if ((col[i - 1][j + 1] ^ col[i + 1][j - 1]) == 1) {
                    tot++;
                    if (ans_vis[tot]) ans[i][j] = 6;
                }
            }
        }
    }
    int Points = 0;
    for (int i = 2; i < n; i++) {
        for (int j = 2; j < m; j++) {
            switch (ans[i][j]) {
                case 0:
                    putchar('P');
                    break;
                case 1:
                    putchar('G');
                    break;
                case 2:
                    putchar('W');
                    break;
                case 3:
                    putchar('|');
                    Points++;
                    break;
                case 4:
                    putchar('-');
                    Points++;
                    break;
                case 5:
                    putchar('\\');
                    Points++;
                    break;
                case 6:
                    putchar('/');
                    Points++;
                    break;
            }
        }
        putchar('\n');
    }
    cerr << "Points = " << Points << '\n';
    return 0;
}

```
结果：跑了 $1h$ 还没跑完，喜提 $59$，***But***已经过去了一个早上。。。

好不容易停了课居然为了这种题浪费我时间。。。

## Part 2 爬山
既然模拟退火这么垃圾，就用爬山算法！！！

打着打着又过去了 $3.5h$。

展示一下我第二份 $8KB$ 的代码：

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
const ll mod = 1e9 + 7;
const int N = 505;
const int INF = 0x3f3f3f3f;
mt19937 rnd(time(NULL) ^ clock());
ll rad(int x, int y) {
    return rnd() % (y - x + 1) + x;
}
int vis[N * N], anss[N * N], pre[N * N];
basic_string<int> choice[N * N];
basic_string<int> pos;
map<int, int> gg;
struct Node {
    int x, y, z, id;
    Node() {}
    Node(int a, int b, int c, int f) {
        x = a;
        y = b;
        z = c;
        id = f;
    }
} a[N];
int id(int x, int y) {
    return x * N + y;
}
int n, m, cnt;
char s[N][N];
void check(int x, int y, int dx, int dy) {
    if (x + dx + dx > n || x + dx + dx <= 0 || y + dy + dy > m || y + dy + dy <= 0) return;
    if (s[x][y] == 'W' || s[x + dx + dx][y + dy + dy] == 'W') return;
    if (s[x + dx][y + dy] != 'W') return;
    if (s[x][y] == s[x + dx + dx][y + dy + dy]) return;
    cnt++;
    a[cnt] = nod(id(x, y), id(x + dx, y + dy), id(x + dx + dx, y + dy + dy), cnt);
    choice[id(x, y)] += cnt;
    choice[id(x + dx, y + dy)] += cnt;
    choice[id(x + dx + dx, y + dy + dy)] += cnt;
    return;
}
void init() {
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            check(i, j, 1, 0);
            check(i, j, 0, 1);
            check(i, j, 1, -1);
            check(i, j, 1, 1);
        }
    }
}
int ans = 0, zd = 0;
void fk(int ID) {
    if (gg[ID] != 0) return;
    gg[ID] = 1;
    pos += ID;
    pre[ID] = vis[ID];
}
ll yanse, col[N * N];
int head, tail, Q[N * N];
void del(int ID) {
    if (ID == 0) return;
    fk(a[ID].x);
    fk(a[ID].y);
    fk(a[ID].z);
    ans--;
    vis[a[ID].x] = vis[a[ID].y] = vis[a[ID].z] = 0;
    Q[++tail] = a[ID].x;
    Q[++tail] = a[ID].y;
    Q[++tail] = a[ID].z;
    return;
}
void get(int tt, int yanse) {
    if (vis[a[tt].x]) del(vis[a[tt].x]);
    if (vis[a[tt].y]) del(vis[a[tt].y]);
    if (vis[a[tt].z]) del(vis[a[tt].z]);
    ans++;
    fk(a[tt].x);
    fk(a[tt].y);
    fk(a[tt].z);
    vis[a[tt].x] = vis[a[tt].y] = vis[a[tt].z] = tt;
    col[a[tt].x] = col[a[tt].y] = col[a[tt].z] = yanse;
    return;
}
void gao(int bh, ll yanse) {
    head = tail = 1;
    Q[head] = bh;
    while (head <= tail) {
        int now = Q[head++], siz = choice[now].size();
        if (col[now] == yanse) continue;
        if (siz == 0) continue;
        int tim = siz;
        int tt = -1;
        while (tim--) {
            tt = rad(0, siz - 1);
            tt = choice[now][tt];
            if (vis[a[tt].x] == tt) {
                tt = -1;
                continue;
            }
            int cnt = 0;
            cnt = (col[a[tt].x] == yanse) + (col[a[tt].y] == yanse) + (col[a[tt].z] == yanse);
            if (cnt >= 1) {
                tt = -1;
                continue;
            }
        }
        if (tt == -1) continue;
        get(tt, yanse);
    }
}
void work(int lim) {
    for (int x = 1; x <= n; x++) {
        for (int y = 1; y <= m; y++) {
            anss[id(x, y)] = vis[id(x, y)];
        }
    }
    zd = ans;
    for (int i = 1;; i++) {
        int x = rad(1, n), y = rad(1, m);
        int bh = id(x, y);
        int ID = vis[bh];
        pos.clear();
        gg.clear();
        if (vis[bh] == 0) {
            yanse++;
            gao(bh, yanse);
        }
        if (ans < zd) {
            for (int tt : pos) vis[tt] = pre[tt];
            ans = zd;
        }
        if (ans > zd) {
            for (int x = 1; x <= n; x++)
                for (int y = 1; y <= m; y++)
                    anss[id(x, y)] = vis[id(x, y)];
        }
        zd = max(zd, ans);
        if (i % 1000 == 0) {
            cout << zd << endl;
        }
        if (zd >= lim) break;
    }
}
void out() {
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            vis[id(i, j)] = anss[id(i, j)];
        }
    }
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++) {
            int ID = id(i, j), tt = vis[ID];
            if (tt != 0 && ID == a[tt].y) {
                int xx = i, yy = j;  // 这是你的坐标
                int px = a[tt].x / N, py = a[tt].x % N;
                int tx = a[tt].z / N, ty = a[tt].z % N;
                // 上中下
                if (px + 1 == xx && yy == py) s[i][j] = '|';
                // 左中右
                if (py + 1 == yy && xx == px) s[i][j] = '-';
                // ↘
                if (px + 1 == xx && py + 1 == yy) s[i][j] = '\\';
                // ↙
                if (px + 1 == xx && py - 1 == yy) s[i][j] = '/';
            }
        }
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            cout << s[i][j];
        }
        cout << endl;
    }
}

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        scanf("%s", s[i] + 1);
    }
    init();
    work(48620);
    cout << zd << "\n";
    freopen("05.out", "w", stdout);
    out();
    return 0;
}

```

跑了 $10$ 分钟，过了！！！

我是冠军！！！
