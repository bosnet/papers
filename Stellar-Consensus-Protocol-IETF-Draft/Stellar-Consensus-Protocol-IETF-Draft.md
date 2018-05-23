This document is the Korean translation of [The Stellar Consensus Protocol(SCP) IEFT Draft](https://tools.ietf.org/id/draft-mazieres-dinrg-scp-01.html), translated by BlockchainOS DevTeam. 

We give thanks to [David Maziéres](http://www.scs.stanford.edu/~dm/) for the permission of the translation of this paper.

However, this SCP IETF draft not complete yet. it is working in progress. [check this link.](https://github.com/stanford-scs/scp-spec).
so this document may changed near future. Please note this.
___

# Introduction

인터넷 인프라의 다양한 측면들은 인증된 매핑 같은 비가역적이고 투명한 데이터 셋의 업데이트에 의존한다.
공개키 인증서와 폐지, 투명로그[RFC6962], HSTS[RFC6797]와 HPKP[RFC7469]의 사전 설치 목록들, IP 주소 위임이 이러한 예에 포함된다[I-D.paillisse-sidrops-blockchain].
이 문서에서 언급하는 스텔라 합의 프로토콜은 인터넷 인프라 이해관계자들이 public state 에 대해 비가역적인 트랜잭션을 적용하는데 있어 서로 협력하게 한다 .
SCP 는 개방된 Byzantine agreement 프로토콜이며 허가된 개인 단체가 최소한의 quorum 멤버쉽을 명시하는 방식으로 ( quorum : 신뢰하는 특정 피어들을 의미함 ) Sybil 공격을 막는다. 각 참여자들은 전체적으로 신뢰 할 수 있는 피어의 조합을 선택한다. 이런 의존성 집합들이 추이적으로 종속되고, 프로토콜을 준수하는 많은 정직한 노드들을 충분히 포함하는한, safety는 보장 된다. 잘못된 설정들이 이론적으로는 가능할수도있지만 , 여러 유사점들을 통해 실제에서 왜 전이적 종속성이 중첩되는지에 대한 대한 통찰을 얻을 수 있다.
예를 들어, 여러개의 분리된 인터넷 프로토콜 네트워크가 주어졌을 때, 사람들은 세계 최고의 웹 사이트를 포함하는 네트워크가 인터넷이라는 사실에 동의하는데 이의가 없을 것이다.이러한 합의는  무엇이 세계최고의 웹 사이트를 구성하는가 에 대한 만장일치가 없이도 유지 된다.
마찬가지로, 네트워크 운영자가 피어링이나 전송을 가치있게 고려할 모든 자율 시스템(AS)을 나열하면 이러한 세트의 transitive closure 는 "1 차 ISP"지정에 대한 만장일치의 동의가 없더라도 상당한 중복을 포함하게 된다. 결과적으로, 다른 브라우저와 운영 체제가 유효한 인증 기관 목록이 약간 다르긴하지만 집합에 많은 중첩이 있으므로 "모든 CA"의 유효성 검사가 필요한 이론 시스템이 예상에서 벗어나진 않는다.
영문으로 된 safety 증명을 포함해 SCP  안의 좀 더 자세한 개요와 그 근거에 대해서는  scp 논문을 통해서 알 수 있다[SCP]. 특히 이 레퍼런스 ( SCP ) 는 비정상 노드가 있음에도 불구하고 quorum intersection 이라고 불리는 safety를 위한 속성이 SCP 에서 safety를 보장하고 어떠한 configuration에 대해서도 SCP 를 비잔틴 노드 장애로부터 최적으로 보호하는 데 충분하다는 것을 보여준다.
이 문서는 SCP의 end-system logic과 메시지 wire format을 설명한다.

# The Model

이번 세션은 컨센선스 프로토콜의 configuration 과 input/output 값에 대해서 설명한다. 

## Configuration

SCP 프로토콜 안의 각 참여자 또는 노드는 디지털화 된 서명 키를 가지고 있고 이러한 디지털 서명 키는  Node Id 로 이름붙이고 공개 키에 대응된다. 각 노드들은 쿼럼 슬라이스라고 부르는 자신을 포함한 하나 또는 하나 이상의 노드의 집합을 선택한다.
쿼럼 슬라이스는 쿼럼 슬라이스를 선택하는 노드가 전체 네트워크에 대해 전체적으로 말하는 것으로 생각하는 크거나 중요한 중요한 피어 집합을 표시하는 것이다. Quorum 은 최소한 각 멤버의 하나이상의 quorum slice 를 포함하는 비어있지 않는 집합이다.
예를 들어 v1에 단일 쿼럼 슬라이스 {v1, v2, v3}이 있고 v2, v3 및 v4에 각각 단일 쿼럼 슬라이스 {v2, v3, v4}가 있다고 해보자.이 경우, {v2, v3, v4}는 각 구성원에 대한 슬라이스를 포함하므로 쿼럼이다.
한편 {v1, v2, v3}은 v2 또는 v3에 대한 쿼럼 슬라이스를 포함하지 않으므로 쿼럼이 아니다. 이 예에서 v1을 포함하는 가장 작은 쿼럼은 모든 노드 {v1, v2, v3, v4}의 집합이다.

전통적인 비잔틴 합의 프로토콜과 달리 SCP의 노드는 자신이 속한 쿼럼 (따라서 쿼럼 슬라이스 중 적어도 하나를 포함)에 대해서만 주의를 기울인다. 직관적으로, 이것은 Sybil 공격으로부터 노드를 보호한다.위의 예에서 만약 v3 가 프로토콜을 따르지 않고 v5,v6,... v100  같은 악의적인 96개의 sybil이 만들어진다면 정직한 노드의 Quorum은  여전히 서로를 포함하여 v1, v2 및 v4가 출력 값에 대해 계속 일치하도록 한다.
SCP 프로토콜의 모든 메시지는 송신자의 Quorum slice 를 지정한다.따라서 메시지를 수집 함으로써 노드는 무엇이 Quorum 을 구성하는 것을 동적으로 학습하고 특정 메시지가 자신이 속한 Quorum 에 의해 언제 전송 했는지 결정할 수 있다.다시 말하면, 노드는 자신이 속하지 않는 Quorum에 대해 신경 쓰지 않는다.

## Input and output

SCP는 연속적으로 숫자가 매겨진 slot으로 부터 일련의 출력 값을 만든다. Slot을 시작할 때, 각 노드의 상위 레벨 소프트웨어가 후보 입력 값(거래 풀에서 이번 거래 하나 또는 composite)을 제공한다. SCP의 목표는 정상 노드가 slot의 출력을 만들기 위해 노드의 입력 값 하나 또는 집합에 동의하는 것이다. 하나의 슬롯이 완료되고, 5초 이후에 프로토콜은 다음 슬롯으로 넘어가서 다시 수행된다.

하나의 값은 일반적으로 복제된 state machine으로 적용되는 동작 집합으로 encode 된다(입력 값은 거래 같은 동작의 집합이다). 슬롯 사이에 멈춰 있는 동안, 노드는 다음 액션의 집합을 누적하고, 그렇게 함으로써 랜덤하게 많은 개인적인 state machine operation이 합의하면서 소요되는 비용을 아낀다(조각의 합의가 많아지면 느리기 때문에, 한데 뭉쳐서 한 번에 합의한다).

실제로, 하나 또는 소수 노드의 입력 값 만이 주어진 슬롯의 출력 값에 실제로 영향을 미친다. Section 3.4에서 설명하듯이, 사용할 노드의 입력 값은 슬롯 번호와 출력 히스토리 그리고 노드의 public key 의 암호화 해시의 영향을 받는다. 노드가 출력 값에 영향을 줄 확률은 다른 노드의 쿼럼 슬라이스에 나타나는 빈도에 따라 다르다(즉, 다른 노드의 쿼럼 슬라이스에 많이 나타날 수록, 다음 출력 값에 내가 영향을 줄 확률이 높다. 즉, 여러 쿼럼에 포함될 수록 합의 과정에 많이 참여한다.).

SCP의 관점에서 볼 때, 값은 단지 사람이 읽기 어려운 바이트 배열이고, 그 해석은 상위 레벨 소프트웨어에 맡겨진다. 그러나, SCP에는 여러 후보 값을 하나의 composite 값으로 합치는 combining function가 있다. 노드가 슬롯에 여러 값을 지정하면, SCP 노드 들은 이 하나의 composite 값으로 만들기 위해 combining function을 호출한다. 예를 들어, 값이 트랜잭션의 집합으로 구성된 응용 프로그램에서, combining function은 트랜잭션 집합의 union을 취할 수 있다. 또는, 값이 타임 스탬프와 트랜잭션 집합을 나타내는 경우, combining function은 가장 높은 해시 값을 가진 트랜잭션 집합과, 가장 높은 nominated timestamp를 쌍으로 지정할 수 있다(두 factor로 함께 sorting 할 수 있다는 뜻?).

# Protocol

이 프로토콜은 노드의 쿼럼 슬라이스 간에 디지털 서명 된 메시지를 교환하는 것으로 구성된다. 모든 메시지의 형식은 XDR을 사용한다[RFC4506]. 쿼럼 슬라이스 외에도 메시지는 개념적 문장 집합에 대한 투표를 컴팩트하게 전달한다. 쿼럼 슬라이스에서 투표하는 핵심 기술을 federated voting이라고 정의한다. 우리는 다음 챕터에서 federated voting을 설명하고, 그 하위 절에서 세부 프로토콜 메시지를 설명한다.

## Federated Voting

Federated voting은 노드가 statement를 confirm 하는 과정이다. 모든 투표가 성공하는 것은 아니며, statement a 에 투표하는 시도가 stuck 되면 노드는 a 또는 !a 모두 confirm 할 수 없다. 그러나, 노드가 statement a를 confirm 하는 데에 성공하면, federated voting은 다음 두 가지를 보장한다.

어떤 설정과 실패 시나리오에서도, 두 노드의 safety를 보장하는 어떤 프로토콜 에서도, 어떤 두 개의 정상 노드도 모순되는 statements를 confirm 하지 않는다. (즉, ill-behaved node에도 불구하고, 두 노드의 쿼럼 인터섹션은 safety를 만족한다.)
1로 safety가 보장 된 노드가 statement a를 confirm 한다면, 그리고 그 노드가 전체적으로 잘 행동하는(well-behaved) 노드로 구성된 하나 이상의 쿼럼의 멤버라면, 결국  쿼럼의 모든 멤버는 또한 a를 confirm 할 것이다.
직관적으로, 이런 조건은 FLP imposibility result와 호환되는 liveness의 약한 형태를 보장할 뿐 아니라, 노드 간의 agreement를 보장하는 핵심(key) 이다 [FLP].

노드 ‘v’가 peer로 부터 federated voting message m 의 서명된 복사본을 받았을 때, 메시지에 따라 두 가지 threshold가 state를 변경한다. 우리는 이러한 threshold 들을 다음과 같이 정의한다:

Quorum threshold: ‘v'가 속한 쿼럼의 모든 구성원(‘v’를 포함해서)이 메시지 m을 발행한 경우
Blocking threshold: ‘v’의 각 쿼럼 슬라이스 중 적어도 하나의 멤버(‘v’ 자신을 포함할 필요는 없음)가 메시지 m을 발행한 경우
각각의 노드 ‘v’ 는 federated voting 하는 동안 a statement와 관련된 여러 유형의 메시지를 보낼 수 있다:

vote a state는 다음의 의미를 갖는다.
a는 valid statement 이고, !a 처럼 반대되는 어떤 메시지에도 vote 하지 않을 것이라고 약속한다.
accept a 는 다음의 의미를 갖는다.
노드는 a 에 동의하거나 동의하지 않을 수 있다. 만약 동의하지 않는다면, 시스템은 전체가 올바른 노드로 구성된 어떤 쿼럼도 v를 포함하지 않는 비잔틴 실패의 비극적인 set을 경험한다.
(그럼에도 불구하고, a를 accept 하는 것 만으로는 충분하지 않다. 그렇게 하면 합의를 위반할 수도 있다. 이것은 올바른 정족수가 부족해서 stuck 되는 것 보다 더 나쁜 상황이다.)
vote-or-accept a는 위의 두 메시지의 합집합이다. 노드가 vote a 또는 accept a 메시지를 보내면 암시적으로 이 메시지를 보낸다. vote와 accept를 구별하는 것이 불편하고 불필요한 경우, 노드는 명시적으로 vote-or-accept 메시지를 보낼 수 있다.
confirm a 는 accept a 가 전송 노드에 의해 quorum threshold에 도달했음을 가리킨다. 이 메시지는 accept a 와 똑같이 해석되지만, 이 메시지를 받은 노드는 전송 노드의 쿼럼 슬라이스를 무시함으로써, 그들의 쿼럼을 최적화할 수 있다. 왜냐하면, confirm a의 의미는 전송 노드의 쿼럼 슬라이스를 이미 확인했다는 의미이기 때문이다.
Figure 1은 federated voting process를 설명한다. 노드 v 는 과거 투표 또는 v가 보낸 accept message와 반대되는 statement가 아닌, valid statement에 투표한다. vote message가 quorum threshold에 도달하면 노드는 a를 받아들인다. 사실, v는 vote-or-accept message가 quorum threshold에 도달하면 a를 승인하고, 일부 노드가 먼저 투표하지 않고도 a를 승인할 수 있습니다. 특히, 이미 !a에 투표해서 a에는 투표할 수 없는 노드도 여전히 a를 받아들입니다. accept a 메시지가 blocking threshold에 도달했을 때(이것은 !a 가 비극적인 비잔틴 실패 케이스를 제외하고는 quorum threshold에 도달할 수 없음을 의미한다.).

만약 accept a 메시지가 quorum threshold에 도달하는 경우, v는 a를 confirm 하고 federated vote는 성공한다. accept 메시지는 첫 번째 투표 메시지가 성공했다는 사실에 근거해서 두 번째 메시지를 구성한다. 한 번 v가 confirmed state가 되면, 다른 노드가 a를 confirm 하는 것을 더욱 효율적으로 돕기 위해 confirm a 메시지를 발행한다. 왜냐하면 confirm a 메시지를 받으면 v에서 그들의 쿼럼을 검색하는 불필요한 행동을 하지 않을 수 있기 때문이다.

```                "vote-or-accept a"          "accept a"
                     reaches                 reaches
                 quorum threshold        quorum threshold
                +-----------------+     +-----------------+
                |                 |     |                 |
                |                 V     |                 V
             +-----------+     +-----------+     +-----------+
  a is +---->|  voted a  |     |accepted a |     |confirmed a|
 valid |     +-----------+     +-----------+     +-----------+
       |           |                 ^
+-----------+      |                 | "accept a" reaches
|uncommitted|------+-----------------+ blocking threshold
+-----------+      |
       |           |
       |     +-----------+
       +---->|  voted !a |
             +-----------+
```

## Basic Types

SCP는 아래와 같이 정의되어 있듯이 32, 64bit의 integer 형을 쓴다.

```C
typedef unsigned int uint32;
typedef int int32;
typedef unsigned hyper uint64;
typedef hyper int64;
```

SCP는 SHA-256 hash function [RFC6234]을 쓰는데 해쉬 값들은 32 바이트의 simple array 를 나타낸다. 

```C
    typedef opaque Hash[32];
```

SCP는 ED25519 를 전자 서명 알고리즘으로 사용한다[RFC8032]. 그러나, 암호학적 기동성을 위해 공개키들은 다른 키 타입들과 호환될 수 있는 union 타입으로 표시된다. 

```C
typedef opaque uint256[32];

enum PublicKeyType
{
    PUBLIC_KEY_TYPE_ED25519 = 0
};

union PublicKey switch (PublicKeyType type)
{
case PUBLIC_KEY_TYPE_ED25519:
    uint256 ed25519;
};

// variable size as the size depends on the signature scheme used
typedef opaque Signature<64>;
```

노드 들은 공개키들 이고 그 값들은 단순히 바이트의 opaque array 이다.

## Quorum Slices

이론적으로, quorum slice는 여러 노드 set의 임의적인 집합이 될 수 있다. 그러나 집합에 대한 임의적인 서술은 정확하게 encode 될 수 없다. 대신 우리는 quorum slice를 전체 n명 중 k명(k-of-n) 멤버들의 set으로 지정하고, 여기서 n명의 멤버들 각각은 개별 노드 ID, 또는 재귀적으로, 다른 n명 중 k명(k-of-n) set이 될 수 있다.

```C
// supports things like: A,B,C,(D,E,F),(G,H,(I,J,K,L))
// only allows 2 levels of nesting
struct SCPQuorumSet
{
    uint32 threshold;            // the k in k-of-n
    PublicKey validators<>;
    SCPQuorumSet1 innerSets<>;
};
struct SCPQuorumSet1
{
    uint32 threshold;            // the k in k-of-n
    PublicKey validators<>;
    SCPQuorumSet2 innerSets<>;
};
struct SCPQuorumSet2
{
    uint32 threshold;            // the k in k-of-n
    PublicKey validators<>;
};
```

어떤 노드 v가 보내는 메시지 안에서 k는 threshold 값이며, n은 validator들의 사이즈와 내부 집합 벡터의 합이다. v가 아래 세 가지 요소를 가질 때, v가 보내는 메시지 m이 quorum threshold에 도달한다.

1. 노드 v 자신이 디지털 서명된 메시지를 발행했고,
2. 메시지 m에 서명한 validators에 속한 노드 수와 재귀적으로 이 조건을 만족하는 innerSets의 수를 더한 것이 k보다 크거나 같고,
3. 이 세 가지 조건들은 condition #2를 만족한 어떤 노드들의 조합에 적용(재귀적으로) 된다.

이러한 statement를 만든 validators의 수와 blocking threshold 가 n - k(n minus k)보다 큰 innerSets의 수를 더한 것이 blocking threshold에 도달했을 때 메시지는 v에 대해 blocking threshold에 도달한다(Blocking threshold 는 앞서 언급한 condition #3처럼 다른 노드들에게 재귀적으로 확인할 필요 없는 로컬 속성이다.).

validator들의 숫자 만큼의 statement의 값과 n - k를 초과한 내부 집합이 blocking threshold에 도달할 경우 메시지는 노드 v의 blocking threshold에 도달한다(Blocking threshold 는 앞서 언급한 condition #3처럼 다른 노드들에게 재귀적으로 확인할 필요 없는 로컬 속성이다.).

Section 3.9에서 설명하는 것 처럼, 모든 프로토콜 메시지들은 송신자의 SCPQuroumSet의 암호화된 해쉬와 디지털 서명의 쌍(pair)이다. 다음 섹션 들에서 설명하는 내부 프로토콜의 메시지들은 이런 quorum slice specification과 디지털 시그니쳐와 함께 수신된다고 이해해야 한다.

## Nomination

 각각의 슬롯에서, NOMINATION 단계로 부터 SCP 프로토콜이 시작된다. 그 목적은 컨센서스 프로토콜을 위해 하나 또는 여러 개의 출력 값을 만들어내는 것이다. 노드는 아래와 같은 포멧으로 단조 증가(monotonically growing)하는 value set을 포함하는 nomination 메시지를 보낸다.

```C
  struct SCPNomination
 {
   Value votes<>;     // X
   Value accepted<>;  // Y
 };
```

votes와 accepted set은 서로소이다. 두 set에 모두 포함될 만한 value는 accepted set에 위치한다.

  votes는 sender에 의해 nominated 된 후보 값으로 구성된다. 각 노드는 일련의 nomination 라운드를 진행한다. 라운드를 진행함에 따라, peer의 set이 늘어나며, 그로부터 받는 SCPNomination 메시지의 votes and accepted 필드를 내 SCPNomination 메시지의 votes 필드로 추가하기 때문에 나의 value set이 늘어난다. Slot i의 round n에서, 각각의 노드는 additional peer를 아래와 같이 결정한다. 그리고 노드는 additional peer로 부터 nominated value를 나의 SCPNomination 메시지에 추가한다.

- Gi(m) = SHA-256(I || output[I-1] || m)이며, output[I-1]은 slot i-1의 컨센서스 결과이다. 그리고 초기 값 Slot 1의 output은 0-byte value이다.
    - Gi(m) 값은 XDR opaque 벡터로 인코딩 된다. 이 벡터는 4바이트의 배수로 zero-padded 된 contents이고 길이는 32바이트이다.
    - Gi의 출력은 빅 엔디안 형식의 256 비트 2진수이다.
- 각각의 피어 v에 대해, v가 들어있는 쿼럼 슬라이스로 weight(v)를 정의한다.
- 노드 v의 집합 neighbors(n)을 정의한다. neighbors(n)는 Gi(“N” || n || v) < 2^{256} * weight(v) 공식으로 찾는다.
    - weight가 높을수록 neighbors에 뽑힐 확률이 높다.
    - neighbors는 쿼럼 슬라이스의 서브셋이다.
- Gi(“P” || n || v)로 priority(n, v)를 정의한다.

Nomination이 끝날 때 까지 각각의 라운드 n에서, 노드는 neighbors(n)의 노드들 중에서 가장 높은 priority(n, v)를 갖는 peer v(리더)로 부터 받은 메시지를 echoing 한다(Echoing은, 리더가 nomination 한 value에 내 value를 포함해서 전달하는 것이다. High level software로부터 받은 것을 전달하는 것은 broadcasting 이다.). v를 echoing 하기 위해,노드는 v의 votes와 accepted set으로 부터 유효한 value를 자신의 votes set으로 병합한다(즉, 다른 노드의 accepted를 accepted로 받지 않고, votes로 받는다. 그 노드가 다른 쿼럼 셋으로 부터 받은 결과일수 있어서 믿지 못하기 때문으로 추측한다). 메시지 validity는 상위 레벨 소프트웨어에 의해 결정되지만, 대부분 replicated state와는 독립적이어야 한다. 예를 들어, 노드는 값을 구문적으로 분석하고, 서명된 트랜잭션이 올바른 디지털 서명을 가지고 있으며, 타임스탬프가 미래는 아닌지 확인한다.(즉, votes에서, accepted로 넘어가는 과정에서는 구문적 분석, 서명, 타임스탬프 정도만 체크한다.)

노드는 votes 또는 accepted 필드가 비어있을 때, SCPNomination 메시지를 보내지 않는다. 이 두 필드가 모두 비었을 때, 이번 라운드에서 그 이웃들로부터 가장 높은 우선순위를 가지는 노드(그렇기 때문에 자신의 votes를 전파해야한다)는 상위 레벨 소프트웨어의 입력 값을 votes 필드에 추가해야 한다(즉, 리더는 거래를 votes 에 추가 가능하다.) 가장 높은 우선순위가 아닌 노드는 그들이 전파한 SCPNomination 메시지를 기다린다. 새로운 값으로 노미네이션 할 수 있는 노드는 가장 높은 우선순위를 갖는 노드(리더) 뿐이다. 리더가 아닌 노드들은 echoing을 기다린다.

만약, 특정 유효한 값 x가 quorum threshold에 도달했을 때(이 의미는 한 쿼럼의 모든 노드가 x를 votes 또는 accepted 필드에 포함하고 있다는 뜻이다.), 그 노드는 x를 votes 필드에서 accepted 필드로 옮긴다. 그리고 그렇게 만들어진 새로운 SCPNomination 메시지를 broadcasting 한다.
비슷하게, 만약 x가 한 노드의 peers들의 accepted 필드 안에서 blocking threshold에 도달(이 의미는 이 노드의 모든 쿼럼 슬라이스가 적어도 하나의 accepted 필드에 x를 가진 노드를 포함한다. 다르게 말하면, 노드의 모든 쿼럼 슬라이스에는 accepted 필드에 x가 있는 노드가 하나 이상 있다.)하면, 그 후 노드는 자신의 accepted 필드에 x를 추가한다(필요하다면 votes에서는 삭제한다.). 이 두 가지 케이스들은 Figure 1에서 accepted state에 추가되는 두 가지 조건과 연관되어 있다.

하나의 노드는 accepted field에서 어떤 값 x가 quorum threshold에 도달하는 순간 NOMINATION 단계를 종료한다. Section 3.1에 따르면, 이런 조건은 노드가 x를 nominated 라고 confirm할 때와 연관이 있다. NOMINATION 상태가 끝난 노드는 더 이상 새로운 값을 votes set에 추가하지 않는다. 그러나, 적당하다면 accepted set에는 계속 새로운 값을 추가 가능하다(votes 필드에서 accepted로 넘어온 값들 또는 blocking threshold로 받은 값들로 추측한다.). 이러한 과정은 더 많은 값들이 백그라운드에서 nominated로 confirmed 되도록 한다.

n 라운드가 2+n 초 동안 지속되고, 그 후에도 만약 NOMINATION 과정이 끝나지 않으면, 노드는 라운드 n+1로 넘어간다. 노드는 현재 라운드 뿐만 아니고, 이전 라운드의 리더로부터 받은 votes도 반영한다. 특히, 노드는 이런 값들이 라운드가 끝난 뒤에 추가될 때에도 이전 라운드의 리더들로 부터 nominated 된 값을 votes 필드로 추가한다(즉, 리더로 부터 nominated 된 값은 votes로 추가 가능하다).

특정 라운드에서 SHA-256 해시가 가장 낮은 10개의 값으로 votes 크기를 제한해서, 메시지 크기가 blowing out 하지 않도록 주의해야 할 것이다.

## Ballots

Nomination 단계 후에는(적어도 하나의 후보 value가 confirmed nominate 됐을 경우) 노드는 세 단계의 balloting 단계를 거치게 된다: Prepare, Commit, Externalize. Balloting 또한 federated voting 과정을 거쳐서 ballots에 대한 commit statement와 abort statement 중 하나를 결정한다. Ballot은 counter 와 candidate value의 쌍으로 이루어진 structure이다.

```C
// Structure representing ballot <n, x>
struct SCPBallot
{
    uint32 counter; // n
    Value value;    // x
};
```

<n, x> 는 ballot 의 counter가 n, value이 x 라는 의미이다.

  Ballot은 value보다는 counter의 값이 중요하게 순서가 정해진다. 즉, b1 < b2 는 b1.counter < b2.counter 또는, b1.counter == b2.counter && b1.value < b2.value 이다. Counter가 큰 ballot이 더 크며, counter가 같을 때는 value가 큰 ballot이 더 크다(value들은 strings of unsigned octet 들이 알파벳 순서로 비교된다).

  프로토콜은 노드들이 어떠한 ballot b 에 대해 “commit b”를 confirm할 때 까지 작은 Ballot부터 순서대로 federated voting 과정을 거치며 진행된다. 이 시점에 consensus는 종결(terminate)되고 그 슬롯의 결과로 b.value를 내놓는다. 하나의 슬롯에 하나의 value만 선택되는것을 보장하고, 개별 ballot이 stuck이 되어도 프로토콜은 stuck이 되지 않는 것을 보장하기 위해서, voting에는 다음 두 개의 제약 조건이 있다.

1. 노드는 같은 ballot b 에 대해 “commit b” 와 “abort b” 를 동시에 vote할 수 없다(두개의 결과는 모순된다.).
2. 노드는 어떠한 ballot b 에 대해, 그보다 더 작고 value가 다른 ballot 들을 abort 하기 전까지는 “commit b”를 vote 할 수 없다.

위의 두 번째 조건은 ballot b 를 commit 하기 전에 많은 수의 ballot들에 abort로 voting 하는 것을 필요로 한다. 우리는 이 과정을 preparing ballot b 라 부르고, 관련된 abort statement의 집합을 다음의 notation들로 소개한다.

- prepare(b) 는 b 보다 작은 모든 ballot에 대한 abort statement 이다. 즉, prepare(b) = { abort b1 | b1 < b AND b1.value != b.value}
- “vote prepare(b)” 는 prepare(b)에 포함되는 모든 abort statement에 대한 vote message의 set이다.
- 비슷하게 “accept prepare(b)”, “vote-or-accept prepare(b)”, “confirm(b)” 는 prepare(b)의 모든 abort statement에 대한 accept, vote-or-accept, confirm 메시지의 set이다.

이 용어 정의를 통해, 노드는 “commit b” 에 대해 vote 하기 전에 prepare(b)를 confirm 해야 한다.

## Prepare Messages

Balloting의 첫번째 단계는 Prepare이다. 이 단계에서 노드는 아래의 SCPPrepare 메시지를 보낸다

```C
struct SCPPrepare
       {
           SCPBallot ballot;         // b
           SCPBallot *prepared;      // p
           SCPBallot *preparedPrime; // p'
           uint32 hCounter;          // h.counter or 0 if h == NULL
           uint32 cCounter;          // c.counter or 0 if c == NULL
       };
```

이 메시지는 다음의 federated voting message를 함축적으로 전달한다.

- Vote-or-accept prepare(ballot)
- Prepared != Null 일때: “accept prepare(prepare)”
- preparedPrime != Null 일때: “accept prepare(preparedPrime)”
- hCounter !=0 일때: “confirm prepare(<hCounter, ballot.value>)”
- cCounter != 0 일때: cCounter <= n <= hCounter 인 모든 n 에 대해 “vote commit(<n, ballot.value>)”

참고로, SCPPrepare message가 유효하기(valid)하기 위해서는 아래의 두 조건을 만족해야 한다.

- “preparedPrime < prepared <= ballot” (모든 NULL이 아닌 “prepared”, “preparedTime”에 대해서)
- “cCounter <= hCounter <= ballot.counter”를 만족해야한다.

받은 Federated vote message 에 기반으로, 각 노드는 어떤 ballot들이 accepted prepared 또는 confirmed prepared 가 되었는지 기록한다. 노드는 다음의 ballot들을 이용해서 자신의 SCPPrepare 메시지의 각 필드를 채운다. 

- Prepared
    - accepted prepared ballot 중 가장 큰 값, 어떤 ballot들도 accepted prepared 되지 않았을 때는 NULL
- preparedPrime
    - “PreparedPrime.value != prepared.value” 를 만족하는 accepted prepared ballot 중에 가장 큰 값, 그런 ballot이 없을때는 NULL
- hCounter
    - 가장 큰 confirmed prepared ballot의 counter 값, 어떠한 ballot들도 confirmed prepared 되지 않았을때는 0. (가장 큰 confirmed prepared ballot의 "value"는 ballot.value로 동일하기 때문에 counter만 포함된다.) 
- ballot
    - 노드가 prepare와 commit을 시도하려는(attempting) 현재 ballot. 각 필드가 설정되는 규칙은 아래에 상세하게 나온다. 
- ballot.counter
    - Prepare 단계로 들어갈때 “counter” 는 1 로 초기화된다.
    - 노드가 자신이 속한 쿼럼의 노드로부터 받은 각 메시지의 ballot.counter가 local ballot.counter보다 크거나 같을때, local 타이머를 “Ballot.counter +1” 초로  설정한다. (참고로 SCPPrepare 뿐만 아니라 SCPCommit과 SCPExternalize에 있는 ballot 필드 또한 포함한다.)
    - 타이머가 시작되면, 노드는 ballot counter를 1씩 증가시키며(?)(맞는듯 by scone) ballot.value 에 나와있는 규칙대로 새 value를 결정(determine) 한다. 
    - local ballot.counter보다 더 큰 "ballot.counter"를 갖고 있는 노드들이 blocking threshold를 이룬다면, 그 즉시 local ballot.counter를 blocking threshold를 이루지 않도록 하는(the case의 뜻이 무엇일지.. local ballot.counter가 다른 ballot.counter보다 작지 않도록 하는?) 최소한의 ballot.counter로 설정한다. (그렇게 하면서 옛 counter와 연관된 보류중인 타이머를 중지시킨다)
    - “ballot < h” 인 새로운 ballot "h" 가 confirmed prepared 된다면, 바로 h를 ballot으로 지정한다. XXX - 이게 일어날리가 있나?
    - counter를 고갈시키지 않기 위해서, ballot.counter는 [100만 + 노드가 현재 슬롯에서 scp 를 실행하는 데에 쓰고있는 시간(초)] 보다 항상 작아야한다. 위의 다섯 가지 규칙들이 counter를 이 값보다 크게 증가시키려고 한다면, ballot.counter를 허용될 정도의 최대값으로 설정하거나, 이미 최대값이라면 1초를 기다린 후에 counter를 증가시킨다. 
- ballot.value
    - Ballot counter가 바뀔 때마다, value 또한 다음과 같이 재계산된다.
    - Confirmed prepared된 ballot이 있다면, 그 “ballot.value” 이 highest confirmed ballot h 의 h.value가 된다. 그렇지 않으면, (hCounter가 0 일때) ballot.value는 모든 confirmed nominated value에 적용되는 deterministic combining function 의 output으로 결정된다. 참고로, balloting 단계의 background에서 confirmed nominated values는 증가할 수 있기 때문에, “ballot.value”는 hCounter가 0일 때도 변할 수 있다.
- cCounter
    - cCounter는 내부적으로 관리되는 “commit ballot” 인 “c”(초기값이 NULL인) 에 기반하여 관리된다. “C”가 NULL일때는 cCounter == 0, 그 외의 경우에는 c.counter. “c”는 다음과 같이 업데이트 된다.
    - "prepared > c && prepared.value != c.value" 또는 "preparedPrime > c && preparedPrime.value != c.value” 일때 c는 NULL로 초기화된다
    - “c == NULL” 이고, highest confirmed ballot “h” (hCounter를 결정하는 ballot) 가 “prepared”또는 “preparedPrime”에 의해 abort 되지 않았을때, "c.value == h.value && c.counter <= h.counter && ballot <= c”를 만족하는 “c”를 lowest balllot으로 설정한다.  XXX - ballot 규칙이 주어졌을때 “c=ballot”을 설정하기 위해?

노드에서 Prepare 단계는, 어떠한 ballot b 에 대해서 “commit b” statement가 federated voting을 통해 accept state가 될때 끝난다.

## Commit messages

Commit 단계에서는 노드가 어떤 ballot b의 accept “commit b” 를 가지고 있다. 그리고 b.counter를 이용해서 이 statement를 confirm 해야 한다. 이 단계에서 노드는 다음과 같은 메시지를 보낸다.

```C
struct SCPCommit
     {
        SCPBallot ballot;       // b
        uint32 preparedCounter; // prepared.counter
        uint32 hCounter;        // h.counter
        uint32 cCounter;        // c.counter
     };
```

이 메시지는 다음과 같은 federated vote message를 전달한다. 여기서 infinity 는 2^32 이다 (ballot counter를 marshaled form으로 나타냈을 때의 최대 값보다 더 큰 값)

- accept commit “<n, ballot.value>”, cCounter <= n <= hCounter 인 모든 n에 대해
- “Vote-or-accept prepare(<infinity, ballot.value>)”
- “Accept prepare(<preparedCounter, ballot.value>)”
- “Confirm prepare(<hCounter, ballot.value>)”
- “Vote commit <n, ballot.value>)”, n >= cCounter인 모든 n에 대해

 노드는 다른 노드에게 보낼 SCPCommit message 내의 필드들을 다음과 같이 계산한다.

- Ballot
    - 이 필드는 Prepare 단계에서 관리되는 것과 동일하게 관리되지만, “ballot.value”는 변하지 않고, “ballot.counter”만 변할 수 있다. “ballot.counter” 값은 어떤 federated voting 메시지에도 나타나지 않은 것을 확인하라. 이 필드를 계속 업데이트 하고 보내는 목적은 아직 Prepare단계에 있는 다른 노드의 counter를 동기화하는데에 도움을 주는 데에 있다. 
- preparedCounter
    - 이 필드는 highest accepted prepared ballot 의 counter 이다. — 이는 Prepare 단계의 “prepared” 필드와 동일하게 관리된다. “value”필드는 “ballot”와 항상 동일하므로 Commit 단계에서는 counter만 보낸다.
- cCounter
    - 노드가 commit c를 accept 했을 때 lowest ballot c의 counter 이다(마찬가지로, c.value와 ballot.value가 같기 때문에 value는 포함하지 않는다.).
- hCounter
    - 노드가 commit c를 accept 했을 때 highest ballot h의 counter 이다(마찬가지로, h.value와 ballot.value가 같기 때문에 value는 포함하지 않는다.).

노드가 어떠한 ballot b에 대해서 “commit b”를 confirm 하는 순간 Externalize 단계로 넘어간다. 

## Externalize messages

어떤 ballot b 에 대해 “commit b”를 confirm했을 때, 노드는 Externalize 단계로 넘어 온다. Externalize 단계로 넘어오자마자 SCP는 b.value를 결과를 현재 슬롯의 value로 내놓는다.

다른 노드들이 해당 슬롯에 대해 더욱 빠르게 consensus를 이루는것을 돕기 위해, 이 단계로 들어온 노드들은 다음과 같은 메시지를 또한 보낸다.

```C
struct SCPExternalize
{
    SCPBallot commit;         // c
    uint32 hCounter;          // h.counter
} externalize;
```

SCPExternalize 메시지는 다음과 같은 federated voting message를 보낸다. 여기서 infinity 는 2^32 이다 (ballot counter를 marshaled form으로 나타냈을 때의 최대 값보다 더 큰 값)

- “accept commit <n, commit.value>”, n >= commit.counter 를 만족하는 모든 n에 대해
- "comfirm commit<n, commit.value>", commit.counter <= n <= hcounter 를 만족하는 모든 n에 대해 .
- "vote-or-accept prepare (<infinity, commit.value>)"
- "comfirm prepare( <inifinity, commit.value>)"

Commit

- 가장 작은 confirmed committed ballot

hCounter

- 가장 큰 confirmed committed ballot의 counter

## Message envelopes

각각의 서명된 메시지에 모든 정보(full context)를 담기 위해서, 모든 서명된 메시지는 "SCPStatement" Union 타입의 일부분이다. 이 타입은 메시지가 적용되는 슬롯의 이름을 나타내는 “slotIndex”과 더불어 메시지의 “type”도 포함한다. 서명된 메시지와 그 서명은 “SCPEnvelope” 구조체 안에 포함된다.

```C
enum SCPStatementType
{
    SCP_ST_PREPARE = 0,
    SCP_ST_COMMIT = 1,
    SCP_ST_EXTERNALIZE = 2,
    SCP_ST_NOMINATE = 3
};

struct SCPStatement
{
    NodeID nodeID;    // v (node signing message)
    uint64 slotIndex; // i

    union switch (SCPStatementType type)
    {
    case SCP_ST_PREPARE:
        SCPPrepare prepare;
    case SCP_ST_COMMIT:
        SCPCommit confirm;
    case SCP_ST_EXTERNALIZE:
        SCPExternalize externalize;
    case SCP_ST_NOMINATE:
        SCPNomination nominate;
    }
    pledges;
};

struct SCPEnvelope
{
    SCPStatement statement;
    Signature signature;
};
```

## Safety considerations

노드들이 쿼럼슬라이스를 "잘(well)" 설정하지 않는다면 이 프로토콜은 safe 하지 않다.

# Acknowledgements

스텔라 재단은  프로토콜 개발을 지원하고 SCP 의 첫번째 프로덕션을 배포했다.
Dirk Kutscher, Sydney Li, Colin Man, Melinda Shore, and Jean-Luc Watson 을 포함한 IRTF DIN 그룹은 이 Specification에 대한 구상과 동기를 주는데 도움이 되었다.
