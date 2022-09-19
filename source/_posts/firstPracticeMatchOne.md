---
title: 第一次练习比赛一
excerpt: SAU第一次练习比赛一
categories:
- 算法
tags:
- 算法
---

## 前言
此篇是我自己对SAU第一次练习比赛一的题的理解，不代表是正确的题解

## P1046 陶陶摘苹果
**题目描述**
陶陶家的院子里有一棵苹果树，每到秋天树上就会结出 10 个苹果。苹果成熟的时候，陶陶就会跑去摘苹果。陶陶有个 3030 厘米高的板凳，当她不能直接用手摘到苹果的时候，就会踩到板凳上再试试。
现在已知 10 个苹果到地面的高度，以及陶陶把手伸直的时候能够达到的最大高度，请帮陶陶算一下她能够摘到的苹果的数目。假设她碰到苹果，苹果就会掉下来。

**输入格式**
输入包括两行数据。第一行包含 10 个 100 到 200 之间（包括 100 和 200 ）的整数（以厘米为单位）分别表示 10 个苹果到地面的高度，两个相邻的整数之间用一个空格隔开。第二行只包括一个 100 到 120 之间（包含 100 和 120 ）的整数（以厘米为单位），表示陶陶把手伸直的时候能够达到的最大高度。

**输出格式**
输出包括一行，这一行只包含一个整数，表示陶陶能够摘到的苹果的数目。

**思路**
要是先输入身高，再输入十个数字，我都不想开数组，用两个变量解决，但是并不是。
先for输入数字，再输入身高，把身高+30，for循环判断身高>=树高的ans++就行。

**code**
```cpp
int t[15];
int main(){
    int ans=0;
    int x;
    for(int i=0;i<10;i++) cin>>t[i];
    cin>>x;
    x+=30;
    for(int i=0;i<10;i++){
        if(x>=t[i]) ans++;
    }
    cout<<ans<<endl;
    return 0;
}
```

## P1304 哥德巴赫猜想
**题目描述**
输入一个偶数 N(N<=10000)，验证4~N所有偶数是否符合哥德巴赫猜想：任一大于 2 的偶数都可写成两个质数之和。如果一个数不止一种分法，则输出第一个加数相比其他分法最小的方案。例如 10，10=3+7=5+5，则 10=5+5 是错误答案。

**输入格式**
第一行N

**输出格式**
4=2+2 6=3+3 …… N=x+y

**思路**
直接打个素数表，然后打印4=2+2，从6开始到n，分解成3+x，然后每个偶数判断3和n-3是不是素数，是打印，不是就+2。

**code**
```cpp
#define MAXN 100005
#define MAXL 1299710
int prime[MAXN];
int check[MAXL];

void pri(){
    int tot = 0;
    memset(check, 0, sizeof(check));
    for (int i = 2; i < MAXL; ++i)
    {
        if (!check[i]) prime[tot++] = i;
        for (int j = 0; j < tot; ++j)
        {
            if (i * prime[j] > MAXL) break;
            check[i*prime[j]] = 1;
            if (i % prime[j] == 0) break;
        }
    }
}

int main(){
    int n;
    pri();
    scanf("%d",&n);
    printf("4=2+2\n");
    for(int i=6;i<=n;i+=2){
        printf("%d=",i);
        for(int j=3;;j+=2){
            if(check[j]==0&&check[i-j]==0){
                printf("%d+%d\n",j,i-j);
                break;
            }
        }
    }
    return 0;
}
```

## P1739 表达式括号匹配
**题目描述**
假设一个表达式有英文字母（小写）、运算符（+，—，*，/）和左右小（圆）括号构成，以“@”作为表达式的结束符。请编写一个程序检查表达式中的左右圆括号是否匹配，若匹配，则返回“YES”；否则返回“NO”。表达式长度小于255，左圆括号少于20个。

**输入格式**
一行：表达式

**输出格式**
一行：“YES” 或“NO”

**思路**
要清楚什么是错，每读取一个字符，如果是(，ans++；如果是)，ans--；如果ans<0，直接结束读取字符，输出NO，如果读到@，结束循环，判断ans的数值，如果为0，输出YES，否则NO。

**code**
```cpp
int main(){
    char c,s[300];
    int ans=0;
    while(cin>>c){
        if(c=='@') break;
        if(c==')'){
            ans--;
            if(ans<0){
                printf("NO\n");
                return 0;
            }
        }
        if(c=='(') ans++;
    }
    if(ans==0) printf("YES\n");
    else printf("NO\n");
}
```

## P1271 【深基9.例1】选举学生会
**题目描述**
学校正在选举学生会成员，有n(n≤999)名候选人，每名候选人编号分别从 1 到 n，现在收集到了m(m<=2000000)张选票，每张选票都写了一个候选人编号。现在想把这些堆积如山的选票按照投票数字从小到大排序。输入 n 和 m 以及 m 个选票上的数字，求出排序后的选票编号。

**输入格式**
无

**输出格式**
无

**思路**
一看是个排序，再看范围，1000，直接桶排

**code**
```cpp
int t[1005];
int main(){
    memset(t, 0, sizeof(t));
    int n,m,x;
    cin>>n>>m;
    for(int i=0;i<m;i++){
        cin>>x;
        t[x]++;
    }
    for(int i=1;i<=n;i++){
        while(t[i]--) cout<<i<<" ";
    }
}
```

## P1601 A+B Problem（高精）
**题目描述**
高精度加法，相当于a+b problem，不用考虑负数.

**输入格式**
分两行输入。a,b <= 10^500

**输出格式**
输出只有一行，代表a+b的值

**思路**
大数加法，自己大一做的第一道题，hhhh，从最后一位相加，判断是否有进位即可

**code**
```cpp
const int maxx=505;
int ta[maxx],tb[maxx];
char sa[maxx],sb[maxx];
int ans[maxx];

int main(){
    cin>>sa>>sb;
    int lena=strlen(sa),lenb=strlen(sb);
    int m=min(lena,lenb);
    int n=0,flag=0;
    for(int i=1;i<=m;i++){
        ans[n]=(sa[lena-i]-'0')+(sb[lenb-i]-'0')+flag;
        flag=ans[n]/10;
        ans[n]%=10;
        n++;
        // cout<<n<<" "<<i<<endl;
        // for(int j=n-1;j>=0;j--){
        //     cout<<ans[j];
        // }
        // cout<<endl;
    }
    if(m==lena){
        for(int i=m+1;i<=lenb;i++){
            ans[n]=(sb[lenb-i]-'0')+flag;
            flag=ans[n]/10;
            ans[n]%=10;
            //cout<<ans[n]<<endl;
            n++;
            
        }
    }
    else if(m==lenb){
        for(int i=m+1;i<=lena;i++){
            ans[n]=(sa[lena-i]-'0')+flag;
            flag=ans[n]/10;
            ans[n]%=10;
            //cout<<ans[n]<<endl;
            n++;
        }
    }
    if(flag!=0) ans[n++]=flag;
    for(int i=n-1;i>=0;i--){
        cout<<ans[i];
    }
}
```

## P1141 01迷宫
**题目描述**
有一个仅由数字0与1组成的n×n格迷宫。若你位于一格0上，那么你可以移动到相邻4格中的某一格1上，同样若你位于一格1上，那么你可以移动到相邻4格中的某一格0上。
你的任务是：对于给定的迷宫，询问从某一格开始能移动到多少个格子（包含自身）。

**输入格式**
第11行为两个正整数n,m。
下面n行，每行n个字符，字符只可能是0或者1，字符之间没有空格。
接下来m行，每行2个用空格分隔的正整数i,j，对应了迷宫中第i行第j列的一个格子，询问从这一格开始能移动到多少格。

**输出格式**
mm行，对于每个询问输出相应答案。、

**思路**
其实就是连通块问题，连通块中有n个节点，就将连通块中所有的节点赋值为n；我先把整个图跑出来，查找的过程就是o(1)。

**code**
```cpp
const int maxx=1005;
int m,n;
char ch[maxx][maxx];
int d[maxx][maxx];
int fx[4]={1,-1,0,0},fy[4]={0,0,1,-1};
bool visit[maxx][maxx]={false};
struct node{
    int x,y;
}Node;
bool check(int x1,int y1,int x2,int y2){
    if(x1<0||y1<0||x1>=n||y1>=n) return false;
    if(ch[x1][y1]==ch[x2][y2]||visit[x1][y1]==true) return false;
    return true;
}
void dfs(int x,int y){
    Node.x=x,Node.y=y;
    int sum=1;
    queue<node>q;
    queue<node>Q;
    q.push(Node);
    while(!q.empty()){
        node f=q.front();
        q.pop();
        Q.push(f);
        visit[f.x][f.y]=true;
        for(int i=0;i<4;i++){
            int newx=f.x+fx[i];
            int newy=f.y+fy[i];
            if(check(newx,newy,f.x,f.y)){
                sum++;
                Node.x=newx;
                Node.y=newy;
                visit[newx][newy]=true;
                q.push(Node);
            }
        }
    }
    while (!Q.empty()){
        node a=Q.front();
        Q.pop();
        d[a.x][a.y]=sum;
    }
}
int main(){
    cin>>n>>m;
    for(int i=0;i<n;i++){
        for(int j=0;j<n;j++){
            cin>>ch[i][j];
        }
        getchar();
    }
    for(int i=0;i<n;i++){
        for(int j=0;j<n;j++){
            if(visit[i][j]==false){
                dfs(i,j);
            }
        }
    }
    while (m--){
        int x,y;
        cin>>x>>y;
        cout<<d[x-1][y-1]<<endl;
    }
    return 0;
}
```