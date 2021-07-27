#### Submission 2

##### Lecture 6/Reference/7 

**Problem: Please list some examples of returning multiple values simultaneously from functions.**

In this case, we could use reference as function parameters and I will complete the example given in the problem.

```
// We want to compute the average, minimum and maximum simultaneously
double average (int array[], const int& length, int& min, int& max)
{
	double res;
	min = array[0];
	max = array[0];
	for(int i = 0; i < length; i++)
	{
		res += array[i] * 1.0;
		if (array[i] < min)
			min = array[i];
		if (array[i] > max)
			max = array[i];
	}
	res /= length;
	return res;
}
```

##### Lecture 6/Copy constructer/13

If we want to disable the default copy constructer generated by complier, we could pass the parameters in the form of reference or declare copy constructer. However, if we put the copy constructer in the scope of private, complier will bring up an error. In practice, if we don't want to use copy constructer, the best way is to disable it using delete, if we do have to use the copy constructer, please remember to rewrite it.

##### Lecture 6/Copy constructer/15

As mentioned above, we could disable the copy constructer in the following way.

```
class A
{
public:
	A (const A&) = delete;
	...
};
```


