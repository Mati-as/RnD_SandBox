![[Pasted image 20240724125446.png]]

```C#
using System.Net;
using System.Net.Sockets;
using System.Text;

namespace Servercore
{
    class Program
    {
        static void Main(string[] args)
        {
            // DNS : Domain Name System
            // 172.1.2.3 이런식으로 하드코딩해서 서버주소를 넣는경우,
            // 자동으로 처리가 어려워 도메인처리가 어려워 질 수 있음
            // 따라서 Domain주소를 넣어 찾게하면 관리가 쉬워짐 www.newavedog ->
            
            string host = Dns.GetHostName();
            IPHostEntry ipHost =  Dns.GetHostEntry(host);  // ip가 여러개일 수 있어 배열로 전달됨, 예를들어 구글같은경우에는 ip를 여러개로 해서 트래픽을 분산 
                                                             // 어떻게 도메인으로 123.123.이 나왔는지는 책을 봐야할듯싶다. 
            IPAddress ipAddr = ipHost.AddressList[0];
            IPEndPoint endPoint = new IPEndPoint(ipAddr, 7777); //식당주소, 식당정문,후문인지 문의 식별번호 -> 포트번호 
            // 클라이언트가 엉뚱한 포트번호로 접근하면 실패하게됨 

            
            //TCP, UDP 와 차이 구분 중요 

            //문지기
            Socket listenSocket = new Socket(endPoint.AddressFamily,SocketType.Stream,ProtocolType.Tcp);

            //error detect
            try
            {
                //문지기 교육 : listenSocket의 핸드폰에 주소를 연동시키는 작업을 합니다.
                listenSocket.Bind(endPoint);

                //영업 시작
                //backLog: 최대 대기수 : 
                listenSocket.Listen(10); // 즉 10이 추가되면 fail되는 수치. 

                while (true)
                { 
                    Console.WriteLine("Listening...");

                    //손님입장
                    Socket clientSocekt = listenSocket.Accept(); //대리인을 생성합니다. 손님과 대화하려면 소켓을 통해서 대화해야합니ㅏㄷ.

                    //뭔가를 받는다
                    byte[] recvBuff = new byte[1024];
                    int recvBytes = clientSocekt.Receive(recvBuff); //받는 양을 정항 버퍼가 필요합니다.
                    string recvData = Encoding.UTF8.GetString(recvBuff, 0, recvBytes); //"안녕하세요"를 인코딩을 어떻게 할건지 정하기 , 그다음에 버퍼어엤는 값을 스트링으로 가져온다
                                                                                       //게임의경우 string으로 처리하는건 일부임. 시작인덱스 0(중간값이 들어올수도있으니 주의), recvBtytes
                    Console.WriteLine($"[From Client]: {recvData}");


                    //전송한다
                    byte[] sendBuff = Encoding.UTF8.GetBytes("Welcome to Mati's Server!");
                    clientSocekt.Send(sendBuff);

                    //쫓아내기
                    clientSocekt.Shutdown(SocketShutdown.Both); //shutdown의 경우 듣거나 보내는것 모두를 종료한다는 의미
                    clientSocekt.Close(); // shutdown없이 이것만 해도 동작하긴함 

                }
            }
            catch (Exception e)
            {
                Console.WriteLine(e.ToString);

            }

            // 생각해보기
            /*
             * 1. 손님이 입장을 안하면 어떻게 될것인가? 
             *  - 블로킹함수라해서, accept()가 동작안하면 멈추게됨
             *   - 블로킹과 논블로킹에 대한 학습 필요
             *   - accept는 블로킹이라 무한대기가능성
             *   - Send도 블로킹 함수가 될 여지가 있음
             *   - 
             */

        }
    }
}
```

```C#
using System.Net.Sockets;
using System.Net;
using System.Text;

namespace DummyClient
{
    class Program
    {
        static void Main(string[] args)
        {
            string host = Dns.GetHostName();
            IPHostEntry ipHost = Dns.GetHostEntry(host);
            IPAddress ipAddr = ipHost.AddressList[0];
            IPEndPoint endPoint = new IPEndPoint(ipAddr, 7777); //서버랑 포트번호 통일필요

            // 휴대폰 설정
            Socket socket = new Socket(endPoint.AddressFamily, SocketType.Stream, ProtocolType.Tcp); // 사실 Tcp면 Stream이어야됨
      
            try
            {
                //문지기에게 입장문의

                socket.Connect(endPoint);
                Console.WriteLine($"Connected to {socket.RemoteEndPoint.ToString}");


                //보내기 
                byte[] sendbuff = Encoding.UTF8.GetBytes("Hello Mati!");
                int sendBytes = socket.Send(sendbuff);


                //받기
                byte[] recvBuff = new byte[1024];
                int recvBytes = socket.Receive(recvBuff);
                string recvData = Encoding.UTF8.GetString(recvBuff, 0, recvBytes);
                Console.WriteLine($"[From Server] : {recvData}");

                //나가기
                socket.Shutdown(SocketShutdown.Both); // 혹시 모르니 방어적 코딩
                socket.Close();
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.ToString());
            }



        }
    }
}
```