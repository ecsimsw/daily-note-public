## TCP Handshake

### 3 Way handshake

TCP의 연결 설정은 성능과 보안 측면에서 중요하다. 연결 설정 과정은 3단계로 이를 3-way handshake라고 부른다. 그 연결 과정은 다음과 같다.
 
1. 클라이언트 - [SYN:1, seq: init_c], SYN segment를 전송
2. 서버 - SYN segment를 수신하고 [SYN:1, seq:init_s, ack:init_c+1], SYN_ACK segment 전송
3. 클라이언트 - [SYN:0, seq: init_c+1, ack: init_s+1, DATA] 전송

클라이언트는 서버에 SYN 플래그가 설정된 패킷을 전송하고, 서버는 이것이 확인된다면 SYN, ACK 플래그를 표시하여 답장한다. 클라이언트는 서버의 답장을 ACK 플래그가 설정된 패킷을 전달하여 확인시킨다.

단순히 `클라이언트 - 서버 - 클라이언트가 각각 SYN, SYN_ACK, ACK를 전달한다`보다, 왜 3way가 필요한지를 알아야한다. 
연결 수립을 위해서 서버의 상태를 확인하고, 연결 준비를 요청한다를 위해선 SYN, SYN_ACK의 2way만 필요한게 아닐까는 생각을 했었다.   

서버도 자신의 패킷이 제대로 전달되는지 확인해야한다. 마지막 Client의 ACK은 서버의 SYN_ACK도 잘 전달된다를 확인하기 위함이다. 즉 양 채널의 송수신을 모두 보장하기 위한 최소 시나리오가 3way가 된다.

### 4 Way handshake

연결 종료에는 같은 원리지만 한가지 단계가 추가되어 4 way handshake 방식이 사용된다. 연결 해제를 위한 비트는 FIN을 이용한다.

1. 클라이언트 - [FIN:1, seq: fin_seq] 전송
2. 서버 - [ACK] 전송
3. 서버 - [FIN] 전송
4. 클라이언트 - [ACK] 전송 
5. 클라이언트 - 이후, ACK의 loss에 따른 FIN의 재전송에 대비해 일정 시간 동안 지연해둔다.

#### 클라이언트의 마지막 ACK가 필요한 이유 
마지막 서버에서 더이상 전달할게 없다는 의미로 FIN을 전송한다. 마지막 클라이언트의 ACK는 서버에 FIN이 제대로 전달받았음을 확인하는 패킷이다.    

만약 FIN이 유실되어 서버의 FIN을 클라이언트가 못 받는다면, 클라이언트는 지속적으로 서버의 FIN을 받기 위해서 기다릴 것이다. 때문에 ACK로 FIN 수신을 확인해줘야 서버도 그제서야 Close가 가능하다. 
반대로 클라이언트의 마지막 ACK가 없다면 서버는 일정 시간을 기다렸다가 FIN을 재전송하게 된다.

#### 클라이언트의 ACK 이후 time wait이 필요한 이유
네트워크 지연으로 서버의 FIN보다 data를 패킷이 더 늦게 전달될 수 있다. 
클라이언트가 서버의 FIN만 듣고 바로 수신 확인(ACK) 후 종료한다면 이 FIN 다음에 도착된 패킷을 놓치게 된다. 
일정 시간 동안 클라이언트의 close를 늦춰, 지연된 패킷을 받을 수 있도록 한다.
