---
layout: post
title: Never Create Containers of auto_ptrs
tags: c++, STL, effective STL, containers, auto_ptr
---

One more item I read from effective STL last evening was about containers of auto_ptr. I was just looking to know more about auto_ptr. I haven't used auto_ptr before. And I am glad I read this item before I could use auto_ptr.

When you assign an auto_ptr to another auto_ptr, the source auto_ptr becomes null. That seems like a nasty ptr type.

So, if you have a container of auto_ptr objects and if the container goes through some algorithm that temporarily stores value(s) from the container in some variable(s), you are going to lose those values. An example algorithm, as demonstrated in the item would be quick sort which stores values for pivot elements.

Let's write some code and see things go wrong with a container of auto_ptr objects.

-

It took a while.

The first problem I hit was that it's impossible to create a vector of auto_ptr objects. auto_ptr constructor takes a mutable reference. Whereas vector::push_back takes a const reference and then tries to create a new auto_ptr object from it, which is impossible.

After some struggle with iterators, auto_ptr, array references, comparison functions and pointers, I had working code to display the horror of auto_ptr.

The code demonstrates 3 different ways of sorting:

1. sorting a shared_ptr array using std::sort
2. sorting an auto_ptr array using qsort
3. sorting an auto_ptr array using std::sort

While #1 and #2, succeed, #3 fails with seg fault. Thus, proving that auto_ptr containers are a bad idea.

-

{% highlight cpp %}
#include <iostream>
#include <cstdlib>
#include <algorithm>
#include <memory>
#include <stdlib.h>

using namespace std;


#define SIZE 30

auto_ptr<int> ap();
void print_a(auto_ptr<int> (&a)[SIZE]);
void print_s(shared_ptr<int> (&a)[SIZE]);
int cmp(const void*, const void*);
bool cmp_a(auto_ptr<int> a, auto_ptr<int> b);
bool cmp_s(shared_ptr<int> a, shared_ptr<int> b);


int main(int argc, char* argv[]) {
	auto_ptr<int> a[SIZE];
	auto_ptr<int> a2[SIZE];
	shared_ptr<int> b[SIZE];

       	for (int i = 0; i < SIZE; i++) {
		a[i] = ap();
		a2[i] = auto_ptr<int>(new int(*a[i]));
		b[i] = shared_ptr<int>(new int(*a[i]));
	}
	
	cout << "unsorted:" << endl;
	print_a(a);

	cout << "sorted shared_ptr array using std::sort:" << endl;
	sort(begin(b), end(b), cmp_s);
	print_s(b);

	cout << "sorted auto_ptr array using qsort:" << endl;
	qsort(a, SIZE, sizeof(auto_ptr<int>), cmp);
	print_a(a);

	cout << "sorted auto_ptr array using std::sort:" << endl;
	sort(begin(a2), end(a2), cmp_a);
	print_a(a2);

	return 0;
}

auto_ptr<int> ap() {
	return auto_ptr<int>(new int(rand() % 100));
}

int cmp(const void* a, const void* b) {
	return **(auto_ptr<int>*)a - **(auto_ptr<int>*)b;
}

bool cmp_a(auto_ptr<int> a, auto_ptr<int> b) {
	return *a < *b;
}

bool cmp_s(shared_ptr<int> a, shared_ptr<int> b) {
	return *a < *b;
}

void print_a(auto_ptr<int> (&a)[SIZE]) {
	for (int i = 0; i < SIZE; i++ )
		cout << *a[i] << " ";
	cout << endl;
}

void print_s(shared_ptr<int> (&a)[SIZE]) {
	for (int i = 0; i < SIZE; i++ )
		cout << *a[i] << " ";
	cout << endl;
}
{% endhighlight %}


Output:

  unsorted:
  83 86 77 15 93 35 86 92 49 21 62 27 90 59 63 26 40 26 72 36 11 68 67 29 82 30 62 23 67 35 
  sorted shared_ptr array using std::sort:
  11 15 21 23 26 26 27 29 30 35 35 36 40 49 59 62 62 63 67 67 68 72 77 82 83 86 86 90 92 93 
  sorted auto_ptr array using qsort:
  11 15 21 23 26 26 27 29 30 35 35 36 40 49 59 62 62 63 67 67 68 72 77 82 83 86 86 90 92 93 
  sorted auto_ptr array using std::sort:
  Segmentation fault (core dumped)
  
-

The code needs to be compiled with:
-std=C++11 (for std::begin and std::end)
-w (to ignore auto_ptr deprecation warnings)

-
