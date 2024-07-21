# 算法&数据结构模板



## 算法

### 高精度

#### 函数高精度

##### 高精+高精

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



##### 高精-高精

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

##### 高精*低精

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



##### 高精/低精

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



##### 高精*高精

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



##### 高精/高精

用减法模拟除法，对被除数的每一位都减去除数，一直减到当前位置的数字（包含前面的余数）小于除数

#### 重构高精度

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



#### 压位数组

### 排序

#### 冒泡排序 O（ ）

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



#### 选择排序 O（ ）

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



#### 插入排序 O（ ）

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



#### 快速排序 O（n ）

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



#### 归并排序 O（n ）

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



### 搜索

#### 二分查找

#### 折半搜索

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

#### DFS

##### 回溯

##### 剪枝

#### BFS

#### Dijkstra

#### A*

#### IDA*

#### 双向搜索

### 贪心

### 模拟

### 动态规划dp

#### 记忆化搜索

#### 线性dp

##### 前缀和

###### 一维前缀和

将1-i的值累加并存储，相减可以求区间和（有需要可取模）

```cpp
for(int i=1;i<=n;i++){
    scanf("%d",&num[i]);
    num[i]=(num[i]+num[i-1])%mod;
}

```

###### 二维前缀和

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

##### 差分

<mark>差分主要用于让一个序列某一特定范围内的所有值都加上或减去一个常数</mark>

###### 一维差分：

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

###### 二维差分：

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

#### 背包dp

#### 区间dp

#### 树形dp

在DFS遍历树时通过之前节点（从叶节点开始）或者区间的值dp出当前节点的值,即在遍历结束返回时进行dp

#### 状态压缩dp

#### 数位dp（处理数字位数或者个别数字出现）

构建dp[i][j]，i表示当前位数，j表示当前首位数字，根据题意构建dp[i][j]与dp[i-1][k]的关系（0<k<10）

#### 单调队列优化的dp

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

### 字符串

#### 哈希

#### KMP（高效寻找已给出的重复子串）

对已给参考字符串进行预处理，求出next数组即如果在对比时下一位对不上号返回的字符串下标，在比对时直接将字符串指针移至参考字符串j位置。

Next数组：1.根据定义，不断延参考字符串向后遍历已有next数组，如果指向字符与该字符不相同，则继续沿着指向字符的next继续判断，直至指针指向字符与该字符相同（最大公共前后缀）或者指向0（无法匹配），将next下一位赋上指针的下一位（j+1）或者0（从哪一位继续判断）

             2.本质上，next[i]为参考字符串前i位字符子串的最大公共前后缀的长度

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

##### 前缀函数s[i]

定义：S的子串S’（0，i）在有一对相等的前缀和后缀，是s[i]为其长度 eg：”abaabca”子串”abbab”有相等前缀后缀“ab“长度为2，s[5]=2

计算：1. 相邻的前缀函数值至多增加1

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

#### Trie树

#### AC自动机

#### 回文自动机（Manacher算法）

#### 后缀数组

#### 后缀自动机

### 离散化

<mark>把无限空间中有限的个体映射到有限的空间中去，以此提高算法的时空效率</mark>

1.      **原来相等的值离散化后仍然相等**。那么在对原序列进行**排序**后，进行**去重操作**即可

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

2.      **原来相等的值，离散化后不相等。**那不进行去重操作就好了，但是要新增一个id属性

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

### 数学

#### 位运算

#### 快速幂

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

#### 素数

```cpp
#include <cmath>
bool isPrime(int n){
    for(int i=2;i<=sqrt(n);i++){
        if(n%i==0) return false;
    }
    return true;
}

```

#### 最大公约数（GCD）



```cpp
int gcd(int a, int b) {
    if (b == 0)  return a;
    return gcd(b, a % b);
}

```

#### 费马小定理：

**如果p是一个质数，而整数a不是p的倍数，则有a^（p-1）≡1（mod p）可以通过a^（p-2）代替1/a**

#### 同余方程

##### 中国剩余定理

##### 线性同余方程

##### 高次同余方程

#### 博弈

#### 排列组合

#### 线代

##### 矩阵

#### 多项式

#### 筛法

### 图论

#### 最小生成树

##### Prim算法（对点贪心，适用于稠密图）

##### Kruskal算法（对边贪心，适用于稀疏图）

预处理：将并查集初始化

<mark>1.     对边值从小到大排序 sort()</mark>

<mark>2.     将边的始终点放入并查集(重复的即x==y说明如果加上该边形成了一个环，放弃该边)，遍历所有边直至结束</mark>

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

#### 链式前向星(如果不是树感觉不如开vector数组)

head[u]：表示以u作为起点的第一条边的编号。

next[cnt]：表示编号为cnt的边的下一条边，这条边与cnt同一个起点。

to[cnt]：表示编号为cnt的边的终点。

初始时，令 head[u]=cnt=0，表示尚未有边，每个点都是孤立点。

当加入一条新的边e(u,v)时：

①    cnt++，表示这条边的编号。

②     next[cnt]=head[u]：将原来以u作为起点的第一条边，作为该边的后续边。

③     to[cnt]=v：当前边的终点设置。

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

#### 最短路问题

##### Dijkstra算法

##### Floyd算法

##### SPFA/队列优化 Bellman-Ford算法（处理负边的权值，但时间复杂度过高）

判断负环：

1. 开始算法前，调用拓扑排序进行判断（一般不采用，浪费时间）

<mark>2.如果某个点进入队列的次数超过N次则存在负环（N为图的顶点数）</mark>

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
        for(int i=head[p];i!=0;i=Next[i]){
            int t=to[i];
            if(val[i]+dis[p]<dis[t])
		//if(val[i]+dis[p]>dis[t])
		{
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

#### 最近公共祖先（LCA）

1.    <mark> 预处理：计算log2[MAX]用于倍增</mark>

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

#### 树上启发式合并

#### 差分约束

通过链式前向星建有向图，再建立超级源点方便计算，使用[SPFA](#_SPFA/队列优化_Bellman-Ford算法（处理负边的权值，但时)判断是否负环并查找最短路，最短路径即为方程组的一组解（如果存在负环，最短路无解，则原不等式组也无解。）

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

#### 二分图

### 杂项

#### 双指针

#### 离散化

#### 约瑟夫问题

#### 珂朵莉树

#### 模拟退火

#### 莫队

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

##### 树上莫队

## 数据结构

### Vector容器

#include<vector>;

一、vector 的初始化：

    (1)
vector<int> a(10); //定义了10个整型元素的向量（尖括号中为元素类型名，它可以是任何合法的数据类型），但没有给出初值，其值是不确定的。

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

（3）copy(a.begin( ),a.end(
),b.begin( )+1); //把a中的从a.begin()（包括它）到a.end()（不包括它）的元素复制到b中，从b.begin()+1的位置（包括它）开始复制，覆盖掉原有元素

（4）<mark>find(a.begin( ),a.end( ),10);</mark> //在a中的从a.begin()（包括它）到a.end()（不包括它）的元素中查找10，若存在返回其在向量中的位置

### 链表（对中间元素操作不影响其他）

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

#### 单向链表

#### 双向链表

#### 跳舞链

#### 跳跃表

### 栈——先进后出(FILO)

#include<stack>

q.push(x);             //将x压入栈顶

q.top();          //返回栈顶的元素

q.pop();         //删除栈顶的元素

q.size();         //返回栈中元素的个数

q.empty();            //检查栈是否为空,若为空返回true,否则返回false

#### 单调栈（与单调队列相似）

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

### 队列——先进先出(FIFO)

#include< queue>

queue<储存的类型> 容器名

eg: queue<int> q;

q.push() 在队尾插入一个元素

q.pop() 删除队列第一个元素

q.size() 返回队列中元素个数

q.empty() 如果队列空则返回true

q.front() 返回队列中的第一个元素

q.back() 返回队列中最后一个元素

#### 双向队列

#include<deque>

deque<type> d;  type为声明的变量类型，其中d为声明的变量名

d.push_back( )  从尾部添加元素

d.push_front( )  从头部添加元素

d.pop_back( )  从尾部删除元素

d.pop_front( )  从头部删除元素

d.insert( it , n , num )  在it指针指向的元素前插入n个元素num

d.erase( it )   删除it指针指向的元素

#### 单调队列 O（n）

##### 一维

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

##### 二维（用单调队列先后处理行和列）

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

#### 循环队列（暂时不知道能干啥，没学）

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

#### 优先队列（我的评价是不如sort）

**优先队列自动按从大到小排序（结构体队列需重构‘<‘）**

**priority_queue<****结构类型****>** **队列名****;**

**q.size( ); //****返回q里元素个数**

**q.empty( ); //****返回q是否为空，空则返回1，否则返回0**

**q.push(k) ;//****在q的末尾插入k**

**q.pop( ); //****删掉q的第一个元素**

**q.top( ); //****返回q的第一个元素**

##### less和greater优先队列

**priority_queue <int,vector<int>,less<int>

> p;**

**priority_queue <int,vector<int>,greater<int>

> q;**

**less****是从大到小，****greater****是从小到大**

### 并查集

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

#### 按秩合并

1.初始化

```cpp
for (int i = 1; i <= n; ++i)
{
    f[i] = i;
    rank[i] = 1;
}

```

2.合并

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

#### 带权并查集

#### 种类并查集（难点）

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

### 树

#### 二叉树

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

##### 二叉搜索树

二叉搜索树：一棵二叉树，可以为空；如果不为空，满足以下性质：

1.     非空左子树的所有键值小于其根结点的键值。

2.     非空右子树的所有键值大于其根结点的键值。

3.     左、右子树都是二叉搜索树。

- 递归版：

```cpp
node *find(char val,node *head){
    if(head->val==val) return head;
    node *temp= nullptr;
    if(head->val>val && head->l) temp=find(val,head->l);
    if(head->val<val && head->r) temp=find(val,head->r);
    return temp;
}

```

- 迭代版：

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

##### Set（STL）

##### Map（STL）

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

##### 堆

##### 平衡二叉树

###### 线段树（区间修改+查询）

线段树每个结点代表一个区间[L,R],左子区间[L,M]，右子[M+1,R]，M=(L+R)/2。（二叉树折半查找）

建树：节点编号为u，左子树编号为u*2，右子树编号u*2+1，在遍历完左右子树后，向上更新

<mark>注：记得树和懒标记开4倍空间</mark>

询问：每次询问将其下所有没有处理的懒标记使用并释放，分别向左右子树递归询问（记得将超出范围的子区间剪枝），将答案累加（根据题目可改），当递归到询问范围内的区间时直接返回其节点数值

修改：如果修改区间完全包含当前区间，根据题意修改该节点数值，给该节点打上等于修改数值的的懒标记（编号等于节点编号），否则先释放懒标记再分别遍历左右子树（记得将超出范围的子区间剪枝），遍历完重新向上更新

释放懒标记：给左右子树的懒标记加上父节点的懒标记值，根据题意修改左右子树节点的值，取消标记

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

带有加法和乘法的区间修改

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

乘法修改：

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

###### 红黑树

###### Treap树

<mark>通过随机化的 priority 属性，以及维护堆性质的过程，「打乱」了节点的插入顺序。从而让二叉搜索树达到了理想的复杂度，避免了退化成链的问题</mark>

根据优先级进行建树（根节点优先级最大），在新节点插入后根据优先级进行旋转，插入节点优先度随机使二叉树节点分布更加均匀，防止退化成链表

更新节点：

```cpp
//更新以root为根的子树的大小
void update(node *&root){
    root->size=root->num;
    if(root->son[0]!=NULL) root->size+=root->son[0]->size;
    if(root->son[1]!=NULL) root->size+=root->son[1]->size;
    return;
}

```

旋转：

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

插入（建树）：

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

###### AVL树

##### 霍夫曼树

#### 非二叉树

##### 字典树

##### B树

##### B+树

### 树状数组

### ST表

### 哈希表


