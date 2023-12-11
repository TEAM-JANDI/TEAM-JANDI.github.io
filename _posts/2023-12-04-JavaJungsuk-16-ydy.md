---
layout: post
title: "[16챕터] 네트워크"
subtitle: 자바의 정석 스터디 26회차
categories: Java
tags: [java, 자바의정석, 네트워크, checksum]
---

TCP 헤더 중 checksum이라는 필드가 있는데, 이를 통해 데이터 무결성을 체크한다고 한다. 어떻게 체크한다는 건지 궁금하여 찾아보게 되었다.

## Checksum
- 데이터의 무결성을 체크하기 위해 존재한다.
- IPv4에서는 선택사항이고, IPv6에서는 필수이다.
- 체크섬 계산 시 포함되는 항목 : pesudo header, header + data


### pesudo header
- IP header를 참조하여 만들어지는데, 체크섬 계산에만 사용되며 패킷을 보낼때는 제외된다.
  - OSI7계층에 따르면 tcp와 ip는 계층이 다른데 어떻게 송/수신측 tcp가 모두 IP의 데이터를 참조할 수 있는지? : 
    chatgpt 曰 "OSI 7계층 모델은 참조 모델이며 실제로 프로토콜은 보다 통합된 방식으로 상호 작용하는 경우가 많습니다. 
    TCP는 OSI 모델에서 IP 위에서 작동하지만 IP 헤더의 정보를 인식하고 상호 작용합니다. 
    이러한 협력은 네트워크를 통한 데이터의 적절한 캡슐화 및 라우팅에 필수적입니다."
- 총 12바이트로, 다음과 같은 정보를 가진다.
  - source IP(4byte) : 출발지 IP
  - destination IP(4byte) : 목적지 IP
  - reserve(1byte) : 예약필드. 항상 0.
  - protocol(1byte) : protocol(TCP : 6, UDP : 17)
  - length(2byte) : (TCP/UDP) 헤더 + data 총 길이


### checksum 계산 및 오류 확인 방법
- 송신측
  1. ip header를 참조하여 pesudo header를 생성한다.
  2. header의 checksum field를 0으로 초기화한다.
  3. pesudo header, segment를 2byte의 데이터 블록( 16bit word)로 분할한 뒤 합을 구한다.
  4. 모든 데이터 블록의 합을 구한다.이때 발생하는 carry는 모두 wrap around 한다.
  5. 합을 구한 결과에 1의 보수를 취하고, 결과값을 header의 checksum field에 넣는다.
  6. 수신측에 pesudo header를 제외하고 segment(header + data)만 전송한다.
- 수신측
  1. ip header를 참조하여 pesudo header를 생성한다.
  2. 송신측으로부터 받은 segment와 수신측에서 생성한 pesudo header를 2byte의 데이터 블록으로 분할한 뒤 합을 구한다. 이때, segment의 checksum도 계산에 포함한다.
  3. 합의 결과가 1로만 구성된다면 정상, 0이 하나라도 포함되었다면 비정상으로 처리한다.  


- 계산 예시
  - https://www.youtube.com/watch?v=AtVWnyDDaDI
  - https://heegyukim.medium.com/computer-network-7-udp-86d45323d5c7


- tcp를 예로 들었으나 udp도 동일하다. segment를 datagram으로 명칭만 변경하면 된다.
- 오류 체크하는 방법은 방법이 여러 가지이거나 작성자들 마다 의견이 다른 것 같다. 내가 참고한 영상에서는 수신측에서 계산 시 checksum을 포함하여 계산하며, 계산 결과가 모두 1일 때 정상으로 처리한다고 하였다.
  그러나 일부 다른 글들에서는 수신측에서도 계산 시 checksum 필드를 0으로 초기화하고 계산하며,
  계산 후 송신측에서 보낸 checksum과 수신측에서 계산한 checksum의 값을 비교하여
  일치하면 정상, 일치하지 않으면 비정상으로 처리한다고 하기도 한다.


### 체크섬 특징
- 전송 중 데이터가 변경되더라도, 체크섬 계산 시 정상 값이 나오는 경우라면 오류 검출을 하지 못한다.
- IPv4 헤더에는 있으나, IPv6 헤더에는 없어졌다.
- IPv4에서의 UDP의 경우 checksum이 옵션이며, checksum을 사용하지 않으려면 checksum을 전부 0으로 채운다.
- cheksum을 사용하지만 계산 값이 0인 경우는, cheksum을 사용하지 않는 경우와 구분하기 위해서 전부 1로 채운다.


### 체크섬 결과가 비정상일 경우
- TCP의 경우 수신자가 체크섬을 통해 오류를 감지하면 해당 세그먼트를 삭제하고(손실처리), N-ACK를 보내거나, 응답을 아예 보내지 않는(timer 기능 활용) 등의 방법을 통해 데이터가 손실되었음을 알려 송신자가 재전송 할 수 있도록 한다.
- UDP의 경우 오류 복구나 재전송을 위한 기능이 없으므로 오류가 발생한 데이터그램을 무시한다. 경우에 따라 어플리케이션에서 오류 처리 기능을 구현할 필요가 있다.


---

참고  
