1.leetcode第605题

![](vivo1.png)

```c++
#include<bits/stdc++.h>
using namespace std;
/*
5
1 0 0 0 0

2
 */
int main()
{
	int N;
	cin >> N;
	vector<int> wh(N, 0);
	for (int i = 0; i < N; ++i)
		cin >> wh[i];
	int ans = 0;
	for(int i=0;i<wh.size();++i)
		if (wh[i] == 0) {
			if ((i == 0||wh[i - 1] == 0) && (i == wh.size() - 1||wh[i + 1] == 0)) {
                //检查顺序改变可能会越界，因为第一个元素默认前面为0，所以(i == 0||wh[i - 1] == 0)
                //末尾也是同理，贪心解决；
				wh[i] = 1;
				ans++;
			}
		}
	cout << ans << endl;
	return 0;
}
```



2. leetcode第887题

   ![](vivo2.png)
   
   实在太难理解了。其实这一题不能同时在多个楼层扔鸡蛋，只能一次在一层楼扔鸡蛋，所以我们**假设dp\[i\]\[j\]表示我们走i步，有j个鸡蛋的时候，能检查dp\[i\]\[j\]层楼**；那么转移方程就是dp\[i\]\[j\]=1+dp\[i-1\]\[j-1\]+dp\[i-1\]\[j\]，如果我们能够有i次机会扔鸡蛋，那么我们**能检测的楼层=我们扔鸡蛋那一楼+鸡蛋碎了剩余i-1步能检测的楼层+鸡蛋没碎剩余i-1步能检测的楼层**；当然，初始值为dp\[1\]\[0\]=0，没鸡蛋检测不了；dp\[1\]\[j\]=1，无限鸡蛋，一次也只能检测一层楼；由于检测次数i肯定是从1增大到无穷的，所以我们不枚举i了，初始化的dp就是i=1的情况；因为dp\[i\]\[j\]只与dp\[i-1\]\[j\]和dp\[i-1\]\[j-1\]有关，也就是我们下面的dp\[i\]和dp\[i-1\]，所以我们更新的时候从末尾开始更新；

```c++
#include <bits/stdc++.h>
/*
1 2

2
*/
using namespace std;
int main() {
	int K, N, step = 1;
	cin >> K >> N;
	vector<int> dp(K + 1, 1);
	dp[0] = 0;
	while (dp[K] < N) {
		for (int i = K; i > 0; --i)
			dp[i] = dp[i] + dp[i - 1] + 1;
		step++;
	}
	cout << step << endl;
	return 0;
}
```

3. leetcode第23题

   ![](vivo3.png)

```c++
#include <bits/stdc++.h>
/*
4
2 4 8
3 4 6
5 8 9 11
1 6 7 10 12

1 2 3 4 4 5 6 6 7 8 8 9 10 11 12
*/
using namespace std;
struct ListNode {
	int val_;
	ListNode* next;
	ListNode(int val) {
		val_ = val;
		next = NULL;
	}
};
struct cmp {
	bool operator()(ListNode *a,ListNode *b) {
		return a->val_ > b->val_;
	}
};
void solveit(ListNode *p, string &str) {
	ListNode *q = p;
	int start = 0, end = 0;
	while (end<str.size()){
		int temp;
		while (end < str.size()&&str[end] != ' ')
			end++;
		if (end == str.size())
			temp = stoi(str.substr(start, end - start + 1));
		else
			temp = stoi(str.substr(start, end - start));
		ListNode *templ = new ListNode(temp);
		q->next = templ;
		q = q->next;
		start = ++end;
	}
}
int main() {
	int n = 0;
	cin >> n;
	vector<ListNode*> wh(n);
	cin.get();
	string temp;
	priority_queue < ListNode*, vector<ListNode*>,cmp> pq;
	for (int i = 0; i < n; i++) {
		ListNode *p = new ListNode(-1),*q=p;
		wh[i] = p;
		getline(cin, temp);
		solveit(p, temp);
		pq.push(p->next);
	}
	ListNode *newhead = new ListNode(-1),*work=newhead;
	while (!pq.empty()) {
		ListNode *pqtop = pq.top();
		work->next = pqtop;
		work = work->next;
		pq.pop();
		pqtop = pqtop->next;
		if (pqtop)
			pq.push(pqtop);
	}
	work = newhead->next;
	while (work->next) {
		cout << work->val_ << " ";
		work = work->next;
	}
	cout << work->val_ << endl;
	return 0;
}
```

