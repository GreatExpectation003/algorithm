前言

1 题目描述

2 分析

2.1 关键代码

2.2 关键代码分析

3 代码

前言
详细的代码里面有自己的部分理解注释，注意该博客内容实现的高精度-除-高精度是利用前面写的高精度-减-高精度实现的的时间复杂度是O（n^2）

1 题目描述
给定两个非负整数（不含前导 0） A，B，请你计算 A/B 的商和余数。

输入格式

共两行，第一行包含整数 A，第二行包含整数 B。

输出格式

共两行，第一行输出所求的商，第二行输出所求余数。

数据范围

1≤A的长度≤100000,
1≤B的长度≤100000,
B 一定不为 0

输入样例：

9000000000000000000000000
2000000000000000000000000
AI写代码
cpp
运行
输出样例：

4
1000000000000000000000000
AI写代码
cpp
运行
2 分析
要写的是两个大整数的除法,结合前面写过的模板,应该可以用 A * B 的高精度乘法配合 A + B  的加法来实现,但是时间上肯定很高很高了(因为我不会FFT的高精度乘法)或者只用高精度的减法来实现,所以综合看来还是减法实现比较好

2.1 关键代码
//A / B ， C 是商，r 是余数
vint div2(vint &A, vint &B, vint &r) {
	vint C;
 
	if(!cmp(A,B)) {
		C.push_back(0);
		r = A;
		return C;
	}
 
	int t = 0;
//	vint temp(A.size());
	vint temp;
	for(int i = A.size() - 1; i >= 0; i -- ) {
		//相当于
		//  i  2 1 0
		//     0 1 3, 这个地方的下标是 0 1 2, 和上面不一样 
		//     _____
		//  9 /1 2 3
		//     0
		//     -----
		//     1 2
		//       9
		//     -----
		//       3 3
		//       2 7
		//     -----
		//         6
		//  temp 保存余数，每次看作两个高精度数 temp 和 B 的减法
		//  temp 值的低位在数组高方便处理，否则去掉前导 0 和控制大小比较麻烦
		//  也可能是我没想明白，大家可以改进改进
		//  C = [0,1,3] ,注意是高位在数组低位
 
		//存的是 [1,2] ,比较或者减法需要反过来
		temp.push_back(A[i]);
 
		//因为 temp 是值的高位在数组低位，所以在比较前需要反转
		reverse(temp.begin(),temp.end());
		//反转之后注意有前导 0 ，比如 12123 / 12 ，不去掉会影响结果
		while(temp.size() > 1 && temp.back() == 0) temp.pop_back();
 
		//如果 tmp 大于 B, 说明可以减, 否则不能减
		while(cmp(temp,B)) {
			//结果是值的低位在数组低位
			temp = sub(temp,B);
			t++;
		}
 
		//减完之后还需要把 tmp 反转回来, 因为 sub 返回的是值低位在数组低位
		reverse(temp.begin(),temp.end());
 
		//t 不会 > 10
		C.push_back(t);
		t = 0;
 
	}
	
	//在上面把 temp 又变成值的高位在数组低位，所以在返回前需要反转
	reverse(temp.begin(),temp.end());
	r = temp;
 
	//因为 C = [0,1,3] 因为数值的高位在数组低位 ，要保证统一，需要反转
	reverse(C.begin(),C.end());
	//反转之后，商 C 前几个位置可能会出现前导0, 需要去掉
	while(C.size() > 1 && C.back() == 0) C.pop_back();
 
	return C;
}
AI写代码
cpp
运行

2.2 关键代码分析
首先,在我的想法中,既然需要用减法实现,那就需要做替换,用哪两个数来做减法,答案肯定是用 A 的高位足够大时的那几位减掉 B 的值.

其次,此时 A 的这些高位肯定比 B 大,并且因为 B 是高精度数,所以此时 A 的足够大的高位,也就是余数必定也是高精度数,而且每次这个余数都要改变,需要用它来减掉 B ,这时候能够减的次数就是商 C 的值.然后重复前面的过程就行了.

最后,例子如 123 / 9, A = [3,2,1], B = [9],注意数值的低位在数组低位,计算过程是从高位开始算,正常除法也是这样, 刚开始 temp = A2  = [1],比 B 小,不做减法 t = 0, C = [0],然后 temp = [1,2],因为需要统一计算,必须要反转,注意反转后需要去除前导 0 ,反转后 temp 为 [2,1] 比 B 大,循环减法,在这里面 t <= 9, 然后减法完成之后, t = 1 就是商,再将 temp 反转,因为存放余数的时候,数值低位在数组高位方便处理,然后重复就可以了

3 代码
#include<iostream>
#include<vector>
#include<algorithm>
 
using namespace std;
typedef long long LL;
typedef vector<int> vint;
 
const int N = 1e5 + 10;
 
//A >= B
bool cmp(vint &A, vint &B) {
	if(A.size() != B.size())return A.size() > B.size();
	for(int i = A.size() - 1; i >= 0 ; i -- ) {
		if(A[i] != B[i])return A[i]>B[i];
	}
	return true;
}
 
//C= A - B
vint sub(vint &A,vint &B) {
	vint C;
 
	int t = 0;
	//调用前保证 A >= B
	for(int i = 0; i < A.size(); i ++ ) {
		t = A[i] - t;
		if(i < B.size()) {
			t = t - B[i];
		}
		C.push_back((t + 10) % 10);
		if(t < 0) t = 1;
		else t = 0;
	}
 
	//记得去除前导 0
	while(C.size() > 1 && C.back() == 0) {
		C.pop_back();
	}
 
	return C;
}
 
//A / B ， C 是商，r 是余数
vint div2(vint &A, vint &B, vint &r) {
	vint C;
 
	if(!cmp(A,B)) {
		C.push_back(0);
		r = A;
		return C;
	}
 
	int t = 0;
//	vint temp(A.size());
	vint temp;
	for(int i = A.size() - 1; i >= 0; i -- ) {
		//相当于
		//  i  2 1 0
		//     0 1 3, 这个地方的下标是 0 1 2, 和上面不一样 
		//     _____
		//  9 /1 2 3
		//     0
		//     -----
		//     1 2
		//       9
		//     -----
		//       3 3
		//       2 7
		//     -----
		//         6
		//  temp 保存余数，每次看作两个高精度数 temp 和 B 的减法
		//  temp 值的低位在数组高方便处理，否则去掉前导 0 和控制大小比较麻烦
		//  也可能是我没想明白，大家可以改进改进
		//  C = [0,1,3] ,注意是高位在数组低位
 
		//存的是 [1,2] ,比较或者减法需要反过来
		temp.push_back(A[i]);
 
		//因为 temp 是值的高位在数组低位，所以在比较前需要反转
		reverse(temp.begin(),temp.end());
		//反转之后注意有前导 0 ，比如 12123 / 12 ，不去掉会影响结果
		while(temp.size() > 1 && temp.back() == 0) temp.pop_back();
 
		//如果 tmp 大于 B, 说明可以减, 否则不能减
		while(cmp(temp,B)) {
			//结果是值的低位在数组低位
			temp = sub(temp,B);
			t++;
		}
 
		//减完之后还需要把 tmp 反转回来, 因为 sub 返回的是值低位在数组低位
		reverse(temp.begin(),temp.end());
 
		//t 不会 > 10
		C.push_back(t);
		t = 0;
 
	}
	
	//在上面把 temp 又变成值的高位在数组低位，所以在返回前需要反转
	reverse(temp.begin(),temp.end());
	r = temp;
 
	//因为 C = [0,1,3] 因为数值的高位在数组低位 ，要保证统一，需要反转
	reverse(C.begin(),C.end());
	//反转之后，商 C 前几个位置可能会出现前导0, 需要去掉
	while(C.size() > 1 && C.back() == 0) C.pop_back();
 
	return C;
}
 
int main() {
	string a,b;
	cin>>a>>b;//a = "123",b = 12
	vint A,B;
	//A=[6 , 5 , 4 , 3 , 2 , 1]，因为可能需要进位，个位放数组低位方便在数组高位加上进位
	for(int i = a.size() - 1 ; i >= 0 ; i --) {
		A.push_back(a[i] - '0');
	}
	for(int i = b.size() - 1 ; i >= 0 ; i --) {
		B.push_back(b[i] - '0');
	}
 
//	vint C = sub(A,B);
//
//	for(int i = C.size() - 1 ; i >= 0 ; i --) {
//		cout<<C[i];
//	}
 
//	if(b == 0) {
//		cout<<"error";
//	} else {
	vint r;
	vint C = div2(A,B,r);
 
	for(int i = C.size() - 1 ; i >= 0 ; i --) {
		cout<<C[i];
	}
	cout<<"\n";
	for(int i = r.size() - 1 ; i >= 0 ; i --) {
		cout<<r[i];
	}
	//cout<<"\n"<<r;
//	}
 
	return 0;
}
AI写代码
cpp
运行

————————————————
版权声明：本文为CSDN博主「望525」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_63306482/article/details/136017267
