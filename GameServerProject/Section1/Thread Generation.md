- C#에서는 스레드를 사실상 직접적으로 관리할 일은 드뭅니다.
	- 디테일한 제어는 닷넷프레임워크에서 관리하기떄문입니다. 
``` C#
using System;
using System.Threading;

namespace ServerCore
{
    class Program
    {
        static void MainThread(object state)
        {
            for (int i = 0; i < 5; i++)
            {
                Console.WriteLine("Hello Thread");
            }
        }
        static void Main(string[] args)
        {
            ThreadPool.QueueUserWorkItem(MainThread);//세세하 인력 관리는 닷넷프레임워크에서 하게됩니다.
            Thread t = new Thread(MainThread);
            t.Name = "First Clerk";
            t.IsBackground = true;  
            // isBackground인 경우 메인이 종료되면 종료됩니다. //C++과 다르게 기본적으로 isBackground가 false입니다. 
            t.Start();// 일해!


            Console.WriteLine("Waiting for Thread"); 


            t.Join(); // 끝날때 까지 기다렸다가 뒤에 구문을 실행하기 
            Console.WriteLine("Hello World");
        }
    }
}
```


```C#
using System;
using System.Threading;

namespace ServerCore
{
    class Program
    {
        static void MainThread(object state)
        {
            for (int i = 0; i < 5; i++)
            {
                Console.WriteLine("Hello Thread");
            }
        }
        static void Main(string[] args)
        {
            ThreadPool.QueueUserWorkItem(MainThread);
             // isBackground가 기본적으로 true입니다.
              // 인력이 떠나고 돌아와야 다시 일할수 있음. 무조건장점이 되는 건 아니기 때문에 주의합니다. (따라서, 작은 일감을 던져주는것이 일반적임)

            for (int i = 0; i < 100; i++)
            {
                Thread t = new Thread(MainThread); // 코어가 스레드를 교환하는 작업이 오히려 더 길어지는 경우가 있음. 따라서 불필요하고 문제가 되는 작업일 확률이 높다 
                t.IsBackground = true;
                t.Start();
            }
            /*
             * new thread의 경우 정직원 같은 개념
             * ThreadPool같은경우는 알아서 실행하고 대기소에서 기다리는 개념
             * 다시말해, 좀 더 유동적이고 깔끔한 방법을 ThreadPool이라고 할 수 있다.
             * 유니티에서 오브젝트 풀링이라는 개념과 같이, 필요할때마다 꺼내는 개념이 비슷한 부분이라고 할 수 있습니다.
             */
            Console.WriteLine("over");

        }
    }
}
```

```C++ 

namespace ServerCore
{
    class Program
    {
        static void MainThread(object state)
        {
              for (int i = 0; i < 5; i++)
              {
                Console.WriteLine($"Thread In MainThread");
              }

        }
        static void Main()
        {
            ThreadPool.SetMinThreads(10, 10);
            ThreadPool.SetMaxThreads(20, 20);

            Console.WriteLine($"Thread");
            // - 스레드풀이 먹통이 되는것을 직접보는 실습

            for (int i = 0; i < 5; i++)
            {
                // 직원이 하는 일감의 단위를 Task에 비유할 수 있습니디ㅏ.
                Task t = new Task(() =>
	                { while (true) { } }, TaskCreationOptions.LongRunning);  // 오래걸리
-
                t.Start();
            }
            ThreadPool.QueueUserWorkItem(MainThread);
        }
    }
}
```