- 다른이론의 경우보다 서버프로그래밍은 다양한 이론을 잘 익혀놓아야함. 
- 안정성이슈과 해킹공격에 대한 방어를 충분하게해야되기 때문
- 따라서 최소한의 네트워크 지식이 필요함

- 다른이론의 경우보다 서버프로그래밍은 다양한 이론을 잘 익혀놓아야함. - 안정성이슈과 해킹공격에 대한 방어를 충분하게해야되기 때문  
- 따라서 최소한의 네트워크 지식이 필요함  
    ### 택배를 배달하는 방식과 굉장히 유사함  
  
- 거주환경이 아니라, ** 택배를 보낼때 어떤식으로 처리할건지**에 대한 관심사와 비슷합니다.  
- 보내는 사람과 받는사람 - 경비에게 보내는경우의 문제점  
    - 주소체계가 있다고 가정할 경우 
![[Pasted image 20240723181558.png]]![[Pasted image 20240723181603.png]]
![[Pasted image 20240723181724.png]]

![[Pasted image 20240723181737.png]]


- 내부체계에서만 사용하는 용어를 사용할 수도있음
	- 저팔계 손오공 등등
	- 다른 아파트에서는 통용될 수없음에 주의
- 다른단지에게 보낸다고 가정하기 
- ![[Pasted image 20240723181922.png]]


#### 네트워크의 세계
![[Pasted image 20240723181947.png]]
- 라우터
	- 
- 스위치
- 단말기
	- 핸드폰, PC
	- 서버도 PC에서 돌아감 
		- 다만 MMORPG 같은경우는 성능이 좋은 여러컴퓨터가 필요해서 그런 머신이 필요할수있으나, 우리가 사용하는 컴퓨터와 크게 다르다고 할 수 없음

![[Pasted image 20240723182030.png]]
- 동일한 네트워크에 있냐 없냐에따라 전달방식이 달라짐
![[Pasted image 20240723182238.png]]- 같은 네트워크에 있는경우
	- 스위치가 같은 주소체계에 있다는것을 판다합니다.
	- 따라서 라우터까지 가서 처리해달라고 요청하지 않습니다.
	- 1->4 로 간다고 가정할때, 
		- 4번인 컴퓨터가 누군지 모를때 모든 컴퓨터에게 물어봄
		- 스위치도 단기기억장치가 있음(캐싱과비슷?)
			- 네모박스안을 기억해놓음
		- 참고로 외부에서 접근이 어려운 내부에서만 사용하는 사설주소일수있음
			- 예시에서 손오공,저팔계 이런느낌

![[Pasted image 20240723182759.png]]
- 다른 네트워크에 있는경우 
	- 하나의 고유주소에만 보낼수잇도록함