- 근성, 양보, 갑질(이벤트)
- 세가지 모두 장단점이 있음
- 세가지를 섞어서 쓰기도합니다
- 컨텐츠까지 멀티스레드로만들지, 플레만 멀티스레드로 할지는 다른문제
- 심리스는 전체가 멀티스레드인경우 장점이잇음
- 바람의나라처럼 씬이 구분된 경우에는, 코어만 멀티스레드인 경우가 많음
- 
```C#

// spinlock은 상호배제가 원칙.
// [] [] []
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

```


```C#

static object _lock = new object();
static SpinLock _lock = new SpinLock();

//vip가 들어와서 모두 내보내고 독점하는 자본주의 방식 
static ReaderWriterLockSlim _lock3 = new ReaderWriterLockSlim();

static Reward GetRewardByid(int id)
{
	_lock3.EnterReadLock();
	_lock3.ExitReadLock();
	return null;
}

static void AddReward(Reward reward)
{
	_lock3.EnterReadLock();
	_lock3.ExitReadLock();
}

static void Main(string[] args)
{
	lock(_lock)
	{
	}
}
```
