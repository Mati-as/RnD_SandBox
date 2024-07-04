
### 이전시간: 하드웨어 최적화, 메모리배리어

- 컴파일러나, 하드웨어 최적화문제는 크게 신경을 안써도되는게, 나중에 락이나 아토믹같은 우아한 기능들이 많이 있습니다.
- 공유변수 접근에 있어서의 다른 문제점을 알아봅시다


### 공유변수 접근의 있어서 문제점 2
```C#
class Program
{
    static volatile int number = 0;

    static void Thread_1()
    {
        for (int i = 0; i < 10000; i++)
            number++;
    }
    static void Thread_2()
    {
        {
            for (int i = 0; i < 10000; i++)
                number--;
        }
    }

    static void Main()
    {
        Task t1 = new Task(Thread_1);
        Task t2 = new Task(Thread_2);
        t1.Start();
        t2.Start();
        Task.WaitAll(t1, t2);
        Console.WriteLine(number);

    }
}
```
그러나 숫자를 올리면..?
![[Pasted image 20240704125139.png]]


### 경합조건 (Race Condition)
- 예시
	- 신입직원을 3명 더 채용했다.
	- 직원은 다들 열정적이다.
	- 콜라를 주문받았다
	- 근데 세명의 직원 전부다 열정적이어서 콜라를동시에 집었다
	- 주문받은 손님은 콜라 세개를 받았다
	- **이런 조건을 레이스 컨디션이라고 합니다**.
-![[Pasted image 20240704125323.png]]


그럼 위 코드에서 왜 경합과정이 일어났을까?

```C#
class Program
{
    static volatile int number = 0;

    static void Thread_1()
    {
        for (int i = 0; i < 100000; i++)
            number++;
    }
    static void Thread_2()
    {
        {
            for (int i = 0; i < 100000; i++)
                number--;
        }
    }

    static void Main()
    {
        Task t1 = new Task(Thread_1);
        Task t2 = new Task(Thread_2);
        t1.Start();
        t2.Start();
        Task.WaitAll(t1, t2);
        Console.WriteLine(number);

    }
}
```
#### number ++의 원리
디세어셈블리를 해서 보면...
- mov ecx,dword ptr []
- inc ecx
- mov dword ptr
- 이렇게 세단계로 일어납니다. 
**![[Pasted image 20240704125544.png]]**

그럼 실제 일어나는 과정과 비슷하게 코드를 재구성 해봅시다.
	- 원자
```C#

```C#
class Program
{
    static volatile int number = 0;

	//원자성 (atomic) 이 없다는 얘기입니다.  
    static void Thread_1()
    {
        for (int i = 0; i < 100000; i++)
		{
			int temp =number; //0
			temp += 1; //1
			number = temp;
		}
    }
    static void Thread_2()
    {
        
		for (int i = 0; i < 100000; i++)
		{
			int temp =number; //0
			temp -= 1;//-1
			number = temp; // -1 <<<< 여기서 thread1 의 넘버가 언제 실행될지 모른다!
		}
    }

    static void Main()
    {
        Task t1 = new Task(Thread_1);
        Task t2 = new Task(Thread_2);
        t1.Start();
        t2.Start();
        Task.WaitAll(t1, t2);
        Console.WriteLine(number);

    }
}
```


# Interlocked 사용
	- `volatile` 키워드 사용 불필요
	- interact 계열은 원자성을 보장합니다
		- 실행이 되거나 안되거나 둘중 하나만 가능하다는게 원자성의 정의입니다.
		- number ++의 과정의 원자성을 보장합니다.
	- 성능이 지나치게 내려갈 수 있다는 단점이 있습니다.
	- 사실 이렇게만 코드 작성하면 캐시의 개념은 의미가 없을 수 있습니다.
		- 작업이 끝나야만 하기때문에,다른 메모리에 접근하는게 큰의미가 없기 때문
		- 
```C#

using System.Threading;

class Program
{
    static volatile int number = 0;

    static void Thread_1()
    {
        for (int i = 0; i < 100000; i++)
        {
            Interlocked.Increment(ref number);
        }
     
    }
    static void Thread_2()
    {

        for (int i = 0; i < 100000; i++)
        {
            Interlocked.Decrement(ref number);
        }

    }

    static void Main()
    {
        Task t1 = new Task(Thread_1);
        Task t2 = new Task(Thread_2);
        t1.Start();
        t2.Start();
        Task.WaitAll(t1, t2);
        Console.WriteLine(number);

    }
}
```

![[Pasted image 20240704130701.png]]

#### 여담) 원자성은 굉장히 중요하다
- 검을 구매한다고 가정하자
	- 골드 -= 100;
		- //여기서 서버다운 나면 검이 추가가 안됨. 
	- 인벤 += 검;
	- 위작업은 반드시 원자적으로 일어나야합니다.
- 예를들면
	- 집행검  USER2 인벤에 넣어라 
		- // 서버다운이 나면 USER1인벤에서 안없어짐
	- 집행검을 USER1 인벤에서 없애라


![[Pasted image 20240704131054.png]]
- 먼저 도착한 직원만 콜라를 가져올수있음
- 나머지는 늦게오면 콜라를 못가져갑니다.


### interlock부연설명
- ref가 아닌경우 말이 안되는것을 이해해야합니다.
```C# 

Interlocked.Increment(ref number);

```