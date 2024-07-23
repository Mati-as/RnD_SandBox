- 문제상황
	- 한가지 자원에 두명이 들어간상황
	- 근본적으로, 들어가고 잠그는과정이 `분리` 되어있다는 점이 원인이되며 이상황을 원척적으로 차단해야합니다.
	- 


- 멀티스레드 관련 질문으로 자주나옴
```C#
class Spinlock
{
	volatile bool _locked = flase;
	
	public void Acquire()
	{
	
		Interlocked.Exchange();
		_locked = true;
	}
	
	public void Release()
	{
		_locked = false;
	}
	
	class Program
	{
	static void Main(string[] args)
	}
	
	}
}


```


- 이벤트의경우
	- 커널단에서 실행되는 것이 큰차이