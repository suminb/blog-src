---
author:
  email: suminb@gmail.com
  first_name: Sumin
  last_name: Byeon
categories:
- Geeky Stuff
draft: false
layout: post
meta: null
published: true
redirect_from:
- /post/kakaopay-qrcode
tags:
- kakao
- kakaopay
- python
- reverse-engineering
title: 카카오페이 QR 코드 리버스 엔지니어링
summary: 카카오페이 QR 코드를 리버스 엔지니어링 해서 원하는 송금 코드를 만들어냈던 이야기
og_image: /attachments/2019/kakaopay-qrcode/kakaopay.png
---

소프트웨어 엔지니어라는 직업이 매력적인 이유는 엄청나게 많지만, 오늘은 그
중에서도 '가내수공업이 가능한 점'을 꼽고 싶다. 필요한 것이 있으면 직접 만들 수도
있다는 말이다. 물론 제대로 된 무언가를 만들어 내기 위해서는 수많은 고민과 적지
않은 노력이 필요하긴 하다.

오늘 여러분과 공유할 이야기는 취미 생활을 하면서 겪었던 문제를 해결하는 과정에
대한 이야기다. (그렇다, 일과 놀이의 경계가 불분명한 삶을 살고 있다.) 이 문제를
해결하기 위해서 굉장히 많은 고민을 하고 여러가지 시행착오를 겪은 것 같은데, 막상
글로 정리하고 보니 분량이 얼마 되지 않아서 "내가 뭘 했나" 싶다.

이 이야기는 크게 네 가지 파트로 구성되어있다: 천원경매, 카카오페이, QR 코드
분석, 그리고 QR 코드 생성이다.

천원경매
--------

인테리어의 최고봉은 집안의 물건을 없애는 것이다. 아무리 미적 감각이 뛰어난
인테리어 전문가라고 해도 좁은 집에 수많은 잡동사니가 뒤엉켜 있는 상황을
정리하기는 쉽지 않다. 집안의 모습은 물건이 적을수록 깔끔해진다. 말처럼 쉽지는
않지만, 필요 없는 물건들을 과감하게 정리하는 것이 답이다. 아직 그것만큼 효과적인
인테리어 개선 방법을 찾지 못했다.

.. figure:: /attachments/2019/kakaopay-qrcode/dollar-auction.jpg
   :width: 80%

그래서 사용하지 않는 물건을 발견할 때마다 `천원경매 <https://1000won.auction>`_\
에 내다 팔고 있다. 천원경매는 사내 장터에서 경매로 물건을 팔기 위해 직접 만든
서비스이다.\ [3]_ 번개장터, 당근마켓, 중고나라 등 중고 거래 서비스가 이미 있긴
하지만, 상호 신뢰가 없는 사람들끼리 중고 거래를 하는 일이 결코 쉬운 일은 아니다.
하지만 회사 사람들끼리 거래를 한다면 소액의 물건을 가지고 서로 사기를 치는 일은
없을 것이라는 믿음이 있기 때문에 거래에 대한 부담감이 비교적 적다.

.. figure:: /attachments/2019/kakaopay-qrcode/jungonara.png
   :width: 240px

   중고 거래의 어려움

그리고 굳이 경매라는 판매 방식을 택한 이유는 중고 상품의 가격을 결정하는 일이
어렵기 때문이다. 내가 특정 가격을 제시했을 경우 팔리지 않을 수도 있고, 그렇게
되면 다시 적당히 가격을 내려서 다시 판매를 시도해야 한다. 물론 아주 저렴하게
내놓거나 무료로 올린다면 비교적 쉽게 처분할 수 있겠지만, 그래도 "일단 무료니까
받아놓고 생각해보자" 하는 사람보다는 해당 상품을 정말로 진지하게 원하는 사람이
가져갔으면 하는 바람이 있다. 자본주의 사회에서 가장 진지한 사람은 역시 돈을 가장
많이 내는 사람이다. 그런 면에서 경매라는 판매 방식은 내가 해결하고자 하는 문제를
모두 해결할 수 있다.

지금은 낙찰이 되었을 때 낙찰 금액과 입금 계좌 정보가 담긴 이메일 메시지가
자동으로 발송되도록 구성되어있다. 이메일을 받은 사람이 해당 계좌로 안내된 금액을
입금하면 판매자가 구매자에게 물건을 전달해주는 방식이다.

.. (TODO: 예제 화면 보여주기)

아직까지는 그런 일이 생기지 않았지만, 금액을 입력하는 일은 사람이 하는
일이다보니 실수를 할 가능성이 언제나 존재한다. 예를 들어서, 30,500원을 송금해야
하는데 30,050원을 송금하는 경우가 생길 수도 있다. 반대로, 낙찰 금액보다 큰
금액을 실수로 송금할 가능성도 있다. 물론, 모르는 사람과의 거래가 아니라 사내
거래이기 때문에 언제든지 차액 정정 거래를 할 수 있겠지만, 이러한 실수의 여지를
남겨두지 않기 위해서는 인간의 개입을 최소화 하는 것이 최선책이라는 생각이
들었다.

카카오페이
----------

`카카오페이 <https://www.kakaopay.com/>`_\ 는 송금, 인증, 청구서, 멤버십 관리
등을 편하게 해결할 수 있도록 도와주는 서비스이다. 나는 주로 친구들이나 직장
동료들끼리 밥값을 나눠 낼 때 사용한다.

며칠 전, 서비스의 이런저런 부분들을 살펴보다가 송금을 요청하는 기능이 있다는
것을 우연히 발견했다. 상대방이 나에게 바로 송금할 수 있도록 QR 코드를
생성해준다. 원하는 금액도 넣을 수 있는데, 금액을 넣으면 QR 코드를 찍었을 때 송금
UI에 그 금액이 미리 입력되어서 나온다. 이 부분을 천원경매에 이용하면 어떨까 하는
생각이 들었다.

.. figure:: /attachments/2019/kakaopay-qrcode/sample1.png
   :width: 320px

   카카오페이 송금 QR 코드

낙찰이 되었을 때 이메일로 무미건조하게 금액과 계좌번호를 텍스트로 표시하는 대신,
이메일 메시지에 QR 코드를 넣으면 편하게, 그리고 실수 없이 낙찰 대금을 송금할 수
있지 않을까 하는 생각이 들었다. 물론 카카오페이를 사용하지 않는 사람들도
있을 수 있으니 금액과 계좌번호는 여전히 표시를 해주어야 할 것이다.

카카오페이 유저 아이디와 금액을 매개변수로 전달했을 때 송금 QR 코드를 생성해주는
기능이 있다면 큰 어려움 없이 내가 생각하는 기능을 구현할 수 있을 것 같았다.

코딩 중에 최고는 안 코딩이다. 코드를 한 줄도 작성하지 않고 문제를 해결할 수
있다면 그게 최선의 해결책이라는 말이다. 그래서 카카오페이에서 개발자로 근무하고
있는 친구에게 슬쩍 물어봤다.

"혹시 이 QR 코드를 생성해주는 API를 제공하는가?"

아쉽게도 답변은 "제공하지 않는다." 였다. 어쩔 수 없다. 없으면 만들어야지.

QR 코드 분석
------------

카카오페이 송금 QR 코드는 크게 두 가지 타입이 있다.

1. 유저 아이디만 나타내는 QR 코드
2. 유저 아이디와 함께 금액이 임베딩(embedding) 된 QR 코드

1번 타입의 경우 스캔을 하면 돈을 보낼 사람의 이름과 함께 금액을 입력하는 UI가
나온다. 2번 타입은 금액이 미리 입력되어서 나온다.

개인 정보 보호를 위해서 QR 코드를 블러 처리했다. 스캔을 하지 않고 눈으로만
보기에도 2번 타입이 조금 더 많은 정보를 담고 있다는 것을 알 수 있었다.

.. figure:: /attachments/2019/kakaopay-qrcode/sample2.png
   :width: 640px

   타입 1 (왼쪽), 타입 2 (오른쪽)

1번 타입을 만드는건 어렵지 않다. 역시, 개인 정보 보호를 위해서 유저 아이디를
``0000...`` 으로 치환했다.

.. code::

   https://qr.kakaopay.com/000000000000000000000000

해당 URL로 접속하면 ``kakaopay://`` URL로 리다이렉트 하는 자바스크립트 코드가
나온다. 곧바로 ``kakaopay://``\ 로 보내지 않고 ``https://``\ 로 보내는 이유는
아마도 카카오톡이 설치되있지 않을 경우 앱스토어로 보내주기 위함일 것이다.

.. code::

   kakaotalk://kakaopay/money/to/qr?qr_code=000000000000000000000000

카카오톡이 설치된 모바일 폰에서 해당 URL을 열면 카카오페이 송금 UI가 바로
나타난다. 사실 여기까지만 해도 천원경매 사용자들이 카카오페이 메뉴를 열어서
판매자에게 송금하는 과정을 조금은 편하게 만들 수 있다.

하지만 내가 원하는건 2번 타입이다. 금액을 미리 입력해서 QR 코드를 발급할 수
있다면 사용자들의 실수를 방지할 수 있기 때문이다.

2번 타입 QR 코드에는 다음과 같은 값이 인코딩 되어있다.

.. code::

   kakaotalk://kakaopay/money/to/qr?qr_code=0000000000000000000000001f402302

유저 아이디 뒷 부분에 무언가 추가적인 데이터(``1f402302``)가 붙어있다. 나는
1,000원을 입력했는데, 그런것 치고는 굉장히 많은 양의 정보가 들어가 있다.

.. code::

   kakaotalk://kakaopay/money/to/qr?uid=000000000000000000000000&amount=1000

만약 이런 방식이었다면 일이 훨씬 수월했겠지만, 이 포스트에서 이야기 할 내용은
훨씬 짧아졌을 것이다. 어쩌면 아예 글을 쓰지 않았을지도 모른다.

잠깐 이야기가 옆으로 샐 뻔 했는데, 가장 중대한 문제는 같은 금액을 입력하더라도
매번 조금씩 다른 QR 코드가 생성된다는 점이었다. 유저 아이디 부분은 동일했지만,
그 뒤에 붙는 금액 데이터가 조금씩 달라졌다. 이유는 잘 모르겠지만 난수를 사용하는
것 같이 보였다. 아마도 금액 데이터를 생(plain)으로 노출시키지 않기 위함이
아니었을까.

하지만 암호화를 하지 않는 이상 특정한 규칙에 의해서 원본 데이터를 다른 데이터로
치환한 것에 불과하고, 어렵지 않게 규칙을 알아낼 수 있을 것 같았다.

- `Bitwise shift <https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.3.0/com.ibm.zos.v2r3.cbclx01/bitshe.htm>`_
- `Exclusive OR (XOR) <https://hackernoon.com/xor-the-magical-bit-wise-operator-24d3012ed821>`_
- `Bit (or byte) order reverse <https://stackoverflow.com/questions/2602823/in-c-c-whats-the-simplest-way-to-reverse-the-order-of-bits-in-a-byte>`_

그래봤자 이 중 하나겠거니 하는 마음으로 조금 더 깊이 들여다보기로 했다.

먼저, 금액을 1원으로 해서 바코드를 여러번 생성해봤다. 금액 데이터는 다음과 같다.

.. code::

   81686
   83780
   86466
   83840
   89480

이렇게 봐서는 뭐가 뭔지 하나도 모르겠다. 비트 단위로 표시를 해보면 어떤 패턴이
보이지 않을까?

.. code:: python

   >>> binary = lambda x: '{:b}'.format(x)
   >>> binary(0x81686)
   '10000001011010000110'

파이썬을 이용해서 16진수로 표시된 값을 바이너리 형식으로 표현해주는 한 줄 짜리
코드를 만들었다.

.. code::

   10000001011010000110
   10000011011110000000
   10000110010001100110
   10000011100001000000
   10001001010010000000

가장 앞쪽 비트(most significant bit)가 1이라는 점 말고는 이렇다할 패턴이 보이지
않았다. 사실, 금액을 1로 잡으면 2진수, 10진수, 16진수 등 무엇으로 보든 1로
보이기 때문에 이런 패턴을 분석할 때 좋은 샘플은 아니다.

이번에는 금액을 1씩 증가 시켜 가면서 금액 데이터가 어떻게 생성되는지 관찰해보기로 했다.

.. csv-table::
   :header: "금액", "QR 코드의 금액 데이터"
   :widths: 2, 6

   1, ``0x86222``
   2, (금액 데이터가 생성되지 않았다. 사용자 실수이거나 버그인 것 같다.)
   3, ``0x185920``
   4, ``0x202043``
   5, ``0x286900``

여전히 잘 모르겠다. 사실, 이때 저 데이터들을 바이너리로 표현해보기만 했어도
패턴을 금방 알아낼 수 있었을 것이다. 이때에는 비트 순서나 바이트 순서가 뒤바뀐
것을 의심하면서 이런저런 가설을 세우고 확인하는 과정을 거치고 있었다.

1원씩 증가시켜 가면서 만든 QR 코드를 분석하는 작업이 여의치 않아서 조금 더
패턴을 찾아보기 쉽게 2진수로 표현했을 때 1로만 구성된 숫자 몇가지를 샘플로
사용하기로 했다.

- 255 (2\ :sup:`8` - 1)
- 4,095 (2\ :sup:`12` - 1)
- 65,535 (2\ :sup:`16` - 1)
- 1,048,575 (2\ :sup:`20` - 1)
- 16,777,215 (2\ :sup:`24` - 1)\ [1]_

카카오페이 UI에서 위의 금액을 일일히 넣어서 QR 코드를 하나씩 생성했다. 그다지
아름답지 못한 성격의 지루한 작업이었지만, 별다른 방법이 없었다.

.. csv-table::
   :header: "금액", "QR 코드의 금액 데이터", "2진수 표현"

   "255", ``0x7f83200``, ``111111110000011001000000000``
   "4,095", ``0x7ff87241``, ``1111111111110000111001001000001``
   "65,536", ``0x7fff87321``, ``11111111111111110000111001100100001``
   "1,048,575", ``0x7ffff81305``, ``111111111111111111110000001001100000101``

이렇게 보니 패턴이 명확하게 보이기 시작했다. QR 코드의 금액 필드는 금액을 19칸
왼쪽으로 시프트 한 값에 무언가를 더한 값이었다. 금액 뒤에 붙은 데이터의 정체는
아직도 잘 모르겠다. 혹시 유효한 QR 코드인지 검사하는 에러 체킹 코드 같은 것이
아닐까 하는 생각도 했었는데, 아무렇게나 넣어도 작동이 되는 것으로 보아 그냥 랜덤
데이터인 것 같다.

.. figure:: /attachments/2019/kakaopay-qrcode/album.png
   :width: 320px

   계속된 실험으로 인해 QR 코드로 가득 찬 사진 앨범

이 글에서는 규칙을 알아내는 과정을 아주 간단하게 요약해서 표현했지만, 이걸
알아내느라 두어시간 동안 굉장히 많은 삽질을 했었다. 이렇게 난독화 된 데이터가
주어졌을 때 보다 효과적으로 패턴을 알아내는 과학적인 방법을 예전에 학교 다닐 때
암호학 수업 시간에 들은 기억이 있는데\ [2]_, 아쉽게도 기억이 잘 나지 않는다.
이번에는 운이 좋아서 큰 어려움 없이 규칙을 알아냈지만, 만약 다음번에 비슷한
문제에 봉착하게 되었는데 몇시간이 지나도 해결될 기미가 보이지 않는다면 그 부분을
다시 복기해봐야겠다.

QR 코드 생성
------------

카카오페이가 송금 QR 코드를 만들어내는 방식을 알아냈으니, 이제는 내 코드로 송금
QR 코드를 만들어 낼 차례이다. 

앞서 이야기 했듯이 카카오페이의 송금 URL은 다음과 같이 이루어져있다.

.. code::

   kakaotalk://kakaopay/money/to/qr?qr_code=${uid}${scrambled_amount}

사용자가 입력한 금액에 따라서 이 URL을 생성하고, 그것을 QR 코드로 만들면 되는
아주 간단한 작업이다. 나는 명령창 환경이 편하기 때문에 대부분의 작업은 vim
에디터를 띄워서 하는 편이지만, 화면에 무언가 보여줄 것이 있을 때에는 주피터
노트북을 이용해서 편하게 프로토타이핑을 할 수 있다. QR 코드 생성은 파이썬의
|qrcode|_ 패키지의 도움을 받았다.

.. |qrcode| replace:: ``qrcode``
.. _qrcode: https://pypi.org/project/qrcode/

.. image:: /attachments/2019/kakaopay-qrcode/qrcode-generation.png
   :width: 80%

그렇게 생성한 QR 코드를 폰에서 스캔 하면 다음과 같이 카카오페이 송금 화면이
뜬다. 코드에서 입력한 금액인 ``35,050``\ 원이 미리 입력되어서 송금 화면이 뜨는
것을 확인할 수 있었다.

.. image:: /attachments/2019/kakaopay-qrcode/kakaopay-send.png
   :width: 320px

마무리
------

아쉽게도 카카오페이에서 송금 QR 코드를 생성하는 API를 제공하지 않아 먼 길을
돌아왔지만, 비교적 큰 어려움 없이 QR 코드를 만들어내는 규칙을 파악할 수 있었고,
그 덕분에 원하는 기능을 만들 수 있었다. 물론, 이 기능이 천원경매 서비스에
들어가려면 아직 조금 더 작업해야 할 부분들이 남아있지만, 기본적인 기능을
구현하는데 필요한 사항들은 모두 마련한 상태라 큰 걱정은 없다. 조만간 천원경매
낙찰 안내 메시지에서 카카오페이 QR 코드를 볼 수 있을 것이다.

Notes
-----

.. [1] 이 값을 넣었다가 카카오페이에서 송금 가능한 최대 금액이 2,000,000원이라는 사실도 알게 되었다.
.. [2] `Introduction to Cryptography <https://www.amazon.com/Introduction-Cryptography-Coding-Theory-2nd/dp/0131862391>`_ 책으로 공부했었다.
.. [3] 천원경매 개발에 대한 자세한 이야기는 `PyCon 2017 발표 자료 <https://www.slideshare.net/suminb/pycon-2017-dollar-auction-78802984>`_\ 에서 찾아볼 수 있다.
