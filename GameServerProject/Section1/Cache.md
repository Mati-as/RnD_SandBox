### 캐시란 무엇인가? 
![[Pasted image 20240703103351.png]]
- 주문을 받자마자 주문현황을 받기 번거롭다.
- 꾹꾹 눌러서 채워질때마다 주문현황에 기록하면 좀 더 편하지 않을까?
- 주문을 여러개 할 떄, 웨이터가 사용하는 미니수첩을 캐시에 비유할 수 있음
![[Pasted image 20240703103620.png]]
- 문제점
	- 수첩은 직원마다 공유하는게 아니라 각자 가지고 있다.
	- 근데 사이다를 기록한 직원은, 콜라를 보고 당황하게 된



![[Pasted image 20240703103636.png]]
### 컴퓨터에서도 똑같은 일이 일어난다
- 코어에는 
	- ALU, 캐시장치가 들어있습니다.
	- 이는 RAM과 통신합니다. 
	- ![[Pasted image 20240703103809.png]]
- 캐시는 알아서 동작함
- **캐시철학: 무엇을 캐싱할것인가?**
	- Temporal: 방금 주문ㅁ한 테이블에서 추가주문이 나올확률이 높다!
	- Spacial Locality: 방금 접근한 주소의 인접한곳에서 추가 주문할 확률이 높다!
- 캐시의 경우에 싱글스레드인경우에는 문제가 없었지만, 
- 멀티스레드 환경에선 문제가 될 수 있다! 
- 스레드마다 각각의 코어, 각각의 캐시가 있을텐데, 오류가 하나도 없다는 보장이 없기 때문이다!

- 캐싱에따른 속도 차이 예제
-```
``` C#

namespace ServerCore
{
    //동시접근시 어떤일이 발생하는지 알아가는 예제
    class ServerC
    {
        static void Main()
        {
            int[,] arr = new int[10000, 10000];

            {
                long now = DateTime.Now.Ticks;
                for (int y = 0; y < 10000; y++)
                {
                    for (int x = 0; x < 10000; x++)
                    {
                        arr[y, x] = 1;
                    }
                }

                long end = DateTime.Now.Ticks;
                Console.WriteLine($"(y,x) 순서로 걸린 시간 {end-now}");

            }

            {
                long now2 = DateTime.Now.Ticks;
                for (int y = 0; y < 10000; y++)
                {
                    for (int x = 0; x < 10000; x++)
                    {
                        arr[x, y] = 1;
                    }
                }
                long end2 = DateTime.Now.Ticks;
                Console.WriteLine($"(x,y) 순서로 걸린 시간 {end2 - now2}");

            }

        }
    }
```
```