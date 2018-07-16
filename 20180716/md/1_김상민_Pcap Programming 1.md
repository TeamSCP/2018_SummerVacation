# Pcap Programming  1[Libpcap] - Network



7 / 16 , 2018 Jinyan:김상민

 

Libpcap(Portable Packet Capturing Library)을 사용하여 Pcap할 수 있는 프로그램을 만들어보는 것에 대해 이야기 해보려합니다.



#### Pcap(Packet Capture)

Pcap는 네트워크상에서 돌아다니는 패킷을 들여다보는 것을 말합니다.

스위치가없는 일반적인 이더넷 환경의 라우터(호스트포함)는  내부 네트워크로 향하는 모든 패킷들을 브로드캐스팅(여러개체에 전달)하게 되고, 각각의 컴퓨터들은 자신의 인터페이스로 들어오는 패킷 중 목적지가 자신인 경우에만 받아들여 운영체제가 처리를 합니다. Pacp는 목적지에 상관없이 자신에게 전달되는 패킷을 받아 들여다보는 것을 말합니다.

Pcap 응용 - Network Traffic Monitoring, Network Debugging



#### LibPcap(Portable Pocket Capturing Library)

Libpcap는 간단하게 패킷을 캡쳐하기 위한 함수모음 입니다.

각 운영체제 벤더들이 패킷 캡쳐 도구들을 제공하고 있어 개발 등에 어려움이 있어 각 도구들의 기능을 포함하면서 시스템에 독립적인 LibPcap이 개발되었습니다. 그렇기 때문에 LibPcap은 사용자 레벨에서의 저수준의 네트워크 모니터링을 가능하게하는 인터페이스를 제공하게됩니다.

또한 Libpcap은 BPF(BSD Packet Filter-통과대역 필터)에 기초하는 필터링을 지원하고 있습니다.



----

## Libpcap Programming



#### Device&Network information API

> - int pcap_lookupnet()
>
> ```
> int pcap_lookupnet(char *device, bpf_u_int32 *netp, bpf_u_int32 *maksp, char *errbuf)
> ```
>
> Network device에 대한 Network 및 mask 번호를 반환합니다.
>
> netp에 mask  Number는 maskp에 저장됩니다.
>
> device는 pcap_lookupdev 등을 통해 얻어온 Network device 이름 입니다.
>
> 에러가 발생할경우 -1이 리턴되고 에러 내용이 errbuf에 저장됩니다.



> - char* pcap_lookupdev
>
> pcap_open_live() 와 pcap_lookupnet() 에서 사용하기 위한 Network device에  대한 포인터를 되돌려준다. 성공할 경우 "eth0" 와 같은 device name을 반환하며 실패할경우 0을 되돌려줍니다.



> - pcap_datalink
>
> ```
> int pcap_datalink(pcap_t *p)
> ```
>
> data link layer [Physical layer] 타입을 반환합니다.



#### Pcatp initalization API

> - pcap_t *pcap_open_live
>
> ```
> pcap_t *pcap_open_live(char *device, int snaplen, int promisc, int to_ms, char *ebuf)
> ```
>
> 첫번째 인자의 Network device에 대한 PCD(Packet capture descriptor-지정자)을 만들기 위한 함수입니다.
>
> 실질적인 모든일은 pcap_open_live 함수를 호출해서 만들어진 PCD를 이용합니다.
>
> snaplen은 받을수있는 패킷의 최대 크기를 나타냅니다.
>
> promisc은 1일경우 promiscuous 모드가 되고,  local network의 모든 패킷을 캡쳐하게 됩니다. 0일 경우에는 목적지가 자신인 패킷을 캡쳐하게 됩니다.
>
> to_ms는 읽기시간초과 지정을 위하여 사용되며 밀리초단위입니다.
>
> ebuf는 pcap_open_live 함수호출에 에러가 발생할경우 NULL을 리턴하고 에러내용을 ebuf에 복사합니다.



> - pcap_t *pcap_open_offline
>
> ```
> pcap_t *pcap_open_offline(char *fname, char *ebuf)
> ```
>
> fname을  가지는 파일로 부터 패킷을 읽고, fname이 "-"일 경우 stdin으로 부터 읽어들입니다.
>
> ebuf는 에러메시지를 저장하기 위해서 사용이됩니다.