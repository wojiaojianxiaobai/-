1.  **RTSP介绍：**

RTSP是应用层协议，负责一次通讯过程的建立、控制（对流媒体提供了诸如暂停，快进等控制）和结束，本身不传输数据，服务器端可以自行选择使用TCP或UDP来传送串流内容。并不特别强调时间同步，所以比较能容忍网络延迟。而且允许同时多个串流需求控制（Multicast），除了可以降低服务器端的网络用量，还可以支持多方视频会议（Video 
onference）。RTSP具有重新导向功能，根据负载情况来转换提供服务的服务器，避免过大的负载集中于同一服务器而造成延迟。

RTSP和RTP的关系：

![](https://github.com/wojiaojianxiaobai/-/blob/master/RTSP1.png)

1.  **定义的方法**

OPTIONS:客户端得到服务器提供的可用方法

DESCRIBE：客户端向服务端得到会话的描述信息（SDP）

SETUP：客户端请求与服务端建立会话，并确定传输模式

PALY：客户端发送播放请求

TEARDOWN：客户端发起关闭请求

PAUSE：客户端向服务端发送暂停请求。

SCALE

GET\_PARAMETER：用来测试客户端与服务器的连通情况。

SET\_PARAMETER：用来测试用户与服务器的连通情况。

REDIRECT

1.  **传输过程：**

一次基本的RTSP操作过程是:首先，客户端连接到流服务器并发送一个RTSP描述命令（DESCRIBE）。流服务器通过一个SDP描述来进行反馈，反馈信息包括流数量、媒体类型等信息。客户端再分析该SDP描述，并为会话中的每一个流发送一个RTSP建立命令(SETUP)，RTSP建立命令告诉服务器客户端用于接收媒体数据的端口。流媒体连接建立完成后，客户端发送一个播放命令(PLAY)，服务器就开始在UDP上传送媒体流（RTP包）到客户端。
在播放过程中客户端还可以向服务器发送命令来控制快进、快退和暂停等。最后，客户端可发送一个终止命令(TERADOWN)来结束流媒体会话

![](https://github.com/wojiaojianxiaobai/-/blob/master/RTSP2.png)

完整流程图

1.  **方法详细：**

**4.1 OPTIONS：**

用于得到服务器提供的方法

C-\>S:

*OPTIONS rtsp://192.168.20.136:5000/xxx666 RTSP/1.0*

*CSeq: 1*

S-\>C：

*RTSP/1.0 200 OK*

*CSeq: 1        //每个回应消息的cseq数值和请求消息的cseq相对应*

*Public: OPTIONS, DESCRIBE, SETUP, TEARDOWN, PLAY, PAUSE
//在Public中回应提供的方法*

**4.2 DESCRIBE：**

用于得到URI所指定的媒体描述信息，一般是SDP信息

C-\>S：

*DESCRIBE rtsp://server.example.com/fizzle/fooRTSP/1.0*

*CSeq: 312*

*Accept: application/sdp, application/rtsl,application/mheg)
//指定客户端可以接收的媒体信息类型*

*S-\>C：*

*RTSP/1.0 200 OK*

*CSeq: 312*

*Date: 23 Jan 1997 15:35:06 GMT*

*Content-Type: application/sdp  //表示回应为SDP信息*

*Content-Length: 376*

*//这里为一个空行*

*//以下为具体的SDP信息*

*v=0 *

*o=mhandley 2890844526 2890842807 IN IP4 126.16.64.4*

*s=SDP Seminar*

*i=A Seminar on the session description protocol*

*u=http://www.cs.ucl.ac.uk/staff/M.Handley/sdp.03.ps*

*e=mjh\@isi.edu (Mark Handley)*

*c=IN IP4 224.2.17.12/127*

*t=2873397496 2873404696*

*a=recvonly*

*m=audio 3456 RTP/AVP 0*

*m=video 2232 RTP/AVP 31*

*m=whiteboard 32416 UDP WB*

*a=orient:portrait*

**4.3 SETUP:**

确定传输机制，建立RTSP会话

*C-\>S:*

*SETUP rtsp://example.com/foo/bar/baz.rm RTSP/1.0*

*CSeq: 302*

*Transport: RTP/AVP;unicast;client\_port=4588-4589
//指定客户端可以接受的数据传输参数，包含传输协议、端口地址、TTL等信息*

*S-\>C:*

*RTSP/1.0 200 OK*

*CSeq: 302*

*Date: 23 Jan 1997 15:35:06 GMT*

*Session:
47112344 //产生一个SessionID，标识一个RTSP会话，客户端后续操作都要包含*

*Transport: RTP/AVP;unicast;client\_port=4588-4589;server\_port=6256-6257
//由服务器选择输出的参数*

**4.4 PLAY**

*在收到SETUP请求成功应答后，告知服务器通过SETUP中指定的机制开始发送数据。*

PLAY请求将正常播放时间（normal play
time）定位到指定范围的起始处，并且传输数据流直到播放范围结束。PLAY请求可能被管道化（pipelined），即放入队列中（queued）；服务器必须将PLAY请求放到队列中有序执行。也就是说，后一个PLAY请求需要等待前一个PLAY请求完成才能得到执行。

比如，在下例中，不管到达的两个PLAY请求之间有多紧凑，服务器首先play第10到15秒，然后立即第20到25秒，最后是第30秒直到结束。

*C-\>S:*

*PLAY rtsp://audio.example.com/audio RTSP/1.0*

*CSeq: 835*

*Session: 12345678*

*Range: npt=10-15 //指定一个时间范围，可以使用SMPTE、NTP或clock单元*

 

*C-\>S:*

*PLAY rtsp://audio.example.com/audio RTSP/1.0*

*CSeq: 836*

*Session: 12345678*

*Range: npt=20-25*

 

*C-\>S:*

*PLAY rtsp://audio.example.com/audio RTSP/1.0*

*CSeq: 837*

*Session: 12345678*

*Range: npt=30-*

Range头可能包含一个时间参数。该参数以UTC格式指定了播放开始的时间。如果在这个指定时间后收到消息，那么播放立即开始。时间参数可能用来帮助同步从不同数据源获取的数据流。

不含Range头的PLAY请求也是合法的。它从媒体流开头开始播放，直到媒体流被暂停。如果媒体流通过PAUSE暂停，媒体流传输将在暂停点（the
pause point）重新开始。

如果媒体流正在播放，那么这样一个PLAY请求将不起更多的作用，只是客户端可以用此来测试服务器是否存活

**4.5 PAUSE：**

PAUSE请求引起媒体流传输的暂时中断。如果请求URL中指定了具体的媒体流，那么只有该媒体流的播放和记录被暂停（halt）。比如，指定暂停音频，播放将会无声。如果请求URL指定了一组流，那么在该组中的所有流的传输将被暂停。如：

*C-\>S:*

*PAUSE rtsp://example.com/fizzle/foo RTSP/1.0*

*CSeq: 834*

*Session: 12345678*

 

*S-\>C:*

*RTSP/1.0 200 OK*

*CSeq: 834*

*Date: 23 Jan 1997 15:35:06 GMT*

PAUSE请求中可能包含一个Range头用来指定何时媒体流暂停，我们称这个时刻为暂停点（pause
point）。该头必须包含一个精确的值，而不是一个时间范围。媒体流的正常播放时间设置成暂停点。当服务器遇到在任何当前挂起（pending）的PLAY请求中指定的时间点后，暂停请求生效。如果Range头指定了一个时间超出了任何一个当前挂起的PLAY请求，将返回错误"457
Invalid Range"
。如果一个媒体单元（比如一个音频或视频禎）正好在一个暂停点开始，那么表示将不会被播放或记录。如果Range头缺失，那么在收到暂停消息后媒体流传输立即中断，并且暂停点设置成当前正常播放时间。

**4.6 TEARDOWN:**

请求终止指定URI媒体流传输并释放相关的资源。

*C-\>S:*

*TEARDOWN rtsp://example.com/fizzle/foo RTSP/1.0*

*CSeq: 892*

*Session: 12345678*

 

*S-\>C:*

*RTSP/1.0 200 OK*

*CSeq: 892*

1.  **SDP协议**

**5.1 SDP协议概述**

SDP(SessionDescription Protocol
)会话描述协议，用于描述多媒体会话，它为会话通知、会话初始和其它形式的多媒体会话初始等操作提供服务。

SDP的设计宗旨是通用性协议，所有它可以应用于很大范围的网络环境和应用程序，但 SDP
不支持会话内容或媒体编码的协商操作。

SDP信息包括：

-   会话名称和目标；

-   会话活动时间；

-   构成会话的媒体；

-   有关接收媒体的信息、地址等。

**5.2 SDP格式**

-   v= （协议版本）

-   o= （所有者/创建者和会话标识符）

-   s= （会话名称）

-   i=\* （会话信息）

-   u=\* （URI 描述）

-   e=\* （Email 地址）

-   p=\* （电话号码）

-   c=\* （连接信息 ― 如果包含在所有媒体中，则不需要该字段）

-   b=\* （带宽信息）

　　一个或更多时间描述（如下所示）：

-   z=\* （时间区域调整）

-   k=\* （加密密钥）

-   a=\* （0个或多个会话属性线路）

-   0个或多个媒体描述（如下所示）

　　时间描述

-   t= （会话活动时间）

-   r=\* （0或多次重复次数）

　　媒体描述

-   m= （媒体名称和传输地址）

-   i=\* （媒体标题）

-   c=\* （连接信息 — 如果包含在会话层则该字段可选）

-   b=\* （带宽信息）

-   k=\* （加密密钥）

-   a=\* （0个或多个会话属性线路）

V=0     ;Version 给定了SDP协议的版本

o=\<username\>\<session id\> \<version\> \<network type\> \<address type\>

\<address\>； Origin ,给定了会话的发起者信息

s=\<sessionname\> ;给定了Session Name

i=\<sessiondescription\> ; Information 关于Session的一些信息

u=\<URI\> ; URI

e=\<emailaddress\>    ;Email

c=\<networktype\> \<address type\> \<connection address\> ;Connect
Data包含连接数据

t=\<start time\>\<stop time\> ;Time

a=\<attribute\>     ; Attribute

a=\<attribute\>:\<value\>

m=\<media\>\<port\> \<transport\> \<fmt list\> ; MediaAnnouncements

1.  **传输方式：**

在使用RTP协议传输H.264视频的有效办法是从H.264视频中剥离出每个NALU，在每个NALU前添加相应的RTP包头，然后将包含RTP包头和NALU的数据包发送出去。对于每一个NALU，根据其包含的数据量的不同，其大小也有差异。在IP网络中，当要传输的IP
报文大小超过最大传输单元MTU（Maximum Transmission Unit
）时就会产生IP分片情况。在以太网环境中可传输的最大 IP 报文（MTU）的大小为 1500
字节。如果发送的IP数据包大于MTU，数据包就会被拆开来传送，这样就会产生很多数据包碎片，增加丢包率，降低网络速度。对于视频传输而言，若RTP
包大于MTU
而由底层协议任意拆包，可能会导致接收端播放器的延时播放甚至无法正常播放。因此对于大于MTU
的NALU 单元，必须进行拆包处理。

相关状态码：

1XX：保留，将来使用

2XX：成功，操作被接收、理解、接受

3XX：重定向，需要完成操作必须进一步操作

4XX：客户端出错，请求有语法错误或无法实现

5XX：服务端出错，服务器无法实现合法的请求

1.  **Libstreaming:**

7.1 Libstreaming介绍：

Libstrream库主要用于摄像头和（或）麦克风的RTP流式传输，基于android4.0以上，支持H.264，H263，AAC和AMR编码器。

7.2 基于Libstreaming的摄像头推流过程：

**7.2.1 需要的权限：**

\<uses-permission android:name="android.permission.INTERNET" /\>

\<uses-permission android:name="android.permission.WRITE\_EXTERNAL\_STORAGE" /\>

\<uses-permission android:name="android.permission.RECORD\_AUDIO" /\>

\<uses-permission android:name="android.permission.CAMERA" /\>

代码：

/\*获取权限\*/

private void permission(){  
if (Build.VERSION.*SDK\_INT* \>= 23) {  
int REQUEST\_CODE\_CONTACT = 101;  
String[] permissions = {Manifest.permission.*WRITE\_EXTERNAL\_STORAGE*,  
Manifest.permission.*READ\_EXTERNAL\_STORAGE*,  
Manifest.permission.*INTERNET*,  
Manifest.permission.*CAMERA*,  
Manifest.permission.*RECORD\_AUDIO*,  
};  
//验证是否许可权限  
for (String str : permissions) {  
if (this.checkSelfPermission(str) != PackageManager.*PERMISSION\_GRANTED*) {  
//申请权限  
this.requestPermissions(permissions, REQUEST\_CODE\_CONTACT);  
return;  
}  
}  
}  
  
}

**7.2.2 设置端口：**

/\*设置RTSP server端口\*/  
SharedPreferences.Editor editor =
PreferenceManager.*getDefaultSharedPreferences*(this).edit();  
editor.putString(RtspServer.*KEY\_PORT*, String.*valueOf*(8086));  
editor.commit();

**7.2.3 设置Session：**

SessionBuilder.*getInstance*()  
.setSurfaceView(mSurfaceView)  
//.setCallback(this)  
.setPreviewOrientation(90) //摄像头旋转  
.setContext(getApplicationContext())  
.setAudioEncoder(SessionBuilder.*AUDIO\_NONE*)  
.setAudioQuality(new AudioQuality(16000,32000)) //音频属性  
.setVideoEncoder(SessionBuilder.*VIDEO\_H264*) //H.264编码方式  
.setVideoQuality(new VideoQuality(320,240,20,500000)) //视频属性  
//.setVideoQuality(new VideoQuality(640,480,30,500000))  
.build();

**7.2.4 开启RTSP流：**

MainActivity.this.startService(new Intent(this,RtspServer.class));

**7.2.5 获取RTSP地址：**

private void displayIpAddress() {  
WifiManager
wifiManager=(WifiManager)getApplicationContext().getSystemService(Context.*WIFI\_SERVICE*);  
WifiInfo wifiInfo=wifiManager.getConnectionInfo();  
int address=wifiInfo.getIpAddress();//获取IP地址，注意获取的结果是整数  
Toast.*makeText*(this, "rtsp://"+intToIp(address)+":8086",
Toast.*LENGTH\_LONG*).show();//用toast打印地址  
IpAddress.setText("rtsp://"+intToIp(address)+":8086");  
  
Log.*i*("TAG\_IPAddress","rtsp://"+intToIp(address)+":8086");  
  
}

参考地址：

<https://blog.csdn.net/leixiaohua1020/article/details/11955341>（**RTSP协议学习笔记**

）

<https://www.cnblogs.com/lidabo/p/6553212.html> （**RTSP协议详解**）

<https://github.com/fyhertz/libstreaming>
（[libstreaming](https://github.com/fyhertz/libstreaming)）
