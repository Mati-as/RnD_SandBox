- 컴파일러 최적화 관련 복습
	- 컴파일러가 최적화를 해주는게 오히려 멀티스레드 상황에서는 독이될수가 있더라!

- 강의요약
	- 메모리베리어는 
		- 코드재배치 억제
		- 메모리 가시성에 보장됩니다. 
			- 가시성 문제란, 한 스레드에서 변경한 데이터가 다른 스레드에서 즉시 보이지 않는경우발생합니다.
			- `Thread.MemoryBarrier()`는 모든 스레드가 메모리 변경사항을 볼 수 있도록 보장합니다. 또한 이는 스레드가 메모리 장벽을 통과할 떄 자신의 캐시를 **플러시** 하고, 최신 데이터를 메모리에서 다시 읽도록 합니다. 
- 수행예제
	- 싱글스레드로 바꿔서 다시 해보기
```C#
int count 0;
		
		while(true)
		{
			x = y = r1 = r2 = 0;
		
			Task t1 = new Task(Thread_1);
			
			Task t2 = new Task(Thread_2);
			
			t1.Start();
			// x =0; y = 1, r1 = 1 , r2 = 0
			t2.Start();
			// x =1; y = 1, r1 = 1, r2 = 1
			Task.WaitAll(t1,t2); //끝날때 까지 메인스레드는 대기합니다. 
		
			if (r1==0 && r2 ==0)
			break;
		}
	}
	
		Console.WriteLine(${count}만에 빠져나옴!}")
```
```C#
using System;

class Program
{
	volatile static int x =0;
	volatile static int y =0;
	volatile static int r1 = 0;
	volatile static int r2 =0;
	
	staic void Thread_1(){
		y = 1;
		r1 = x;
	}
	static void Thread_2(){
		x  = 1;
		re2 = y;
	}
	
	static void Main(string[] args)
	{
		int count 0;
		
		while(true)
		{
			x = y = r1 = r2 = 0;
		
			Task t1 = new Task(Thread_1);
			
			Task t2 = new Task(Thread_2);
			
			t1.Start();
			// x =0; y = 1, r1 = 1 , r2 = 0
			t2.Start();
			// x =1; y = 1, r1 = 1, r2 = 1
			Task.WaitAll(t1,t2); //끝날때 까지 메인스레드는 대기합니다. 
		
			if (r1==0 && r2 ==0)
			break;
		}
	}
	
		Console.WriteLine(${count}만에 빠져나옴!}")
}
```

#### 놀랍게도 빠져나온다!
- `volatile`키워드로 최적화를 방지했는데도, 빠져나온다.. 어떻게 된일이고?
- 경우의 수를봐도  0이 될수가없다.
- **캐시** 의 문제도 아님 (나중사례에서 해당될 수 있음)
- **CPU 같은 경우, 의존성이 없는 명령어라면 순서를 뒤바꿀수 있다.** **?!?!?!?!?!?!?!?!?!?!?**
- 즉 위의 예제의경우 아래와같이 순서가 바뀔수 있다. 따라서 `r1==0 && r2 ==0`을 충족하게 되는것
![[Pasted image 20240704110038.png]]
### 어떻게 허락도 안받고 바꿀수 있단말인가?
- 사실 싱글스레드에서는 문제가 없었지만, 멀티스레드에서는 마음대로 실행버리면 예측이 꼬이게됩니다. 
```C#
	staic void Thread_1(){
		y = 1;
		////////// 순서를 강제할 수 있다면 문제가 해결될것!
		r1 = x;
	}
```

### Memory Barrer `Thread.MemoryBarrier()`

```C#
y1 = 1;
Thread.MemoryBarrier();
r1 = x;
```

- **메모리 베리어 종류**
	- 1. Full Memory Barrier (어셈블리: MFENCE, C# Thread.MemoryBarrier): Store/Load 둘다 막는다
		- store : 실제값을 할당
		- load : 값을 로드함
	- 2. Store Memory Barrier (어셈블리 : SFENCE : Store만 막습니다
	- 3. Load Memory Barrier(어셈블리: LFENCE) : Load만 막습니다. 



## 가시성
[문제상황 복습]
![[Pasted image 20240704110828.png]]

![[Pasted image 20240704110808.png]]
\

- 수첩에 동기화작업을 하고, 사용하는 사람 또한 동기화를 한다음에 커밋을 해야 정상적으로 동작할 수 있습니다.
- Thread.MemoryBarrier() : 

```C#

class Program
{
	int _answer;
	int _complete;
	
	void A()
	{
		_answer =123;
		Thread.MemoryBarrier();
		_complete =true;
		Thread.Memorybarrier();
	}
	void B()
	{
		Thread.MemoryBarrier();
		if (_complete)
		{
			Thread.MemoryBarrier();
			Console.WriteLine(_answer);
		}
	}
}
```
- 위 예제의 경우, Store작업이 연속적으로 일어나고 있기 때문에, 가시성을 위해 Thread.MemoryBarrier()를 추가로 작성합니다. 