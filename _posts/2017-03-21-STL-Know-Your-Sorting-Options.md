---
layout: post
title: Know Your Sorting Options
tags: c++, STL, effective STL, algorithms, sorting
---

Last evening, I read some fascinating things about sorting algorithms in C++/STL. I read an item from effective STL by Scott Meyers. Apart from tingling some humor nerves, he also stimulated some algorithmic thinking.

The sorting algorithms in STL are varied for different kinds of practical use.

1. partial_sort and nth_element for sorting only parts of a list. The difference between them is not yet very clear to me. I'll come to that later.

2. There are the standard sort and qsort for full sorting of a list. And a stable version, stable_sort.

3. partition and stable_partition to just partition a list based on some predicate.

sort, qsort and stable_sort are obvious. But, I am going try out the other two. First, I am going to take up partial_sort and nth_element. Scott tried to explain it well using the words.

partial_sort gives you the top n elements of a list in sorted order. nth_element gives you top n elements of a list, that's it. It does not bother about sorting order amongst those top n elements.

But, a detailed description of nth_element confused me. The description goes something like this:

"The elements returned before nth element always precede it in the sorting order and the elements returned after it always follow it in the sorting order."

If nth_element returns top n elements and if the above description applies, it means nth_element returns top n elements in a sorted order. That makes it the same as partial_sort.

Am I missing something here? I need to find out. The first step would be to implement it in code and test it.

-

Alright. I am done implementing and testing it.

Here's the code:

{% highlight cpp %}
#include <iostream>
#include <time.h>
#include <cstdlib>
#include <vector>
#include <algorithm>

using namespace std;


void init_rand();
vector<int> gen_list(int, int);
void test(vector<int>&, int);
void print(vector<int> const&, int);


int main(int argc, char* argv[]) {
	init_rand();
	vector<int> v = gen_list(50, 100);
	test(v, 10);

	return 0;
}

void init_rand() {
	unsigned int t = static_cast<unsigned int>(time(NULL));
	srand(t);
}

vector<int> gen_list(int n, int max) {
	vector<int> v;
	v.reserve(n);

	while(n--)
		v.push_back(rand() % max);
	return v;
}

void test(vector<int> &v, int n) {
	vector<int> v2(v);

	cout << "unsorted:" << endl;
	print(v, -1);

	partial_sort(v.begin(), v.begin() + n, v.end());
	cout << "partial_sort (" << n << "):" << endl;
	print(v, n);

	nth_element(v2.begin(), v2.begin() + n, v2.end());
	cout << "nth_element (" << n << "):" << endl;
	print(v2, n);
}

void print(vector<int> const &v, int n) {
	int i = 0;
	for(vector<int>::const_iterator it = v.begin(); it < v.end(); it++) {
		if (i == n)
			cout << "# ";
		i++;
		cout << *it << " ";
	}
	cout << endl;
}
{% endhighlight %}

And here's the output:

unsorted:
13 10 22 33 77 71 4 68 17 94 7 50 75 29 91 11 54 80 15 51 28 75 87 86 93 78 56 41 46 74 15 12 36 89 45 65 61 2 33 30 48 92 80 23 21 23 34 75 4 2 
partial_sort (10):
2 2 4 4 7 10 11 12 13 15 # 94 77 75 71 91 68 54 80 50 51 33 75 87 86 93 78 56 41 46 74 29 28 36 89 45 65 61 22 33 30 48 92 80 23 21 23 34 75 17 15 
nth_element (10):
10 2 4 2 7 4 15 13 12 11 # 15 17 21 23 22 23 29 28 30 33 33 45 36 46 34 41 56 78 71 74 93 86 87 89 75 65 61 51 80 54 48 92 80 91 75 50 77 75 94 68 

It's pretty clear how partial_sort and nth_element work and differ from each other. Now, I was back to my confusion. Looking at the output, the description in the book or my interpretation of it was wrong. I had to visit the book again.

I re-read the paragraph.

>>
nth_element sorts a range so that the element at position n (which you specify) is the one that would be there if the range had been fully sorted. In addition, when nth_element returns, none of the elements in the positions up to n follow the element at position n in the sort order, and none of the elements in positions following n precede the element at position n in the sort order.
<<

Now, that I had some output in front of me, the first line clicked. The number "15", immediately to the right of "#" is the nth element. The n is fixed. Earlier, in my head, I was interpreting n as varying from 0 to n-1 over the sorted range. That's why the confusion was there. Now, it makes sense.

When nth_element returns, none of the elements in the positions up to 10 (value: 15) follow the element at position 10 in the sort order, and none of the elements in positions following 10 (value: 15) precede the element at position 10 in the sort order.

Phew! sometimes, it is easy to get lost in words.

-

Now, let's try the remaining sorting option: partition. Just as before, I am going to write some code to test it.

-

That didn't take long. Except, for one minor doubt. Does the comparison function get called with iterator as an argument or the dereferenced value itself. Thanks to the example code for partition on cplusplus.com, that doubt was quickly resolved. The comparison function gets called with dereferenced value.

The program generates a list of random integers and partitions them into odd and even integers.

Here's the code:

{% highlight cpp %}
	#include <iostream>
	#include <time.h>
	#include <cstdlib>
	#include <vector>
	#include <algorithm>

	using namespace std;


	typedef vector<int>::const_iterator CIntIt;

	void init_rand();
	vector<int> gen_list(int, int);
	void test(vector<int>&);
	void print(vector<int> const&, const CIntIt&);


	int main(int argc, char* argv[]) {
		init_rand();
		vector<int> v = gen_list(50, 10);
		test(v);

		return 0;
	}

	void init_rand() {
		unsigned int t = static_cast<unsigned int>(time(NULL));
		srand(t);
	}

	vector<int> gen_list(int n, int max) {
		vector<int> v;
		v.reserve(n);

		while(n--)
			v.push_back(rand() % max);
		return v;
	}

	bool is_odd(int a) {
		return a % 2 == 1;
	}

	void test(vector<int> &v) {
		cout << "unpartitioned:" << endl;
		print(v, v.end());

		CIntIt p_end = partition(v.begin(), v.end(), is_odd);

		cout << "partitioned by odd / even:" << endl;
		print(v, p_end);
	}

	void print(vector<int> const &v, const CIntIt &p_end) {
		for(vector<int>::const_iterator it = v.begin(); it < v.end(); it++) {
			if (it == p_end)
				cout << endl;
			cout << *it << " ";
		}
		cout << endl;
	}
{% endhighlight %}

And here's the output:

unpartitioned:
3 2 6 9 1 0 1 8 3 7 8 5 6 5 9 2 6 4 1 7 4 1 0 0 4 3 7 3 2 5 8 5 7 6 4 8 6 7 8 9 5 6 7 3 2 8 6 8 2 7 
partitioned by odd / even:
3 7 3 9 1 7 1 5 3 7 9 5 7 5 9 7 5 5 1 7 3 1 7 3 
4 0 0 4 2 4 8 6 2 6 4 8 6 6 8 8 8 6 0 6 2 8 6 8 2 2 

An awesome facility in STL!

-

Thanks to item 31, I now have a few tools in my C++ programming toolbox for diffrent types of sorting needs.

The next time someone walks up to me with 12 mangoes and asks me to pick 5 best mangoes in descending order of quality or just 5 best mangoes or asks me to divide the mangoes between ripe and unripe, I know I can do it in an efficient manner. After all, it's summer.

-
