- 근성, 양보, 갑질(이벤트)
- 세가지 모두 장단점이 있음
- 세가지를 섞어서 쓰기도합니다.
```C#
static object _lock = new object();
static SpinLock _lock = new SpinLock();

static void Main(string[] args)
{
	lock(_lock)
	{
	}

bool lockTaken = false;
try 
{
}
}
```
```