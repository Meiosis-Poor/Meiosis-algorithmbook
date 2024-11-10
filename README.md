# 算法&数据结构模板

# 算法

## 高精度

### 函数高精度

* 高精+高精

**<mark>正数+负数转变为正数-正数，负数＋负数转变为-（正数+正数）</mark>**

```cpp
string Add(string num1,string num2){
    int len1=num1.length();
    int len2=num2.length();
    int max;
    string num;
    //补齐位数
    if(len1>len2){
        for(int i=0;i<len1-len2;i++) num2="0"+num2;
        max=len1;
    }
    else{
        for(int i=0;i<len2-len1;i++) num1="0"+num1;
        max=len2;
    }
    int temp=0,p=0; //p为进位的数
    for(int j=max-1;j>=0;j--){
        temp=num1[j]-'0'+num2[j]-'0'+p;
        p=temp/10;
        temp%=10;
        num=char(temp+'0')+num;
    }
    //第一位进位
    if(p>0){
        num=char(p+'0')+num;
    }
    return num;
```

* 高精-高精

暂不支持负数

```cpp
bool cmp(string num1,string num2){
    int len1=num1.length();
    int len2=num2.length();
    if(len1 > len2) return true;
    else if(len1==len2){
        for(int i=0;i<len1;i++) if(num1[i]-'0'<num2[i]-'0') return false;
        return true;
    }
    else return false;
}

string Sub(string num1,string num2){
    string num;
    if(cmp(num1,num2)){
        int len1=num1.length();
        int len2=num2.length();
        int g=0,temp=0;
        for(int i=0;i<len1;i++){
            temp=num1[len1-1-i]-'0'-g;
            if(i<len2) temp-=num2[len2-1-i]-'0';
            //借位
            if(temp<0){
                temp+=10;
                g=1;
            }
            else g=0;
            num=char(temp+'0')+num;
        }
        return num;
    }
    else{
        num=Sub(num2,num1);
        num="-"+num;
        return num;
    }
}

void print(string num){
    int len=num.length();
    int k=0;
    if(num[0]=='-'){
        printf("-");
        k++;
    }
    while (len-1>k&&num[k]=='0') k++; //消掉首位0
    for(int i=k;i<len;i++){
        printf("%c",num[i]);
    }
}
```

* 高精*低精

同高精*高精

```cpp
string Mul(string num1,int num2){
    if(num1=="0" || num2==0) return "0"; //特判乘0
    int len1=num1.length();
    string num;
    int n[MAX*2]={0};
    //将数乘进数组
    for(int i=0;i<len1;i++) n[i]+=(num1[len1-1-i]-'0')*num2;
    //进位并存到字符串里
    for(int k=0;;k++){
        if(k>len1-1 && n[k]==0) break;
        n[k+1]+=n[k]/10;
        n[k]%=10;
        num=char(n[k]+'0')+num;
    }
    return num;
}
```

* 高精/低精

```cpp
string Div(string num1,long long num2){
    string num;
    long long temp=0;
    int len=num1.length();
    long long t;
    for(int i=0;i<len;i++){
        temp=temp*10+(num1[i]-'0'); //往后进一位
        t=temp/num2;
        if(t==0) num=num+"0"; //不够除就继续进一位除（打印时记得消首位0）
        else{
            temp%=num2; //余数继续参与除法
            num=num+char(t+'0');
        }
    }
    return num;
}
```

* 高精*高精

```cpp
string Mul(string num1,string num2){
    if(num1=="0" || num2=="0") return "0"; //特判乘0
    int len1=num1.length();
    int len2=num2.length();
    string num;
    int n[MAX*2]={0};
    //将数乘进数组
    for(int i=0;i<len1;i++){
        for(int j=0;j<len2;j++) n[i+j]+=(num1[len1-1-i]-'0')*(num2[len2-1-j]-'0');
    }
    int len=len1+len2-1;
    //进位并存到字符串里
    for(int k=0;;k++){
        if(k>len-1 && n[k]==0) break;
        n[k+1]+=n[k]/10;
        n[k]%=10;
        num=char(n[k]+'0')+num;
    }
    return num;
}

void print(string num){
    int len=num.length();
    int k=0;
    if(num[0]=='-'){
        printf("-");
        k++;
    }
    while (len-1>k&&num[k]=='0') k++; //消掉首位0
    for(int i=k;i<len;i++){
        printf("%c",num[i]);
    }
}
```

* 高精/高精

用减法模拟除法，对被除数的每一位都减去除数，一直减到当前位置的数字（包含前面的余数）小于除数

### 重构高精度

**皆为正数时使用，如有负数需加特判**

```cpp
#include <cstring>
#define N 1001
struct bign
{
    int len,s[N];
    bign()  {  memset(s,0,sizeof(s));  len=1;  } //开辟动态空间
    bign(int num)  {  *this=num; }
    bign(char *num) { *this=num; }
    //重构=（将整数num转化为字符数组，将字符数组num转化为整数数组）
    bign operator =(int num)
    {
        char c[N];
        sprintf(c,"%d",num);
        *this=c;
        return *this;
    }
    bign operator =(const char *num)
    {
        len=strlen(num);
        for (int i=0;i<len;i++) s[i]=num[len-1-i]-'0';
        return *this;
    }
    //将整数数组转化为字符串
    string str()
    {
        string res="";
        for (int i=0;i<len;i++) res=(char)(s[i]+'0')+res;
        return res;
    }
    //消除首位0
    void clean()
    {
        while (len>1&&!s[len-1]) len--;
    }
    //重构+
    bign operator +(const bign &b)
    {
        bign c;
        c.len=0;
        for (int i=0,g=0;g||i<len||i<b.len;i++)
        {
            int x=g;
            if (i<len) x+=s[i];//按位加a和b，没有该位跳过
            if (i<b.len) x+=b.s[i];
            c.s[c.len++]=x%10;
            g=x/10;//进位
        }
        return c;
    }
    //重构-
    bign operator -(const bign &b)
    {
        bign c;
        c.len=0;
        int x;
        for (int i=0,g=0;i<len;i++)
        {
            x=s[i]-g;//g为上一位借的数
            if (i<b.len) x-=b.s[i];
            //不够借位
            if (x>=0) g=0;
            else{
                x+=10;
                g=1;
            };
            c.s[c.len++]=x;
        }
        c.clean();
        return c;
    }
    //重构*（高精*高精）
    bign operator *(const bign &b)
    {
        bign c;
        c.len=len+b.len;
        //每一位分开乘进对应位数中[a*10^i*b*10^j=c*10^(i+j)]
        for (int i=0;i<len;i++) for (int j=0;j<b.len;j++) c.s[i+j]+=s[i]*b.s[j];
        //进位
        for (int i=0;i<c.len-1;i++) {
            c.s[i+1]+=c.s[i]/10;
            c.s[i]%=10;
        }
        c.clean();
        return c;
    }
    //重构<
    bool operator <(const bign &b)
    {
        if (len!=b.len) return len<b.len; //从长度上直接比较
        for (int i=len-1;i>=0;i--) //逐位比较
            if (s[i]!=b.s[i]) return s[i]<b.s[i];
        return false;
    }
    bign operator +=(const bign &b)
    {
        *this=*this+b;
        return *this;
    }
    bign operator -=(const bign &b)
    {
        *this=*this-b;
        return *this;
    }
};
```

### 压位数组

## 排序

### 冒泡排序 O（ ）

后函数引用皆为vector容器变量

比较相邻两个变量然后交换

```cpp
void Bubble_sort(int num[],int len){
    int temp;
    for(int i=0;i<len-1;i++){
        for(int j=i+1;j<len-i-1;j++){
            if(num[j]>num[j+1]){
                temp=num[j];
                num[j]=num[j+1];
                num[j+1]=temp;
            }
        }
    }
    return;
}
```

### 选择排序 O（ ）

- 第一种：遍历一遍找到最小值然后交换

```cpp
void Selection_sort1(vector<int> &num){
    int n=num.size();
    int min;
    for(int i=0;i<n-1;i++){
        min=i;
        for(int j=i+1;j<n;j++){
            if(num[j]<num[min]) min=j;
        }
        swap(num[i],num[min]);
    }
    return;
}
```

- 第二种：遍历中发现较小值就交换

```cpp
void Selection_sort2(vector<int> &num){
    int n=num.size();
    for(int i=0;i<n-1;i++){
        for(int j=i+1;j<n;j++){
            if(num[j]<num[i]) swap(num[i],num[j]);
        }
    }
    return;
}
```

### 插入排序 O（ ）

逐个遍历每个变量，不断将其与前一个变量比较并交换，直到前面成顺序排列

```cpp
void Insertion_sort(vector<int>& num){
    int n=num.size();
    for(int i=0;i<n;i++){
        for(int j=i;j>0;j++){
            if(num[j]<num[j-1]) swap(num[j-1],num[j]);
            else break; //前面已经排列好了遇到更小的直接跳出遍历
        }
    }
    return;
}
```

### 快速排序 O（n ）

取最左端数为基准，将大于和小于其的数分别放置在其右边和左边，继续递归左右两边直至完全顺序

```cpp
void Quick_sort(vector<int>& num,int l,int r){
    if(l>=r) return; //越界返回
    int base=num[l],i=l,j=r;
    while(i<j){
        while(num[j]>base && i<j) j--; //重合时j指针主动与i指针重合指向的值小于base
        while(num[i]<=base && i<j) i++;
        if(i<j) swap(num[i],num[j]);
    }
    swap(num[l],num[i]);
    Quick_sort(num,l,i-1);
    Quick_sort(num,i+1,r);
    return;
}
```

### 归并排序 O（n ）

对数列不断二分，分到仅有一个数时开始逐个回溯，回溯时将分开的两个子数列进行排序（如需要）并归并。

归并时分别看两个子序列最小值（最左边的数），取出最小值放入数组中，然后再比较下一个最小值放入数组，直到两个子序列全部取完

```cpp
//对一个区间进行归并
void Merge(vector<int>& num,int l,int mid,int r){
    vector<int> temp=num; //取出数组
    int i=l,j=mid+1; //i，j分别指向已经排成顺序的两个子数列的首位
    int index=l; //重新排列计数指针
    while(i<=mid || j<=r){ //比大小逐个放入新数组中
        if(i>mid){ //如果有一边放完直接按顺序放另一边
            temp[index++]=temp[j];
            j++;
        }
        else if(j>r){
            temp[index++]=temp[i];
            i++;
        }
        else if(temp[i]<=temp[j]){
            temp[index++]=temp[i];
            i++;
        }
        else{
            temp[index++]=temp[j];
            j++;
        }
    }
    return;
}

void Merge_sort(vector<int>& num,int l,int r){
    if(l>=r) return; //二分到仅剩一个数时返回
    int mid=(l+r)/2;
    Merge_sort(num,l,mid); //不断二分，返回时归并
    Merge_sort(num,mid+1,r);
    if(num[mid]>num[mid+1]) Merge(num,l,mid,r); //如果无法自动合并成顺序，进行归并
    return;
}
```

## 搜索

### 二分查找（有序！！！！）

<mark>用这两个函数之前必须是在**有序的数组**中！！！！</mark>

**lower_bound** 返回第一个值不小于 val 的位置,**也就是返回第一个大于等于val值的位置**

```cpp
bool cmp(const int& e, const int& val)
{
    return e >= val; //根据需求写，默认是返回大于等于val
}

lower_bound( begin , end , val , cmp )
```

**upper_bound** 返回第一个值大于 val 的位置，**也就是返回第一个大于val值的位置**

### 折半搜索

当dp操作次数较少而开辟空间过大时改为搜索，折半搜索将O（ ）优化为O（ ），合并时，先将一部分进行排列使其有序，然后遍历另一部分，每次进行二分搜索查找可行的答案，然后叠加

eg：

```cpp
void dfs(int l,int r,long long sum,vector<long long>&now)
{
    if(sum>m) return;//如果当前花费超过限额，则返回
    if(l>r){ //起点超过终点说明该部分已经全部搜索完毕
        now.push_back(sum); //则将可行的花费塞入vector中
        return;
    }
    dfs(l+1,r,sum+w[l],now); //选择买这场比赛的门票
    dfs(l+1,r,sum,now); //选择不买这场比赛的门票
}
```

```cpp
vector <long long>a,b;
dfs(1,n/2,0,a);//搜索第一部分
dfs(n/2+1,n,0,b);//搜索第二部分
sort(a.begin(),a.end());//将第一部分排序，使其有序
long long ans=0;
int lenb=b.size();
for(int i=0;i<lenb;i++) ans+=upper_bound(a.begin(),a.end(),m-b[i])-a.begin();
//每次寻找花费比剩下的钱还要少的方案数，注意这里要使用upper_bound
//若使用lower_bound，则出现等于的情况时，方案数会有错误
```

### DFS

#### 回溯

#### 剪枝

### BFS

### Dijkstra

### A*

### IDA*

### 双向搜索

## 贪心

## 模拟

## 动态规划dp

### 记忆化搜索

### 线性dp

#### 前缀和

* 一维前缀和

将1-i的值累加并存储，相减可以求区间和（有需要可取模）

```cpp
for(int i=1;i<=n;i++){
    scanf("%d",&num[i]);
    num[i]=(num[i]+num[i-1])%mod;
}
```

* 二维前缀和

```cpp
//方法一
for (int i = 1; i <= n; i ++){
    for (int j = 1; j <= m; j ++) s[i][j] = s[i - 1][j] + s[i][j - 1] - s[i - 1][j - 1] + s[i][j];
}

// 方法二
for (int i = 1; i <= n; i ++){
    for (int j = 1; j <= m; j ++) s[i][j] += s[i][j - 1];
for (int i = 1; i <= m; i ++) s[i][j] += s[i - 1][j];
}
```

#### 差分

<mark>差分主要用于让一个序列某一特定范围内的所有值都加上或减去一个常数</mark>

* 一维差分：

a[0 ]= 0;

b[1] = a[1] - a[0];

b[2] = a[2] - a[1];

........

b[n] = a[n] - a[n-1];

a数组是b数组的前缀和数组， b数组是a数组的差分数组

根据题目读入b数组，对其求前缀和得到a数组

```cpp
void insert(int l, int r, int c) {
    b[l] += c;
    b[r+1] -= c;
}

for (i = 1; i <= n; i++) insert(i, i, a[i]); //初始化b数组
//对b数组求前缀和，还原a数组
for(int i = 1; i <= n; i++){
    b[i] += b[i-1];
    printf("%d ",b[i]);
}
```

* 二维差分：

二维差分是指对于一个n*m的矩阵a，对于以(x1,y1)为左上角，(x2,y2)为右下角的矩形区域，每个元素都加上常数a，求修改后的矩阵a。

```cpp
void insert(int x1,int y1,int x2,int y2,int c)
{
    b[x1][y1] += c;
    b[x2+1][y1] -= c;
    b[x1][y2+1] -= c;
    b[x2+1][y2+1] += c;
}

//初始化
for(int i = 1; i <= n; i++){
    for(int j = 1; j <= m; j++) insert(i,j,i,j,a[i][j]);
}

//求前缀和
for(int i = 1; i <= n; i++){
    for(int j = 1; j <= m; j++){
        b[i][j] += b[i-1][j] + b[i][j-1] - b[i-1][j-1]; //两块大的相加减重复
        printf("%d ",b[i][j]);
    }
    printf("\n");
}
```

### 背包dp

### 区间dp

### 树形dp

在DFS遍历树时通过之前节点（从叶节点开始）或者区间的值dp出当前节点的值,即在遍历结束返回时进行dp，可以用dp[0][i],dp[1][i]表示是否取编号为i的节点值（根据题意改写）

### 状态压缩dp

将多维问题转化为二进制解决，极大节省空间和时间，将多维之间的关系用位运算表达判断，进而得出dp方程，因为转化为二进制，**只适用于维度适中且关系单一的数据量**（二进制能存两种关系，三进制更多但没学会……）

记录状态轮数如果较多且仅比较最近几个状态可以采用滚动数组降低空间复杂度

dp数组一般标号为存储的状态，变化多端需要仔细分析

补充一个二进制判断1的个数的函数方法

```cpp
int number(long long m) {
    int count = 0;
    while (m) {
        m = m & (m - 1);
        count++;
    }
    return count;
}
```

### 数位dp（处理数字位数或者个别数字出现）

构建dp[i][j]，i表示当前位数，j表示当前首位数字，根据题意构建dp[i][j]与dp[i-1][k]的关系（0<k<10）

### 单调队列优化的dp

能极大降低时间复杂度，有时候使用挺巧妙的，但是真的想不到~

<mark>dp[i]=min(dp[i],dp[j]+f[i][j])</mark>  其中 (l[i]<=j<=r[i]) , l[i]与 r[i] 的序列单调不减。

这种 DP 可以用单调队列分 4 步进行优化：

①    将所有小于 l[i] 的 j 全部踢队。

②    用队首 j 为 dp[i] 赋值，队首 head 要变化。

③    将新元素 j 加入队尾 (j<=l[i])，队尾 tail 也要变化。

④    维护单调队列。

```cpp
long long Min=1e9;
scanf("%d %d",&n,&k);
for(int i=1;i<=n;i++){
    scanf("%lld",&num[i]);
    sum+=num[i];
    if(i>k+1) dp[i]=dp[d.front()]+num[i];
    else dp[i]=num[i];
    if(!d.empty() && d.front()<=i-k-1) d.pop_front();
    while(!d.empty() && dp[i]<dp[d.back()]) d.pop_back();
    d.push_back(i);
}
for(int j=n-k;j<=n;j++) Min=min(Min,dp[j]);
printf("%lld",sum-Min);
```

## 字符串

### 哈希

将大规模字符串快速匹配，将每个字符串/子串映射到某个特定的值上

**eg:BKDRHash**

<mark>注：取余可能会产生冲突问题，根据题目再取余</mark>

```cpp
int Hash(char *str){
    int seed=31,key=0;
    while(*str) key=key*seed+(*str++);
    return (key&0x7fffffff)%mod;
}
```

### KMP（高效寻找已给出的重复子串）

对已给参考字符串进行预处理，求出next数组即如果在对比时下一位对不上号返回的字符串下标，在比对时直接将字符串指针移至参考字符串j位置。

Next数组：

1. 根据定义，不断延参考字符串向后遍历已有next数组，如果指向字符与该字符不相同，则继续沿着指向字符的next继续判断，直至指针指向字符与该字符相同（最大公共前后缀）或者指向0（无法匹配），将next下一位赋上指针的下一位（j+1）或者0（从哪一位继续判断）

2. 本质上，next[i]为参考字符串前i位字符子串的最大公共前后缀的长度

```cpp
void getFail(char* p,int plen){ //plen为字符串p的长度
    Next[0]=0;
    Next[1]=0;
    for(int i=1;i<plen;i++){
        int j=Next[i];
        while(j&&p[i]!=p[j]) j=Next[j];
        Next[i+1]=(p[i]==p[j])?j+1:0;
    }
    return;
}
```

kmp实现：i不断往下遍历字符串（不回头），j指向已给参考字符串，将i与j对比，如果字符相同同时向后一位继续对比，如果不同j指向next[j]继续对比直至i遍历完

```cpp
void kmp(char *s,char *p){
    int last=-1; //最后一个重复的位置
    int slen=strlen(s);
    int plen=strlen(p);
    getFail(p,plen);
    int j=0;
    for(int i=0;i<slen;i++){
        while(j&&s[i]!=p[j]) j=Next[j];
        if(s[i]==p[j]) j++;
        if(j==plen){
            //根据题意写
            cnt++;
            last=i;
        }
    }
    return;
}
```

#### 前缀函数s[i]

定义：S的子串S’（0，i）在有一对相等的前缀和后缀，是s[i]为其长度 eg：”abaabca”子串”abaab”有相等前缀后缀“ab“长度为2，s[5]=2

计算：

1. 相邻的前缀函数值至多增加1

2. 

```cpp
char s1[MAX]={0};
char s2[MAX]={0};
int Next[MAX]={0};
int cnt; //重复个数

void getFail(char* p,int plen){ //plen为字符串p的长度
    Next[0]=0;
    Next[1]=0;
    for(int i=1;i<plen;i++){
        int j=Next[i];
        while(j&&p[i]!=p[j]) j=Next[j];
        Next[i+1]=(p[i]==p[j])?j+1:0;
    }
    return;
}

void kmp(char *s,char *p){
    int last=-1; //最后一个重复的位置
    int slen=strlen(s);
    int plen=strlen(p);
    getFail(p,plen);
    int j=0;
    for(int i=0;i<slen;i++){
        while(j&&s[i]!=p[j]) j=Next[j];
        if(s[i]==p[j]) j++;
        if(j==plen){
            //根据题意写
            if(i-last>=plen){
                cnt++;
                last=i;
            }
        }
    }
    return;
}
```

### Trie树（字典树）

从根节点出发（不算根节点）每一层节点存储第n个字母（也有用边存储字母的），能够快速匹配字符串/前缀，输入法匹配要输入的字即是这种思路

注：如果建结构体实现可能会MLE，建议使用数组建树；**每次重新建树记得初始化数组和pos！！memset大概率TLE**

* 数值映射
  
  将字符转化为对应的数，可以直接将字符减去’0‘/’a'
  
  ```cpp
  int getnum(char x){
      if(x>='A'&&x<='Z') return x-'A';
      else if(x>='a'&&x<='z') return x-'a'+26;
      else return x-'0'+52;
  }
  ```

* 插入
  
  将已知字符串每个字符从根节点出发依此插入，向下遍历，如果没有该节点新建节点并向下延伸，更新子树字符串结尾数和
  
  <mark>注：根节点不存值故pos=1</mark>
  
  ```cpp
  void Insert(string s){
      int len=s.length();
      int p=0;
      for(int i=0;i<len;i++){
          int temp=getnum(s[i]);
          if(trie[p][temp]==0) trie[p][temp]=pos++;
          p=trie[p][temp];
          cnt[p]++;
      }
      return;
  }
  ```

* 检索字符串
  
  将要搜索的字符串每个字符依此在trie上查找，如果完全查完该结尾节点的cnt即为以其为前缀的已知字符串数量，如果有节点为空即没有该前缀的字符串，其他形式可由题意改得
  
  ```cpp
  int Find(string s){
      int len=s.length();
      int p=0;
      for(int i=0;i<len;i++){
          int temp=getnum(s[i]);
          if(trie[p][temp]==0) return 0;
          p=trie[p][temp];
      }
      return cnt[p];
  }
  ```

### AC自动机

AC 自动机是 **以 Trie 的结构为基础**，结合 **KMP 的思想** 建立的自动机，用于解决多模式匹配等任务。

首先建立一个trie树，建立fail失配指针，BFS遍历一遍字典树构建每个节点的失配指针，在查找的时候如果下一位不匹配则指向失配指针指向的节点位置

* 构建失配指针
  
  先将第一层预处理，使其指向源点，将第一层所有节点放入队列中，依此遍历队列中所有节点的子节点（a-z）,如果节点存在,将该节点的失配指针指向其父节点所指向的指针的下一个与其存储字符相同的子节点，并将该节点放入队列；如果该子节点不存在，向上连结对应字符的节点
  
  ```cpp
  void getFail(){
      queue<int>q;
      for(int i=0;i<26;i++){
          if(trie[0][i]){
              f[trie[0][i]]=0;
              q.push(trie[0][i]);
          }
      }
      while(!q.empty()){
          int now=q.front();
          q.pop();
          for(int i=0;i<26;i++){
              if(trie[now][i]){
                  f[trie[now][i]]=trie[f[now]][i];
                  q.push(trie[now][i]);
              }
              else trie[now][i]=trie[f[now]][i];
          }
      }
      return;
  }
  ```

* 查找
  
  遍历字符串，从字典树根节点出发，如果遇到不匹配的字符，将指针移至该节点失配指针指向的节点，如果遍历到的节点为完整字符串的结尾，则包含该完整字符串
  
  注：根据题意进行ans计算和cnt的标记
  
  ```cpp
  int query(string s){
      int now = 0,ans = 0;
      for(int i=0;i<s.size();i++){    //遍历文本串
          now = trie[now][s[i]-'a'];  //从s[i]点开始寻找
          for(int j=now;j && cnt[j]!=-1;j=f[j]){
              //一直向下寻找,直到匹配失败(失败指针指向根或者当前节点已找过).
              ans += cnt[j];
              cnt[j] = -1;    //将遍历过后的节点标记,防止重复计算
          }
      }
      return ans;
  }
  ```

### 回文自动机（Manacher算法）

### 后缀数组

### 后缀自动机

## 离散化

<mark>把无限空间中有限的个体映射到有限的空间中去，以此提高算法的时空效率</mark>

1. **原来相等的值离散化后仍然相等**。那么在对原序列进行**排序**后，进行**去重操作**即可

```cpp
int n;  //原序列长度
int a[1005];//原序列
int len;   //去重后的序列长度
int f[1005];//原序列的副本
for(int i=0;i<n;i++)
{
    scanf("%d",&a[i]);
    f[i]=a[i];
}
sort(f,f+n);//排序
len=unique(f,f+n)-f;//去重后的序列长度
for(int i=0;i<n;i++) a[i]=lower_bound(f,f+len,a[i])-f; //离散化
for(int i=0;i<n;i++) f[i]=a[i];
sort(f,f+n);
len=unique(f,f+n)-f;
for(int i=0;i<n;i++) a[i]=upper_bound(f,f+len,a[i])-f; //离散化
```

2. **原来相等的值，离散化后不相等。那不进行去重操作就好了，但是要新增一个id属性**

```cpp
struct node
{
    int val,id;
    bool operator < (const node &a)const//重载小于运算符用来排序
    {
        return val<a.val;  //值小的在前面
    }
};

int n; //序列长度
node a[1005];//原序列
int f[1005];//离散化后的序列
scanf("%d",&n); 
for(int i=0;i<n;i++){
    scanf("%d",&a[i]);
    a[i].id=i+1;
}
sort(a,a+n);
for(int i=0;i<n;i++) f[a[i].id]=i+1;
```

## 数论

### 位运算

皆在二进制下完成，比较基础，学会了用起来真的很方便

* 与运算&
  
  **只有对应位都为1时，结果才为1**
  
  1&1=1；
  
  1&0=0；
  
  0&0=0；

* 或运算|
  
  **只要对应位有一个为1，结果就是1**
  
  1&1=1；
  
  1&0=1；
  
  0&0=0；

* 异或运算^
  
  **对应位相同为0，不同为1**
  
  1&1=0；
  
  1&0=1；
  
  0&0=0；

* 取反~
  
  对每一位执行取反操作（**0变1，1变0**）

* 左移<<
  
  将二进制整体向左移动x位，左边缺位补0，几乎**相当于对该数乘以$2^x$倍**

* 右移>>
  
  将二进制整体向右移动x位，左边多余直接舍去?，几乎**相当于对该数除以$2^x$倍**

### 快速幂

<mark>a^b mod p：</mark>

```cpp
long long mod(long long a,long long b,long long p){
    if(b==1) return a;
    else if(b==0) return 1;
    long long temp=mod(a,b/2,p);
    if(b%2==0)  return temp*temp%p;
    else return temp*temp%p*a%p;
}
```

### 素数

```cpp
#include <cmath>
bool isPrime(int n){
    for(int i=2;i<=sqrt(n);i++){
        if(n%i==0) return false;
    }
    return true;
}
```

### 最大公约数（GCD）

小技巧：对于任意a、b,总存在$ax+by=gcd(a,b)$！！

```cpp
int gcd(int a, int b) {
    if (b == 0)  return a;
    return gcd(b, a % b);
}
```

### 费马小定理(取模后用来求超大数的倒数（作为分母时）)：

<mark>一般使用快速幂解决得出答案</mark>

**如果p是一个质数，而整数a不是p的倍数，则有$a^{(p-1)}≡1（mod p）$可以通过$a^{（p-2）}$代替$1/a$**

#### 求组合数nCr(排列数同理)

根据公式$nCr=n!/(n−r)!r!​$，先打表各个阶乘得数，然后使用费马小定理求分母的值

```cpp
for(int i=1;i<=2e5;i++) fact[i] = (fact[i - 1] * i) % mod;

long long modExp(long long base, long long exp) { //循环版快速幂
    long long result = 1;
    while (exp > 0) {
        if (exp % 2 == 1)
            result = (result * base) % mod;
        base = (base * base) % mod;
        exp /= 2;
    }
    return result;
}

long long modInverse(long long a) {
    return modExp(a, mod - 2); //a的p-2次方
}

long long nCr(long long n, long long r) {
    if (r > n) return 0;
    long long ans = (fact[n] * modInverse((fact[n - r] * fact[r]) % mod)) % mod;
    return ans;
}
```

### 同余方程

#### 中国剩余定理

#### 线性同余方程

#### 高次同余方程

### 博弈

### 排列组合

### 线代

#### 矩阵

### 多项式

### 筛法

## 图论

### 最小生成树

* Prim算法（对点贪心，适用于稠密图）
  
  **不好用！！！！**
  
  预处理：将距离数组和邻接矩阵初始化（赋INF）
  
  对任意取一点，将距离其最近的点放入集合中，再将距离两点最近的点放入集合，以此类推
  
  eg：使用邻接矩阵，在此仅举出生成树的边值，如需建树还需要记录对应的边的编号
  
  ```cpp
  void prim(int start=1){
      dis[start]=0;
      isIn[start]=true;
      for(int i=1;i<=n;i++) dis[i]=min(dis[i],road[start][i]);
      for(int i=1;i<n;i++){ //重复n-1次
          int temp=INF;
          int t=-1; //距离最近的点的编号
          for(int j=1;j<=n;j++) {
              if(!isIn[j]&&dis[j]<temp){ //找到距离集合最近且未使用的节点
                  temp=dis[j];
                  t=j;
              }
          }
          isIn[i]=true; //标记使用节点
          if(t==-1){ //没有找到，不连通
              return;
          }
          for(int j=1;j<=n;j++){ //将该节点放入集合并更新距离
              if(j==start) continue;
              dis[j]=min(dis[j],road[t][j]);
          }
      }
  }
  ```

* Kruskal算法（对边贪心，适用于稀疏图）

**复杂度更低，但不适用边过多的情况**

预处理：将并查集初始化

<mark>1. 对边值从小到大排序 sort()</mark>

<mark>2. 将边的始终点放入并查集(重复的即x==y说明如果加上该边形成了一个环，放弃该边)，遍历所有边直至结束</mark>

```cpp
void kruskal(){
    sort(e+1,e+m+1,cmp);
    for(int i=1;i<=m;i++){
        int x=find(e[i].start);
        int y=find(e[i].end);
        if(x!=y){
            f[y]=x;
            ans+=e[i].val; //最小生成树边之和(根据题目选择用不用)
        }
    }
    return;
}
```

### 链式前向星(如果不是树感觉不如开vector数组)

head[u]：表示以u作为起点的第一条边的编号。

next[cnt]：表示编号为cnt的边的下一条边，这条边与cnt同一个起点。

to[cnt]：表示编号为cnt的边的终点。

初始时，令 head[u]=cnt=0，表示尚未有边，每个点都是孤立点。

当加入一条新的边e(u,v)时：

- cnt++，表示这条边的编号。

- next[cnt]=head[u]：将原来以u作为起点的第一条边，作为该边的后续边。

- to[cnt]=v：当前边的终点设置。

```cpp
void add(int u,int v)
{
    next[++cnt]=head[u];
    head[u]=cnt;
    to[cnt]=v;
}
```

带权图

```cpp
void add(int u,int v,int w)
{
    next[++cnt]=head[u];
    head[u]=cnt;
    to[cnt]=v;
    val[cnt]=w;
}
```

### 拓扑排序

将一个<mark>**有向无环图（DAG）**</mark>的所有节点排序，使得排在前面的节点不能依赖于排在后面的节点，有DFS和BFS两种方式实现，DFS逻辑更清楚简洁，但不好输出特定顺序（如字典序）的拓扑队列，BFS则能输出字典序（使用优先队列即可）

只列举BFS无前驱顶点优先（入度为0）的示例：预先遍历每个节点，将每个入度为0的节点推入队列，再遍历整个队列，将队首的节点移除，其所指向的所有节点入度减1，如果出现入度为0的节点，将其推入队列中

```cpp
void toposort(int n){
    for(int i=1;i<=n;i++){
        if(in[i]==0){
            ans.push(i);
            printf("%d ",i);
        }
    }
    while(!ans.empty()){
        int i=ans.front();
        ans.pop();
        for(int j : tree[i]){
            in[j]--;
            if(!in[j]){
                ans.push(j);
                printf("%d ",j);
            }
        }
    }
    return;
}
```

### 欧拉路

欧拉定理：

    有向图度数=入度-出度，无向图度数等于顶点的边数

* 欧拉回路：所有顶点都是偶数度（从哪来不走重复路回哪去）

* 欧拉路：有且只有两个顶点是奇数度（从一个奇数度顶点不走重复路到另一个奇数路顶点）

找路径：下为使用邻接矩阵处理的走过所有无向图边且不重复的找路径

```cpp
void find(int i){
    int j;
    for(j=1;j<=maxn;++j){ //遍历所有边
        if(map[i][j]){//判断是否可走
            map[i][j]--;
            map[j][i]--; //标记该点/边
            find(j); //dfs
        }
    }
    lu[++cnt]=i; //记录路径
}
```

### 连通性

#### 割点与割边

* 定理一：一棵树的根节点s是割点，当且仅当s有两个及以上的子节点

* 定理二：一棵树的非根节点u是割点，当且仅当u存在至少一个子节点v，v及其后代都没有回退边回退至u的祖先
  
  eg：无向图Tarjan算法
  
  **注：如果图不连通需要遍历每一个节点为根（已经遍历过的节点直接跳过，即!num[i]）**
  
  ```cpp
  for(int i=1;i<=n;i++){
          if(!num[i]){
              root=i;
              dfs(i,-1);
          }
      }
  ```
  
  ```cpp
  void dfs(int u,int fa){
      num[u]=++id;
      low[u]=id; //根据DFS序编号，构建DFS树
      int son=0; //记录子树个数
      bool isCut=false; //标记割点，防止重复放入
      for(int i=head[u];i!=0;i=nex[i]){
          int v=to[i];
          if(!num[v]){ //没遍历过的点
              son++;
              dfs(v,u);
              low[u]=min(low[v],low[u]); //遍历完子树后确定最小值
              //判断割边改为low[v]>num[u]即可
              if(low[v]>=num[u] && u!=root) isCut= true; //定理二
          }
          //处理回退边，子节点v为父节点u的祖先且不是u-v双向返回边
          else if(num[v]<num[u] && v!=fa) low[u]=min(num[v],low[u]);
      }
      if(u==root && son>=2) isCut= true; //定理一
      if(isCut) ans.push_back(u);
      return;
  }，
  ```

#### 双连通分量

点/边双连通分量定义：在双连通分量中，去掉任意一个点/边，该图仍是连通的

* 点双连通分量：个数由割点决定，将每个遍历的**边**依次放入栈中，每找到一个割点将边全部移出，该些边构成一个点双连通分量
  
  ```cpp
  void dfs(int u,int fa){
      num[u]=low[u]=++id;
      int son=0;
      s[++top]=u;
      for(int i=head[u];i!=0;i=nex[i]){
          int v=to[i];
          if(!num[v]){
              son++;
              dfs(v, u);
              low[u]=min(low[u],low[v]);
              //不需要判断不是根节点
              if(low[v]>=num[u]){
                  w++;
                  while(s[top+1]!=v) ans[w].push_back(s[top--]);
                  ans[w].push_back(u);
              }
          }
          else if(num[u]>num[v] && v!=fa) low[u]=min(num[v],low[u]);
      }
      //点双连通分量实际上存储的是边，该题输出的是这些边的结点，故割点会被两个或以上双连通分量占有
      //不论根节点是否为割点都会被包含，如果连通则最后一条边会被认为是回退边，根节点作为结束点
      //if(u==root && son>=2)
      if(fa==-1 && son==0) ans[++w].push_back(u);
      return;
  }
  
  void Tarjan(int n){
      for(int i=1;i<=n;i++){
          if(!num[i]){
              root=i;
              top=0;
              dfs(i,-1);
          }
      }
      return;
  ```

* 边双连通分量：先遍历一遍DFS树，标记出割边，再重新遍历，将相同low值的点看作一个“缩点”，每个缩点就是一个双连通分量
  
  **注：仅无向图可以，有向图要压栈**
  
  ```cpp
  void dfs(int u,int fa){
      num[u]=low[u]=++id;
      for(int i=head[u];i!=0;i=nex[i]){
          int v=to[i];
          if(!num[v]){
              dfs(v, u);
              low[u]=min(low[u],low[v]);
              //判断割边,将割边标记，注意无向图要标两边
              //偶数^1=偶数+1，奇数^1=奇数-1，故初始cnt=1，边从2，3一对开始
              if(low[v]>num[u]) isCut[i]=isCut[i^1]=1;
          }
          else if(num[u]>num[v] && v!=fa) low[u]=min(num[v],low[u]);
      }
      return;
  }
  
  void dfs2(int u,int lowid){
      dcc[u]=lowid; //标记遍历过该点
      ans[lowid].push_back(u);
      for(int i=head[u];i!=0;i=nex[i]){
          int v=to[i];
          if(dcc[v] || isCut[i]) continue; //如果遍历过该点或者是该边是割边则跳过
          dfs2(v,lowid);
      }
      return;
  }
  
  void Tarjan(int n){
      //标记割边
      for(int i=1;i<=n;i++){
          if(!num[i]) dfs(i,-1);
      }
      //将low值相同的放入一个双连通分量中
      for(int i=1;i<=n;i++){
          if(!dcc[i]) dfs2(i,++w);
      }
      return;
  }
  ```

#### 强连通分量 （有向图）

* **Kosaraju算法**

* **Tarjan算法**
  
  **定理三：一个SCC（强连通分量），从任何一点出发，都至少有一条路能绕回到自己**
  
  **<mark>Tarjan算法求出来的SCC编号即为反向拓扑排序！！！</mark>**
  
  在DFS中如果遍历到已经遍历过的点，这些点从栈中弹出构成强连通分量，进行缩点，将值累加，后续可以用缩点重新建图
  
  ```cpp
  void dfs(int u) {
      num[u] = low[u] = ++id;
      stk.push(u);
      vis[u] = 1; 
      for (int i = head[u]; i != 0; i = nex[i]) {
          int v = to[i];
          if (!num[v]) { 
              dfs(v);
              low[u] = min(low[u], low[v]);
          } 
          //如果v为已在栈中元素（即v为u的祖先与前面Tarjan里意义一样）
          else if (vis[v]) low[u] = min(low[u], num[v]);
      }
      if (num[u] == low[u]) {   // 找到一个强连通分量
          w++;
          int x;
          do {
              x = stk.top();
              stk.pop();
              dcc[x] = w;
              newval[w] += val[x];
              vis[x] = 0; 
          } while (x != u);
      }
      return;
  }
  
  void Tarjan(int n){
      for(int i=1;i<=n;i++){
          if(!num[i]) dfs(i);
      }
      return;
  }
  ```

* 边双连通分量：将相同low值的点看作一个“缩点”，每个缩点就是一个双连通分量

### 最短路问题

* Dijkstra算法（单源最短路）

运用贪心思想，从源点s开始，使用优先队列不断寻找距s最近距离的节点post，更新post所连接的节点距s距离，将已松弛且未遍历的点置入队列，不断循环直到队列未空

**注**：优先队列存储点与点到s的距离，对dis进行比较，建立一个结构体node重载运算符<

```cpp
struct node{
    int dis;
    int post;
    bool operator <( const node &x )const{ //重载运算符<
        return x.dis < dis;
    }
};

void dijkstra(int s,int n){
    for(int i=1;i<=n;i++) dis[i]=INF; //初始化距离  
    dis[s]=0;
    priority_queue<node>q;
    int inque[MAX]={0};
    q.push(node{0,s});
    while(!q.empty()){
        node temp=q.top();
        q.pop();
        int post=temp.post;
        if(inque[post]) continue;
        inque[post]=1;
        for(int j=head[post];j!=0;j=e[j].nex){
            int v=e[j].to;
            //松弛
            if(dis[v]>dis[post]+e[j].val){
                dis[v]=dis[post]+e[j].val;
                if(!inque[v]) q.push(node{dis[v],v});
            }
        }
    }
}
```

* Floyd算法

* Bellman-Ford算法

**注：时间复杂度过高**

对于**单源**最短路，每次对所有m条边进行遍历，如果通过相邻边得出的距离更短则更新，再次遍历，如果遍历次数超过n则有负环。

```cpp
void bf(int s,int n){
    for(int i=1;i<=n;i++) dis[i]=INF;
    dis[s]=0;
    int num=0;
    bool update=true;
    while(update){
        num++;
        update=false;
        if(num>n) break;
        for(int u=1;u<=n;u++) {
            for (int i = head[u]; i != 0; i = nex[i]) {
                int v = to[i];
                if(dis[v]>dis[u] + val[i]){
                    dis[v] =dis[u] + val[i];
                    update=true;
                }
            }
        }
    }
    return;
}
```

* SPFA/队列优化 Bellman-Ford算法（处理负边的权值，但时间复杂度过高）

判断负环：

1.开始算法前，调用拓扑排序进行判断是否成环？（一般不采用，浪费时间）

<mark>2. 如果某个点进入队列的次数超过N次则存在负环（N为图的顶点数）</mark>

选择源点后将源点序号推入队列，不断将队首元素弹出然后遍历其对应节点连结的边的权值，判断其是否小于已有dis数组中的值，如果是且该节点序号未进入队列，将其推入队列并修改dis数组，不断循环至队列为空。

<mark>注：每次推入时记录该元素序号的进入次数，如果大于顶点数n说明存在负环，直接返回false</mark>

```cpp
bool SPFA(int s,int n){ //源点为s，顶点数为n
  queue<int> q;
  dis[s]=0; //源点距离自己为0
  q.push(s);
  num[s]++;
  while(!q.empty()){
      int p=q.front();
      if(num[p]>n) return false; //元素进队数量大于顶点数，为负环
      q.pop();
      inqueue[p]=false;
      for(int i=head[p];i!=0;i=nex[i]){
          int t=to[i];
          if(val[i]+dis[p]<dis[t]){
              dis[t]=val[i]+dis[p];
              if(!inqueue[t]){ //如果指向元素未在队列中则添加
                  q.push(t);
                  inqueue[t]=true;
                  num[t]++;
              }
          }
      }
  }
  return true;
}
```

### 最近公共祖先（LCA）

1.<mark> 预处理：计算log2[MAX]用于倍增</mark>

```cpp
for(int k=2;k<=n;k++) log2[k]=log2[k/2]+1;
```

       f[a][k]代表a节点第2^k级祖先

<mark>注：无向图链式前向星要存两遍，内存也要开两倍</mark>

```cpp
//无向图要存两次
add(x,y);
add(y,x);
```

<mark>2.首先用dfs遍历一遍树（可用链式前向星存储），获取其深度，用倍增算法（dp）记录每个节点2^k次方祖先</mark>

```cpp
//cur为当前节点，fa为该节点父节点，刚开始如果没给根可以随意取根，父节点为0
void dfs(int cur,int fa){ 
    if(vis[cur]) return; //如果之前遍历过则返回
    f[cur][0]=fa;
    depth[cur]=depth[fa]+1;
    vis[cur]=true;
    for(int i=1;i<=log2[depth[cur]];++i) f[cur][i] = f[f[cur][i - 1]][i - 1]; //dp：cur的2^k级祖先是cur的2^(k-1)级祖先的2^(k-1)祖先
    for(int j=head[cur];j!=0;j=nexts[j]) if(to[j]!=fa) dfs(to[j],cur); //遍历所有连结的边
    return;
}
```

<mark>3.LCA主体：先使a，b位于同一深度，不断从头往回倍增跳找到最远的不是公共祖先的点，其父节点就是最近公共祖先</mark>

```cpp
int lca(int a,int b){
    if(depth[a]>depth[b]) swap(a,b); //保证b的深度大于a
    while(depth[a]!=depth[b]) b=f[b][log2[depth[b] - depth[a]]]; //使a，b位于同一深度
    if(a==b) return a; //如果同一深度下正好相遇，直接输出
    //找到最远的不是公共祖先的点
    for(int j=log2[depth[a]];j>=0;--j){
        if(f[a][j] != f[b][j]){
            a = f[a][j];
            b = f[b][j];
        }
    }
    //返回该节点父节点
    return f[a][0];
}
```

### 树上启发式合并

### 差分约束

通过链式前向星建有向图，再建立超级源点方便计算，使用[SPFA]()/队列优化_Bellman-Ford算法（处理负边的权值，但时判断是否负环并查找最短路，最短路径即为方程组的一组解（如果存在负环，最短路无解，则原不等式组也无解。）

<mark>注</mark>：1.eg中为a-b≤x，如果为大于等于，则SPFA中需更改修改dis数组的判断条件

2. 其实我们可以把 dist[0] 设为另一个数 𝑤 而不是0（或者把从0号点连向各点的边权设为 𝑤 ），那么我们得到的便是满足 𝑥1, 𝑥2, ..., 𝑥𝑛≤𝑤 的一组解。实际上，可以证明，它们是满足𝑥1, 𝑥2, ..., 𝑥𝑛≤𝑤的**最大解**（每个变量取到能取到的最大值）

```cpp
while(m--){
    int a,b,x;
    scanf("%d %d %d",&a,&b,&x);
    add(b,a,x);
    //add(a,b,-x); //无向图最短路径会出错，add二选一即可
}
for(int i=1;i<=n;i++) add(0,i,0); //创建超级源点（到所有点的距离都为0）
if(SPFA(0,n)){ //如果存在负环说明无解
    for(int i=1;i<=n;i++) printf("%d ",dis[i]); …//根据题意
}
else printf("NO"); …//根据题意
```

### 二分图

## 杂项

### 双指针

### 离散化

### 约瑟夫问题

### 珂朵莉树

### 模拟退火

### 莫队

```cpp
struct query{
    int l;
    int r;
    int num;
}q[MAX];

bool cmp(query q1,query q2){
    if(q1.l/unit!=q2.l/unit) return q1.l<q2.l;
    if(q1.l/unit & 1) return q1.r<q2.r;
    else return q1.r>q2.r;
}

void add(int a){
    if(cnt[num[a]]==0) cur++;
    cnt[num[a]]++;
    return;
}

void del(int a){
    cnt[num[a]]--;
    if(cnt[num[a]]==0) cur--;
    return;
}
```

#### 树上莫队

# 数据结构

## Vector容器

#include<vector>;

一、vector 的初始化：

    (1)vector<int> a(10); //定义了10个整型元素的向量（尖括号中为元素类型名，它可以是任何合法的数据类型），但没有给出初值，其值是不确定的。

   （2）vector<int> a(10,1); //定义了10个整型元素的向量,且给出每个元素的初值为1

   （3）vector<int> a(b); //用b向量来创建a向量，整体复制性赋值

   （4）vector<int> a(b.begin(),b.begin+3);
//定义了a值为b中第0个到第2个（共3个）元素

   （5）int b[7]={1,2,3,4,5,9,8};

        vector<int> a(b,b+7); //从数组中获得初值

二、vector对象的重要操作：

    （1）a.assign(b.begin( ), b.begin( )+3); //b为向量，将b的0~2个元素构成的向量赋给a

    （2）a.assign(4,2); //是a只含4个元素，且每个元素为2

    （<mark>3）a.back();</mark> //返回a的最后一个元素

    <mark>（4）a.front(); </mark>//返回a的第一个元素

    （5）a[i]; //返回a的第i个元素，当且仅当a[i]存在 *下标只能用于获取已存在的元素

    <mark>（6）a.clear();</mark> //清空a中的元素

<mark>（7）a.empty( );</mark> //判断a是否为空，空则返回ture,不空则返回false

<mark>（8）a.push_back( ); </mark>//在队尾插入一个元素

    <mark>（9）a.pop_back();</mark> //删除a向量的最后一个元素

    <mark>（10）a.erase(a.begin()+1,a.begin()+3);</mark> //删除a中第1个（从第0个算起）到第2个元素，也就是说删除的元素从a.begin( )+1算起（包括它）一直到a.begin()+3（不包括它）

    <mark>（11）a.insert(a.begin()+1,5); </mark>//在a的第1个元素（从第0个算起）的位置插入数值5，如a为1,2,3,4，插入元素后为1,5,2,3,4

    （12）a.insert(a.begin( )+1,3,5); //在a的第1个元素（从第0个算起）的位置插入3个数，其值都为5

    （13）a.insert(a.begin( )+1,b+3,b+6); //b为数组，在a的第1个元素（从第0个算起）的位置插入b的第3个元素到第5个元素（不包括b+6），如b为1,2,3,4,5,9,8，插入元素后为1,4,5,9,2,3,4,5,9,8

    <mark>（14）a.size();</mark> //返回a中元素的个数；

    （15）a.capacity( ); //返回a在内存中总共可以容纳的元素个数

    （16）a.resize(10); //将a的现有元素个数调至10个，多则删，少则补，其值随机

    （17）a.resize(10,2); //将a的现有元素个数调至10个，多则删，少则补，其值为2

    （18）a.reserve(100); //将a的容量（capacity）扩充至100，也就是说现在测试a.capacity();的时候返回值是100.这种操作只有在需要给a添加大量数据的时候才显得有意义，因为这将避免内存多次容量扩充操作（当a的容量不足时电脑会自动扩容，当然这必然降低性能）

    <mark>（19）a.swap(b);</mark> //b为向量，将a中的元素和b中的元素进行整体性交换

（20）a==b;

三、vector常用算法

#include<algorithm>

（1）<mark>sort(a.begin( ),a.end( ));</mark> //对a中的从a.begin()（包括它）到a.end()（不包括它）的元素进行从小到大排列

（2）<mark>reverse(a.begin( ),a.end( ));</mark> //对a中的从a.begin()（包括它）到a.end()（不包括它）的元素倒置，但不排列

（3）copy(a.begin( ),a.end(),b.begin( )+1); //把a中的从a.begin()（包括它）到a.end()（不包括它）的元素复制到b中，从b.begin()+1的位置（包括它）开始复制，覆盖掉原有元素

（4）<mark>find(a.begin( ),a.end( ),10);</mark> //在a中的从a.begin()（包括它）到a.end()（不包括它）的元素中查找10，若存在返回其在向量中的位置

## 链表（对中间元素操作不影响其他）

<mark>模板类list是一个容器，list是由双向链表来实现的，每个节点存储1个元素。list支持前后两种移动方向。</mark>

<mark>优势： 任何位置执行插入和删除动作都非常快</mark>

#include <list>

list 定义对象:

list<A> listname;

list<A> listname(size);

list<A> listname(size,value);

list<A> listname(elselist);

list<A> listname(first, last);

<mark>l.push_back(elem);</mark>//在容器尾部加入一个元素

<mark>l.pop_back( );</mark>//删除容器中最后一个元素

<mark>l.push_front(elem);</mark>//在容器开头插入一个元素

<mark>l.pop_front();</mark>//从容器开头移除第一个元素

<mark>l.insert(pos,elem);</mark>//在pos位置插elem元素的拷贝，返回新数据的位置。

l.insert(pos,n,elem);//在pos位置插入n个elem数据，无返回值。

l.insert(pos,beg,end);//在pos位置插入[beg,end)区间的数据，无返回值。

<mark>l.clear( );</mark>//移除容器的所有数据 erase(beg,end);//删除[beg,end)区间的数据，返回下一个数据的位置。

<mark>l.erase(pos);</mark>//删除pos位置的数据，返回下一个数据的位置。

<mark>l.remove(elem);</mark>//删除容器中所有与elem值匹配的元素。

<mark>l.size();</mark>//返回容器中元素的个数

<mark>l.empty();</mark>//判断容器是否为空

l.resize(num);//重新指定容器的长度为num，
若容器变长，则以默认值填充新位置。 如果容器变短，则末尾超出容器长度的元素被删除。

l.resize(num, elem);//重新指定容器的长度为num，
若容器变长，则以elem值填充新位置。 如果容器变短，则末尾超出容器长度的元素被删除。

l.assign(beg, end);//将[beg, end)区间中的数据拷贝赋值给本身。

l.assign(n, elem);//将n个elem拷贝赋值给本身。

list& operator=(const list &lst);//重载等号操作符

l.swap(lst);//将lst与本身的元素互换

<mark>l.front();</mark>//返回第一个元素。

<mark>l.back();</mark>//返回最后一个元素。 3.6.4.6 list反转排序

<mark>l.reverse();</mark>//反转链表，比如lst包含1,3,5元素，运行此方法后，lst就包含5,3,1元素。

<mark>l.sort();</mark> //list排序

next(pos); //返回pos指向的下一个位置

<mark>list型容器不提供成员函数at()和操作符operator[ ],可以使用迭代器进行元素的访问.</mark>

<mark>list的迭代器不支持it += x或it1'-'it2这样的运算，也不支持<，<=等运算符。</mark>

### 单向链表

### 双向链表

### 跳舞链

### 跳跃表

## 栈——先进后出(FILO)

#include<stack>

q.push(x);             //将x压入栈顶

q.top();          //返回栈顶的元素

q.pop();         //删除栈顶的元素

q.size();         //返回栈中元素的个数

q.empty();            //检查栈是否为空,若为空返回true,否则返回false

### 单调栈（与单调队列相似）

<mark>栈内元素总是保持单调</mark>

当插入元素小于栈顶元素的时候，需要将栈顶的大于该元素的元素依次弹出，再插入。

```cpp
int num[MAX]={0};
stack<int> s;
scanf("%d",&n);
for(int i=1;i<=n;i++) {
    scanf("%d", &num[i]);
    while (!s.empty() && num[s.top()] < num[i]) s.pop();
    s.push(i); //一般存元素的下标
}
```

## 队列——先进先出(FIFO)

#include< queue>

queue<储存的类型> 容器名

eg: queue<int> q;

q.push() 在队尾插入一个元素

q.pop() 删除队列第一个元素

q.size() 返回队列中元素个数

q.empty() 如果队列空则返回true

q.front() 返回队列中的第一个元素

q.back() 返回队列中最后一个元素

### 双向队列

#include<deque>

deque<type> d;  type为声明的变量类型，其中d为声明的变量名

d.push_back( )  从尾部添加元素

d.push_front( )  从头部添加元素

d.pop_back( )  从尾部删除元素

d.pop_front( )  从头部删除元素

d.insert( it , n , num )  在it指针指向的元素前插入n个元素num

d.erase( it )   删除it指针指向的元素

### 单调队列 O（n）

* 一维

<mark>单调队列是一种主要用于解决滑动窗口类问题的数据结构，即，在长度为n的序列中，求每个长度为m的区间的区间最值</mark>

```cpp
deque<int> Q; // 存储的是编号!!!
for (int i = 0; i < n; ++i){
    if (!Q.empty() && i - Q.front() >= m) // 队首元素已移出框外
    Q.pop_front();
    //形成单调队列，只要比将放入元素大/小的元素就移出队列
    while (!Q.empty() && num[Q.back()] < num[i]) Q.pop_back();
    Q.push_back(i); // 新元素入队
    if (i >= m - 1) printf("%d",num[Q.front()]);
}
```

* 二维（用单调队列先后处理行和列）

a×b矩阵中的每一行可以看作长度为b的序列，我们用单调队列求出其中所有长度为n的区间的最值。对每行都如此操作，即可求出每个 1×n的长方形区域中的最大、最小整数。

这些最值又构成一个矩阵，再把这个新矩阵的每一列看作一个序列，求其中所有长度为n的区间最值。这样，可以求出原矩阵每个n×n的正方形区域中的最大、最小整数。

得到一个 (a−n+1)×(b−n+1)的矩阵，包含了所有n×n正方形区域的最大值

```cpp
deque<int> d;
int num[MAX][MAX]={0};
int fmax[MAX][MAX]={0};
int smax[MAX][MAX]={0};
for(int i=1;i<=a;i++){
    d=deque<int>();
    for(int j=1;j<=b;j++){
        scanf("%d",&num[i][j]);
        if(!d.empty() && d.front()<=j-n) d.pop_front();
        while(!d.empty() && num[i][d.back()]<num[i][j]) d.pop_back();
        d.push_back(j);
        if(j>=n) fmax[i][j-n+1]=num[i][d.front()];
    }
}

for(int i=1;i<=b-n+1;i++){
    d=deque<int>();
    for(int j=1;j<=a;j++){
        if(!d.empty() && d.front()<=j-n) d.pop_front();
        while(!d.empty() && fmax[d.back()][i]<fmax[j][i]) d.pop_back();
        d.push_back(j);
        if(j>=n) smax[j-n+1][i]=fmax[d.front()][i];
    }
}
```

### 循环队列（暂时不知道能干啥，没学）

将顺序队列的头尾相接臆造为一个环状的空间

1.     定义：

```cpp
//队列的最大空间
#define MAXSIZE 8
//队列的管理结构
struct Queue
{
    int *base; //指向队列空间的基址
    int       front; //头指针
    int       rear; //尾指针
}Queue;
```

2.     初始化

```cpp
//队列初始化
void InitQueue(Queue *Q)
{
    //为队列分配存储空间
    Q->base = (int *)malloc(int(ElemType) * MAXSIZE);
    assert(Q->base != NULL);
    //初始时队列为空，头指针和尾指针都指向0位置
    Q->front = Q->rear = 0;
}
```

3.     入队

```cpp
//入队操作
void EnQueue(Queue *Q, int x)
{
    //判断循环队列是否已满
    if(((Q->rear+1)%MAXSIZE) == Q->front)
        return;
    //队列未满，将数据入队
    Q->base[Q->rear] = x;
    //更改尾指针的指向
    Q->rear = (Q->rear+1)%MAXSIZE;
}
```

4.     出队

```cpp
//出队操作
void DeQueue(Queue *Q)
{
    //判断循环队列是否为空
    if(Q->front == Q->rear)
        return;
    //如果非空，实现可循环出队
    Q->front = (Q->front+1)%MAXSIZE;
}
```

5.     打印队列

```cpp
//打印循环队列中的数据
void ShowQueue(Queue *Q)
{
    //遍历循环队列中的元素，并将数据打印
    for(int i=Q->front; i!=Q->rear;)
    {
        printf("%d ",Q->base[i]);
        //此操作是为了实现循环遍历
        i = (i+1)%MAXSIZE;
    }
    printf("\n");
}
```

6.     获取队首元素

```cpp
//获取队头元素
void GetHdad(Queue *Q, int *v)
{
    //判断循环队列是否为空
    if(Q->front == Q->rear)
        return;
    //如果队列不为空，获取队头元素
    *v = Q->base[Q->front];
}
```

7.     队列长度

```cpp
//获取队列长度（元素个数）
int Length(Queue *Q)
{
    //计算尾指针位置与头指针位置的差距
    int len= Q->rear - Q->front;
    //如果为正数，那么len就是队列的长度；如果为负数，那么MAXSIZE+len才是队列的长度
    len = (len>0) ? len : MAXSIZE+len;
    return len;
}
```

8.     清空队列

```cpp
//清空队列
void ClearQueue(Queue *Q)
{
    //将队头指针和队尾指针都置为0
    Q->front = Q->rear = 0;
}
```

### 优先队列（我的评价是不如sort）

**优先队列自动按从大到小排序（结构体队列需重构‘<‘）**

priority_queue<****结构类型****> **队列名;**

q.size( ); //**返回q里元素个数**

q.empty( ); //**返回q是否为空，空则返回1，否则返回0**

q.push(k) ;//**在q的末尾插入k**

q.pop( ); //**删掉q的第一个元素**

q.top( ); //**返回q的第一个元素**

##### less和greater优先队列

**priority_queue <int,vector<int>,less<int>p;**

**priority_queue <int,vector<int>,greater<int>q;**

**less是从大到小，greater是从小到大**

## 并查集

1.     初始化

```cpp
int f[MAXN];
int n;
for (int i = 1; i <= n; ++i) f[i] = i;
```

2.     查找

```cpp
int find(int x)
{
    if(f[x] == x)
        return x;
    else
        return find(f[x]);
}
```

3.     合并

```cpp
void merge(int i, int j)
{
    f[find(i)] = find(j);
}
```

4.     路径压缩

<mark>减少下一次查询的时间</mark>

```cpp
int find(int x)
{
    if(x == f[x])
        return x;
    else{
        f[x] = find(f[x]);  //父节点设为根节点
        return f[x];         //返回父节点
    }
}
//以上代码常常简写为一行：

int find(int x)
{
    return x == f[x] ? x : (f[x] = find(f[x]));
}
```

### 按秩合并

1. 初始化

```cpp
for (int i = 1; i <= n; ++i)
{
    f[i] = i;
    rank[i] = 1;
}
```

2. 合并

```cpp
void merge(int i, int j)
{
    int x = find(i), y = find(j);    //先找到两个根节点
    if (rank[x] <= rank[y])
        f[x] = y;
    else
        f[y] = x;
    if (rank[x] == rank[y] && x != y)
        rank[y]++;                   //如果深度相同且根节点不同，则新的根节点的深度+1
}
```

### 带权并查集

先向上进行查找，待查找遍历完成后计算价值，最后将指向节点赋值

```cpp
int find(int a){
    if(f[a]==a) return a;
    else{
        int temp=find(f[a]);
        val[a]+=val[f[a]]; //根据题意进行计算
        f[a]=temp;
        return f[a];
    }
}
```

### 种类并查集（难点）

将每个人各分在每个种类中，A类[1,n]，B类[n+1,2n]，C类[2n+1,3n]……（几种双向关系建立几个类，eg：敌人与朋友（敌人的敌人）建立两类，A->B敌人关系,B->A敌人关系；捕食和被捕食建立三类，A->B捕食，B->C捕食，C->A捕食）建议全部开辟在一个数组里

根据题目将俩人在不同类里的元素进行合并，eg：1，2是A-B关系，将1与n+2合并（等价于2与n+1合并）

```cpp
int x=find(p[j].p1);
int y=find(p[j].p2);
if(x==y){
    f=p[j].angry;
    break;
}
hail[y]=find(p[j].p1+n);
hail[x]=find(p[j].p2+n);
```

## 树

### 二叉树

```cpp
struct node{
    char val;
    node *l,*r;
    node(){
        this->val='0';
        l= nullptr;
        r= nullptr;
    }
    node(char val){
        this->val=val;
        l= nullptr;
        r= nullptr;
    }
};
```

先序遍历（父右左）：

```cpp
void pre(node *head){
    printf("%c",head->val);
    if(head->l) pre(head->l);
    if(head->r) pre(head->r);
    return;
}
```

中序遍历（右父左）：

```cpp
void in(node *head){
    if(head->l) in(head->l);
    printf("%c",head->val);
    if(head->r) in(head->r);
    return;
}
```

后续遍历（右左父）：

```cpp
void post(node *head){
    if(head->l) post(head->l);
    if(head->r) post(head->r);
    printf("%c",head->val);
    return;
}
```

查找：

```cpp
node *find(char root,node *head){
    if(head->val==root) return head;
    node *temp= nullptr;
    if(head->l) temp=find(root,head->l);
    if(head->r && temp== nullptr) temp=find(root,head->r);
    return temp;
}
```

#### Set（STL）

#### Map（STL）

·       map: #include < map >（红黑树）

·       unordered_map: #include < unordered_map>（哈希表）

* **运行效率方面**：unordered_map最高，而map效率较低但 提供了稳定效率和有序的序列。
* **占用内存方面**：map内存占用略低，unordered_map内存占用略高,而且是线性成比例的。

map<key_type, value_type>变量名

//常用

<mark>size()     // 计算元素个数</mark>

empty()    // 判断是否为空，空返回 true

clear()    // 清空容器

<mark>erase()    // 删除元素</mark>

<mark>find()     // 查找元素</mark>

insert()   // 插入元素

<mark>count()    // 计算指定元素出现的次数</mark>

begin()    // 返回迭代器头部

end()      // 返回迭代器尾部

//非常用

swap()        // 交换两个map容器，类型需要相同

max_size()    // 容纳的最大元素个数

rbegin()      // 指向map尾部的逆向迭代器

rend()        // 指向map头部的逆向迭代器

lower_bound() // 返回键值大于等于指定元素的第一个位置

upper_bound() // 返回键值大于指定元素的第一个位置

equal_range() // 返回等于指定元素的区间

<mark>新增/修改：变量名[key] = value</mark>

#### 堆

#### 平衡二叉树

##### 线段树（区间修改+查询）

线段树每个结点代表一个区间[L,R],左子区间[L,M]，右子[M+1,R]，M=(L+R)/2。（二叉树折半查找）

* 建树：节点编号为u，左子树编号为u * 2，右子树编号u * 2+1，在遍历完左右子树后，向上更新

<mark>注：记得树和懒标记开4倍空间</mark>

* 询问：每次询问将其下所有没有处理的懒标记使用并释放，分别向左右子树递归询问（记得将超出范围的子区间剪枝），将答案累加（根据题目可改），当递归到询问范围内的区间时直接返回其节点数值

* 修改：如果修改区间完全包含当前区间，根据题意修改该节点数值，给该节点打上等于修改数值的的懒标记（编号等于节点编号），否则先释放懒标记再分别遍历左右子树（记得将超出范围的子区间剪枝），遍历完重新向上更新

* 释放懒标记：给左右子树的懒标记加上父节点的懒标记值，根据题意修改左右子树节点的值，取消标记

```cpp
void push_down(int u,int len){ //更新u的子节点
    if(add[u]){
        add[u<<1]+=add[u];
        add[(u<<1)+1]+=add[u];
        t[u<<1]+=(len-(len>>1))*add[u];
        t[(u<<1)+1]+=(len>>1)*add[u];
        add[u]=0; //取消标记
    }
    return;
}

void buildtree(int l=1,int r=n,int u=1){
    if(l==r){
        t[u]=num[l];
        return;
    }
    int mid=(l+r)/2;
    buildtree(l,mid,u<<1);
    buildtree(mid+1,r,(u<<1)+1);
    t[u]=t[u<<1]+t[(u<<1)+1];
    return;
}

long long query(int l,int r,int a=1,int b=n,int u=1){
    if(a>=l && b<=r) return t[u];
    push_down(u,b-a+1); //向下更新以防打上懒标记没有处理
    int mid=(a+b)/2;
    long long ans=0;
    if(l<=mid) ans+=query(l,r,a,mid,u<<1);
    if(r>mid) ans+=query(l,r,mid+1,b,(u<<1)+1);
    return ans;
}

void update(long long ad,int l,int r,int a=1,int b=n,int u=1){ //a,b为当前节点的区间左右两端，ad为区间修改数，lr为目标修改区间左右两端，u为节点编号
    if(a>=l && b<=r){
        t[u]+=(b-a+1)*ad;
        if(a<b) add[u]+=ad; //判断是否是子节点（即a！=b）
        return;
    }
    int mid=(a+b)/2;
    push_down(u,b-a+1);
    if(l<=mid) update(ad,l,r,a,mid,u<<1);
    if(mid<r) update(ad,l,r,mid+1,b,(u<<1)+1);
    t[u]=t[u<<1]+t[(u<<1)+1];
    return;
}
```

* 带有加法和乘法的区间修改

引入两个懒标记add，mul（mul初始值为1）

在向下更新时保证加法及乘法的修改顺序，需满足先乘mul再加add

```cpp
void push_down(int u,int len){ //更新u的子节点
    if(add[u] || mul[u]!=1){
        mul[u<<1]=(mul[u<<1]*mul[u])%M;
        mul[(u<<1)+1]=(mul[(u<<1)+1]*mul[u])%M;
        add[u<<1]=(add[u<<1]*mul[u]+add[u])%M;
        add[(u<<1)+1]=(add[(u<<1)+1]*mul[u]+add[u])%M;
        t[u<<1]=(t[u<<1]*mul[u]+(len-(len>>1))*add[u])%M;
        t[(u<<1)+1]=(t[(u<<1)+1]*mul[u]+(len>>1)*add[u])%M;
        add[u]=0; //取消标记
        mul[u]=1;
    }
    return;
}
```

* 乘法修改：

```cpp
void updatemul(long long mu,int l,int r,int a=1,int b=n,int u=1){ //a,b为当前节点的区间左右两端，ad为区间修改数，lr为目标修改区间左右两端，u为节点编号
    if(a>=l && b<=r){
        t[u]*=mu;
        t[u]%=M;
        if(a<b){
            mul[u]=(mul[u]*mu)%M; //判断是否是子节点（即a！=b）
            add[u]=(add[u]*mu)%M;
        }
        return;
    }
    int mid=(a+b)/2;
    push_down(u,b-a+1);
    if(l<=mid) updatemul(mu,l,r,a,mid,u<<1);
    if(mid<r) updatemul(mu,l,r,mid+1,b,(u<<1)+1);
    t[u]=(t[u<<1]+t[(u<<1)+1])%M;
    return;
}
```

##### 红黑树

##### 二叉搜索树

二叉搜索树：一棵二叉树，可以为空；如果不为空，满足以下性质：

1.     非空左子树的所有键值小于其根结点的键值。

2.     非空右子树的所有键值大于其根结点的键值。

3.     左、右子树都是二叉搜索树。

* 递归版：

```cpp
node *find(char val,node *head){
    if(head->val==val) return head;
    node *temp= nullptr;
    if(head->val>val && head->l) temp=find(val,head->l);
    if(head->val<val && head->r) temp=find(val,head->r);
    return temp;
}
```

* 迭代版：

```cpp
node *find(char val,node *head){
    while(head){
        if(val >head->val ) head = head->r; /*向右子树中移动，继续查找*/
        else if(val <head->val ) head = head->l; /*向左子树中移动，继续查找*/
        else return head; /*查找成功，返回结点的找到结点的地址*/
    }
    return NULL; /*查找失败*/
}
```

###### Treap树

<mark>通过随机化的 priority 属性，以及维护堆性质的过程，「打乱」了节点的插入顺序。从而让二叉搜索树达到了理想的复杂度，避免了退化成链的问题</mark>

根据优先级进行建树（根节点优先级最大），在新节点插入后根据优先级进行旋转，插入节点优先度随机使二叉树节点分布更加均匀，防止退化成链表

- 更新节点：

```cpp
//更新以root为根的子树的大小
void update(node *&root){
    root->size=root->num;
    if(root->son[0]!=NULL) root->size+=root->son[0]->size;
    if(root->son[1]!=NULL) root->size+=root->son[1]->size;
    return;
}
```

- 旋转：
  
  根据优先度等级进行旋转调整，将后插入或删除完节点后的子节点旋转到符合有限度排序的地方

```cpp
//旋转（中序遍历顺序不变） op=0为左，op=1为右
void rotate(node *&root,int op){
    node *temp=root->son[op^1]; //op^1<=>1-op;
    root->son[op^1]=temp->son[op];
    temp->son[op]=root;
    update(root);
    update(temp);
    root=temp;
    return;
}
```

- 插入（建树）：

```cpp
void insert(node *&root,int x){
    //如果该节点为空则建立该节点，并初始化
    if(root==NULL){
        root=new node();
        root->rank=rand();
        root->val=x;
        root->size=1;
        root->num=1;
        return;
    }
    root->size++; //要插入的节点必在该节点子树上，每遍历到该节点子树大小+1
    if(root->val==x) root->num++;
    else if(root->val<x){
        insert(root->son[1],x);
        if(root->rank<root->son[1]->rank) rotate(root,0); //遍历返回后检查优先度是否符合顺序，左子树大右旋，右子树大左旋
    }
    else{
        insert(root->son[0],x);
        if(root->rank<root->son[0]->rank) rotate(root,1);
    }
    update(root); //每次更新节点
    return;
}
```

- 删除节点：
  
  找到目标节点后根据左右子节点优先度进行旋转（如果有的话），直到<mark>目标节点旋转到叶节点</mark>然后将其删除

```cpp
//删除x
void del(node* &root,int x){
    if(root==NULL) return;
    if(root->val==x){
        //如果该节点数有多个减去一个
         if(root->num>1){
             root->num--;
             root->size--;
         }
         else{
             if(root->son[0]==NULL && root->son[1]==NULL ) root=NULL; //目标节点为叶节点直接删除
             else if(root->son[0]==NULL) root=root->son[1]; //仅有左/右子节点，将其直接连结
             else if(root->son[1]==NULL) root=root->son[0];
             else{
                 //左节点大向右旋递归删右节点，右节点大与之相反
                 if(root->son[0]->rank>root->son[1]->rank){
                     rotate(root,1);
                     del(root->son[1], x);
                 }
                 else{
                     rotate(root,0);
                     del(root->son[0], x);
                 }
             }
         }
    }
    //根据BST规则查找目标节点
    else if(root->val<x) del(root->son[1],x);
    else del(root->son[0],x);
    if (root) update(root); //更新遍历过的节点
    return;
}
```

- 求前驱（小于 x，且最大的数）/后继（大于 x，且最小的数）：
  
  找到x，前驱即遍历过程中最后的右子节点值（root < x），后继即遍历过程中最后的左子节点值（root > x）

```cpp
//前驱
int pre(node *&root,int x,int val=-INF){
    if(root==NULL) return val;
    if (root->val >= x) return pre(root->son[0], x, val);
    else return pre(root->son[1], x, root->val);
}

//后继
int post(node *&root,int x,int val=INF){
    if(root==NULL) return val;
    if (root->val <= x) return post(root->son[1], x, val);
    else return post(root->son[0], x, root->val);
}
```

- 查询排名：
  
  x排名即为遍历过所有小于等于x的节点（包括x自身）的左子树之和+1
  
  ```cpp
  int findrank(node *&root,int x,int val=0){
      if(root==NULL) return val+1;
      int rank = (root->son[0] == NULL) ? 0 : root->son[0]->size; //防止左子树为空出现RE报错
      if(root->val==x) return val+rank+1;
      else if(root->val<x) return findrank(root->son[1],x,val+root->num+rank);
      else return findrank(root->son[0],x,val);
  }
  ```

- 根据排名找节点：
  
  同查询排名理，只需考虑x节点自身数量导致的排名范围即可，根据小于当前节点的数（当前节点排名）比目标排名关系找到目标节点
  
  ```cpp
  int rankfind(node *&root,int x,int val=0){
      int rank = (root->son[0] == NULL) ? 0 : root->son[0]->size;
      if(rank+val+1<=x && rank+val+root->num>=x) return root->val;
      else if(rank+val+root->num<x) return rankfind(root->son[1],x,val+root->num+rank);
      else return rankfind(root->son[0],x,val);
  }
  ```

###### Splay树

**<mark>适用于经常查询和使用一个数，以及合并/分裂树,可以把某一结点向上旋转到指定位置</mark>**

每次查找都会将节点旋转到根节点，而且每次操作都会在根节点上操作，故而每次操作前都需要查找（maybe），仅将比较特殊的几个方法列出，其余同Treap树，无过多改动

**特定数据例如单调插入会退化成链导致时间复杂度暴增！！！建议用Treap树/插入时引入rank进行局部旋转**

- 旋转：
  
  与Treap树旋转原理相同（所有平衡树都是这么旋转），但是Splay树旋转实现给子节点旋转，Treap树旋转实现给父节点旋转，代码实现不太相同，所以Splay树还需要存储每个节点的父节点，用数组建树比较好完成。
  
  ```cpp
  void rotate(int x, int op) {
      int f = fa[x]; //父节点
      int gf = fa[f]; //爷爷节点
      fa[x] = gf;
      //将爷爷节点的子节点替换为x
      if (gf) {
          if (tree[gf][0] == f) tree[gf][0] = x;
          else tree[gf][1] = x;
      }
      tree[f][op ^ 1] = tree[x][op];
      if (tree[x][op]) fa[tree[x][op]] = f;
      tree[x][op] = f;
      fa[f] = x;
      update(f);
      update(x);
  }
  ```

- Splay（伸展）：
  
  将目标节点向上旋转到根节点，根据父节点及其爷爷节点（如果有的话）关系进行旋转
  
  ```cpp
  //将x旋转为goal的子节点（如果goal=0则旋转到根）
  void splay(int x,int goal){
      while(fa[x]!=goal){
          if(fa[fa[x]]==goal) rotate(x,tree[fa[x]][0]==x); //左子节点右旋，右子节点左旋，妙！
          else{
              int f=fa[x];
              int op=(tree[fa[f]][0]==f);
              if(tree[f][op]==x){ //x和父节点，爷爷节点不共线
                  rotate(x,op^1);
                  rotate(x,op);
              }
              else{               //x和父节点，爷爷节点共线
                  rotate(f,op);
                  rotate(x,op);
              }
          }
      }
      if(goal==0) root=x;
  }
  ```
* 插入
  
  同正常BST根据节点值向下搜索并插入，在每次插入完后进行一次Splay，将新节点移至根节点
  
  ```cpp
  void insert(long long x){
      if(!root){ //建立初始根
          val[++cnt]=x;
          num[cnt]=1;
          s[cnt]=1;
          fa[cnt]=0;
          root=cnt;
          return;
      }
      int now=root;
      int f=0;
      while(true){
          s[now]++;
          if(val[now]==x){
              num[now]++;
              splay(now,0);
              return;
          }
          else{
              int op=(val[now]<x);
              f=now;
              if(!tree[now][op]){
                  tree[now][op]=++cnt;
                  val[cnt]=x;
                  num[cnt]=1;
                  s[cnt]=1;
                  fa[cnt]=f;
                  splay(cnt,0);
                  return;
              }
              now=tree[now][op]; //向下查找
          }
      }
  }
  ```

* 查找
  
  为了方便根据权值查找节点序号，将查找模块单独提出，特别地在每次查找完后将目标节点旋转到根部(**注：为了满足某些题意，如果查找树中不存在的权值将会根据树返回离该值最近的节点权值，也有某些题查找不到直接返回0**)
  
  ```cpp
  int find(long long x) {
      int now=root;
      while (val[now] != x) {
          if (val[now] > x){
              if(!tree[now][0]) break;
              now = tree[now][0];
          }
          else{
              if(!tree[now][1]) break;
              now = tree[now][1];
          }
      }
      splay(now, 0);
      return now;
  }
  ```

* 删除
  
  使用Splay里合并的最常见方法，将目标节点旋转到根节点后，如果节点里数量只有一个将左右两棵子树合并即可，根据左子树所有值都小于右子树最小值，将左子树并在右子树最小值上完成删除x
  
  ```cpp
  void del(long long x){
      int k=find(x); //找到目标节点并旋转到根节点
      if(!k || num[k]==0) return;
      //如果有多个删除一个即可
      if(num[k]>1){
          num[k]--;
          s[k]--;
          return;
      }
      int less=tree[k][1];
      while(tree[less][0]) less=tree[less][0];//找到右子树的最小值
      if(less){
          splay(less,k); //转至k的子节点，即右子树的根节点
          tree[less][0]=tree[k][0]; //将k的左子树转移到右子树上
          fa[less]=0; //记得改父节点信息
          root=less;
      }
      else root=tree[k][0]; //如果右子树为空则新树直接为左子树本身
      fa[tree[k][0]]=less;
      fa[k]=0; //以防万一清除k节点信息
      tree[k][0]=0;
      tree[k][1]=0;
      return;
  }
  ```

#### 霍夫曼树

### 非二叉树

#### 字典树

#### B树

#### B+树

## 树状数组

真的是很神奇的存储方式

lowbit通过二进制进行运算，数组tree[]中存储的是后lowbit(x)个数,负数补码是原码取反后+1，与运算后本质上是求x二进制下最后一位1，tree数组存储的是部分区间和，可以类比线段树，但码量和效率都高于线段树

**注：线段树适用范围比树状数组大**

```cpp
#define lowbit(x) (x & (-x))
```

* 给第x个数加k
  
  将所有包含x的元素加上k，进行区间加的时候按差分数组b进行加减
  
  ```cpp
  void add(int x,int k){
      while(x<=n+1){
          tree[x]+=k;
          x+= lowbit(x); //真的很灵性，结合图才看明白
      }
      return;
  }
  ```

* 求1-x的和
  
  区间[x,y]的和即为sum(y)-sum(x-1)，如果存差分数组的话sum(x)即为ax的值
  
  $eg：sum[7]=tree[7]+tree[6]+tree[4],6=7-lowbit(7),4=6-lowbit(6)$
  
  ```cpp
  int Sum(int x){
      int ans=0;
      while(x>0){
          ans+=tree[x];
          x-= lowbit(x);
      }
      return ans;
  }
  ```

## ST表

## 哈希表
