### Release 모드로 인해 컴파일최적화가 일어나, 예기치 않은 버그가 생길 수 있다!
- 전역으로 접근할 수 있음
```C#
//동시접근시 어떤일이 발생하는지 알아가는 예제
    class Program
    {
        static void ThreadMain()
        {
            Console.WriteLine("Thread Start");
            while (_stop == false)
            {

            }
            Console.WriteLine("Thread Ended");
        }
        private static bool _stop = false;
        static void Main()
        {
            Task t = new Task(ThreadMain);
            t.Start();

            Thread.Sleep(1000); // 1초
            _stop = true;

            Console.WriteLine("Stop Called");
            Console.WriteLine("End Waiting....");
            t.Wait();
            Console.WriteLine("Successfully Ended");
        }
```
- 바로 위 예제의 잠재적 위험은 무엇인가? 
- Release 모드로 들어가면 최적화가 발생하는데 이는 종종 디버그의 어려움이 됩니다.
```C++
using System;
using System.Threading;
using System.Threading.Tasks;

namespace ServerCore
{
    //동시접근시 어떤일이 발생하는지 알아가는 예제
    class ServerC
    {

        static void ThreadMain()
        {
            Console.WriteLine("Thread Start");
            while (_stop == false)
            {

            }
            Console.WriteLine("Thread Ended");
        }
        volatile private static bool _stop = false; // 최적화를 방지하기위해, 휘발성이니까 건들지 말라는 의미
        static void Main()
        {
            Task t = new Task(ThreadMain);
            t.Start();

            Thread.Sleep(1000); // 1초
            _stop = true;

            Console.WriteLine("Stop Called");
            Console.WriteLine("End Waiting....");
            t.Wait();
            Console.WriteLine("Successfully Ended");
        }
    }

```

- 주의
	- `volatile`의 경우 C++과다르게 C#에서는 다르게동작하고, 다르게 동작합니다.
	- 메모리배리어, 아토믹같은 다른 옵션을 사용하라고 하기때문에 C# 에서는 volatie키워드를 잊어버려도 괜찮음 



