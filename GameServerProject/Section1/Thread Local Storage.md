- 왜필요한가를 이해해야합니다.
- 식당예시를 다시들어봅시다
	- 화장실예시를 다시들어보자
	- 여태까지 화장실을 이상적으로 사용하는법을 배웠다!
	- ![[Pasted image 20240722223308.png]]
	- 근데 현실은 생각보다 녹록치않다.
	- 주방이나 결제프론트,서빙하는 사람 모두 락이 필요하기때문
	- 게임예시
		- 강화를하면 클라이언트 세션, 데이터베이스세션에 연관성있게 반영되어야합니다.
		- 락을 모두 배치하면 될것같지만, **한쪽으로 몰리는경우에는 그냥 망한다**
		- MMORPG 같은경우(WOW에서 몇백명의 유저들이 몰리는경우) 몇백명의 유저가 몰리는경우
			- 같은 공간에 패킷을 발사하게됩니다.
				- 게임에서 실행영역이 넓은데 반해 한곳에 몰리는 경우를 말함
			- 여기서 문제가 되는점은, 락이 많은경우 한번에 한번씩밖에 활용못함
				- 어차피 멀티스레드라 한명이 처리해도 똑같은 분량인것을 뭉쳐서 처리하기때문에 더 부하가 걸리고 **느려지는** 경우가있음
		- **따라서 무조건 Lock을 하고 다루는것이 능사는 아니다.**
			- 무조건 락걸고 대충해결하면 되는거아니야? -> 아니야!
			- Thread Local Stroage 를 배워보자

#### Thread Local Storage
- 한식예시
	- 반찬이 많을거다
	- 한그릇에 한 반찬이 있다.
	- 한그릇만 가져가서 가져놓고 하면 굉장히 비효율적임
	- 그래서큰  쟁반하나에 모아서 가져간다.

- 힙영역과 데이터영영역이 있음
- 스택같은경우 스레드마다 제각각
- 힙영역과 데이터영역은 스레드가 각각사용
![[Pasted image 20240722224023.png]]
- 모두가 따로 사용할 수 있는 전역공간이 있으면 어떨까? 라는 아이디어에서 시작
```C#
using System.Threading;
using System.Threading.Tasks;

namespace ServerCore
{
    class Program
    {
        /// <summary>
        /// /static 이기 때문에 모두가 접근가능 
        /// </summary>
        //private static string ThreadName;
        private static string ThreadName;
        //private static ThreadLocal<string> ThreadName = new ThreadLocal<string>();

        static void WhoAmI()
        {
            ThreadName= $"My Name Is {Thread.CurrentThread.ManagedThreadId}";
            //다른 스레드에게 영향을 미쳤는지 검사
            Thread.Sleep(1000);
            Console.WriteLine(ThreadName);
    

        }
        static void Main(string[] args)
        {
            //Task task... 이렇게 안만들어도됨
            Parallel.Invoke(WhoAmI,WhoAmI,WhoAmI,WhoAmI,WhoAmI,WhoAmI,WhoAmI,WhoAmI);
        }
    }
}
```

```C#
       static void WhoAmI()
        {
            // issue 계속 덮어 쓰고있음.
            ThreadName.Value= $"My Name Is {Thread.CurrentThread.ManagedThreadId}";
            //다른 스레드에게 영향을 미쳤는지 검사
            Thread.Sleep(1000);
            Console.WriteLine(ThreadName.Value);

    

        }
        static void Main(string[] args)
        {
            //Task task... 이렇게 안만들어도됨
            ThreadPool.SetMinThreads(1, 1);
            //아래의 경우 쓰레드가 끝나자마자 다른 쓰레드를 실행하기때문에 중복된 값이 발생함. 
            ThreadPool.SetMaxThreads(3, 3);
            Parallel.Invoke(
```

```C#
using System.Threading;
using System.Threading.Tasks;

namespace ServerCore
{
    class Program
    {
        /// <summary>
        /// /static 이기 때문에 모두가 접근가능 
        /// </summary>
        //private static string ThreadName;
        //static string ThreadName
        private static ThreadLocal<string> ThreadName = new ThreadLocal<string>(() =>
        {
           return  $"My Name Is {Thread.CurrentThread.ManagedThreadId}";}
        );

        static void WhoAmI()
        {
            bool repeat = ThreadName.IsValueCreated;
            if(repeat) Console.WriteLine(ThreadName.Value + ": this is repeat");
            else
            {
                Console.WriteLine(ThreadName.Value);
            }

            //다른 스레드에게 영향을 미쳤는지 검사
            Thread.Sleep(1000);
            Console.WriteLine(ThreadName.Value);

    

        }
        static void Main(string[] args)
        {
            //Task task... 이렇게 안만들어도됨
            ThreadPool.SetMinThreads(1, 1);
            //아래의 경우 쓰레드가 끝나자마자 다른 쓰레드를 실행하기때문에 중복된 값이 발생함. 
            ThreadPool.SetMaxThreads(3, 3);
            Parallel.Invoke(WhoAmI,WhoAmI,WhoAmI,WhoAmI,WhoAmI,WhoAmI,WhoAmI,WhoAmI);
        }
    }
}


```

```C#
using System.Threading;
using System.Threading.Tasks;

namespace ServerCore
{
    class Program
    {
        /// <summary>
        /// /static 이기 때문에 모두가 접근가능 
        /// </summary>
        //private static string ThreadName;
        //static string ThreadName
        private static ThreadLocal<string> ThreadName = new ThreadLocal<string>(() =>
        {
           return  $"My Name Is {Thread.CurrentThread.ManagedThreadId}";}
        );

        static void WhoAmI()
        {
            // 이미만들어진 스레드는 재활용하도록 수정된 코드 
            bool repeat = ThreadName.IsValueCreated;
            if(repeat) Console.WriteLine(ThreadName.Value + ": this is repeat");
            else
            {
                // null일떄 실행이된다. 
                Console.WriteLine(ThreadName.Value);
            }

            //다른 스레드에게 영향을 미쳤는지 검사
            Thread.Sleep(1000);
            Console.WriteLine(ThreadName.Value);

    

        }
        static void Main(string[] args)
        {
            //Task task... 이렇게 안만들어도됨
            ThreadPool.SetMinThreads(1, 1);
            //아래의 경우 쓰레드가 끝나자마자 다른 쓰레드를 실행하기때문에 중복된 값이 발생함. 
            ThreadPool.SetMaxThreads(3, 3);
            Parallel.Invoke(WhoAmI,WhoAmI,WhoAmI,WhoAmI,WhoAmI,WhoAmI,WhoAmI,WhoAmI);

            ThreadName.Dispose();
        }
    }
}


```

- 위 최종코드는 어떻게 사용,응용할것인가?
	- 1.
		- Job이라는 일감들이 Queue에 저장되어있다면
		- 뭉틍거리를 뽑아와 자기만의 공간에 넣어놓고, 락을 걸어놓지 않은다음 마음껏 뽑아쓴다
	- 2. 
		- TLS는 또한 스레드 고유한 아이디를 만들어 고유한 전역변수를 사용하고싶을때 사용하기도합니다. (사실 실제 용도의 그대로이긴)
```C#

//[JobQueue] // X 락을 계속해서 잠그는게 아니라, 한번 락할때 최대한 많이뺴오는 원리
// 
```