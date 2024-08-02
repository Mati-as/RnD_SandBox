### Question List
-  Delegate, Action , Event Difference
- what if there's infinite loop in pending logic? 
	- so it occur stack overflow? 
		- Answer is yes but theoratically
			- because there's limit in Listen(10)
			- but it can be attacked by a lot of users (intentionally)
- while(true){} ? 
	- there are multiple threads when async functions are called 
	- **race condition **can occur when main thread use different logic



### `Interlocked.Exchange` 

### Seperating Receive

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net.Sockets;
using System.Net.WebSockets;
using System.Text;
using System.Threading.Tasks;

namespace Servercore
{
    internal class Session
    {
        Socket _socket;

        public void Start(Socket socket)
        {
            _socket = socket;
            SocketAsyncEventArgs recvArgs = new SocketAsyncEventArgs();
            recvArgs.Completed += new EventHandler<SocketAsyncEventArgs>(OnRecvCompleted);

            recvArgs.SetBuffer(new byte[1024], 0, 1024); // 참고로 acceptbuffer는 listener 전용입니다. 따라서Setbuffer를 대신사용합니다.
            // recvBuffer대신 사용하게됩니다. // 어마어마하게 큰 버퍼를 보낸다음 쪼개서 받는경우도 있기에 iddex는 항상 0이 아닐 수도 있습니다.
            //recvArgs.UserToken = this; // 식별자로 구분하고싶은거나 연동하고싶은 데이터가 있는경우.

            RegisterRecv(recvArgs);
        }

        public void Send(byte[] sendBuff)
        {
            _socket.Send(sendBuff);
        }



        /// <summary>
        /// q. disconnect를 두번하게되는경우는 에러발생가능
        /// null체크도 방법이 될 수 있지만 멀티스레드에서는 에러발생 가능성 
        /// </summary>
        /// 


        int _disconnected = 0; // flag generation 
        public void Disconnect()
        {
            if(Interlocked.Exchange(ref _disconnected, 1) ==1) return ; // original value를 리턴합니다. 따라서 이미 1이라면 이미 disconnected되었다는의미 
            // 1이되었다면 disconnect 되었다는 의미
            _socket.Shutdown(SocketShutdown.Both);
            _socket.Close();
        }
        #region Network Communication  


        void RegisterRecv(SocketAsyncEventArgs args)
        {
            bool pending = _socket.ReceiveAsync(args);
            if (!pending) OnRecvCompleted(null, args);// 타이밍이 극적으로 바로 맞은경우 호출
        }


        /// <summary>
        /// 내부만 사용하기에 크게 문제될건없음
        /// </summary>
        /// <param name="sender"></p aram>
        /// <param name="args"></param>
        void OnRecvCompleted(object sender, SocketAsyncEventArgs args)
        {
            if(args.BytesTransferred > 0 && args.SocketError == SocketError.Success)
            {
                // TODO
                try
                {
                    // 패킷 분석하는 것 때문에 복잡하게 바뀔예정,, 
                    string recvData = Encoding.UTF8.GetString(args.Buffer, args.Offset, args.BytesTransferred); // In games, strings may be used; start index is 0
                    Console.WriteLine($"[From Jen Server]: {recvData}");

                    // 끝났으니 다시 실행 
                    RegisterRecv(args);
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"OnRecvCompleted Failed {ex}");
                }
             
            }
            else
            {
                // todo - disconnect
            }
        }

        #endregion
    }
}

```