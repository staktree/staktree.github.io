---
layout: single 

title: "[iOS]CoreBluetooth 활용하기#01 - BLE와 Core Bluetooth란 무엇일까요?"

category: iOS

tags : [iOS, swift, CoreBluetooth]

typora-copy-images-to: ../images/2021-05-31

---

**BLE와 Core Bluetooth에 대해 알아봅시다.**





###### **소개**

안녕하세요. **이번 시간에는 BLE와 iOS에서 블루투스를 다루는 프레임워크인 Core Bluetooth에 대해 알아보겠습니다.** iOS에서 블루투스를 사용하는 방법을 검색해보면 BLE와 CoreBluetooth에 대한 정보가 많이 나오는데요. 제가 느끼기에는 어렵고 복잡한 부분이 많았습니다. 그래서 제가 공부한 내용을 바탕으로 조금 쉽게 정리해보면 어떨까하여 이렇게 포스팅을 작성하게 되었습니다. 그럼 이제 BLE와 CoreBluetooth가 어떤 것인지 알아보러가볼까요!    





###### **BLE와 CoreBluetooth는 무엇일까?**

먼저 **BLE는 Bluetooth Low Energy**의 약자이며, 블루투스 통신의 단점이었던 전력소비를 보완한 저전력 블루투스입니다. 통신 속도보다 전력 소모를 줄이는 것에 초점을 맞춘 Bluetooth 4.0에 특화된 기술이라고 하네요. **Core Bluetooth는 이 기술을 쉽게 사용할 수 있도록 애플에서 제공하는 프레임워크입니다.** BLE의 기술 스택을 추상화하여 개발자들이 BLE 기술에 대해 자세하게 알고 있지 않아도 BLE를 활용한 앱을 쉽게 개발할 수 있도록 도와줍니다!





###### CoreBluetooth에 대해 자세히 알아봅시다.

![1](/Users/tak/Documents/staktree.github.io/1.png)

[애플공식문서](https://developer.apple.com/documentation/corebluetooth)를 보면 CoreBluetooth에 대해서 조금 더 이해할 수 있습니다! CoreBluetooth에 대한 개요를 한번 읽어봅시다.  





**Core Bluetooth 프레임 워크는 BLE와 BR/EDR 무선 통신 기술을 장비한 블루투스와 애플리케이션이 통신하는데 필요한 다양한 클래스를 제공합니다.** 하지만 CoreBluetooth가 다른 클래스들의 하위 클래스로 사용될 수 없다는 점을 주의해야합니다. 오버라이딩된 이러한 클래스들은 지원되지 않으며 이는 정의되지 않은 행동을 초래한다고 되어있습니다. 추가적으로 CoreBluetooth의 Background 실행은 iPad와 macOS에서 동작하는 애플리케이션에서는 지원되지 않으며, iOS에서만 가능하다고 합니다. 





[애플공식문서](https://developer.apple.com/documentation/corebluetooth) 스크롤을 내려보면 CoreBluetooth와 관련된 **Topic**들을 확인할 수 있는데요. **Central, Peripheral, Service, 데이터전송**에 대한 설명과 관련된 클래스와 프로토콜들에 대한 내용이 있는 것을 확인할 수 있습니다. 여기서 설명하고 있는 클래스와 프로토콜은 CoreBluetooth가 동작하는데 핵심적인 역할을 합니다. 그럼 **Central**, **Peripheral**, **Service**에서 이 핵심적인 역할들이 어떻게 이루어져있는지 알아봅시다! 여기서 부터가 정말 중요한 내용이니 집중해주세요🤩





###### Central과 Peripheral는 Core Blutooth의 핵심!

**Central은 중앙**, **Peripheral은 주변**이라는 사전적 의미를 갖고 있습니다. 즉, **Peripheral는 흔히 우리가 iOS에 연결하는 주변 기기이고, Central은 이를 제어하는 iOS 애플리케이션을 의미합니다.** (물론, Core Bluetooth는 iOS 애플리케이션이 Peripheral로 동작하는 경우도 지원합니다. )**Master-Slave** 방식으로 설명하자면 **Central은 Master**의 역할을 하고 **Peripheral은 Slave**의 역할을 수행한다고 이해하면 될 거 같아요.





예를 들어 **방의 온도를 제어하는 서비스**가 있다면 **디지털 온도계가 Peripheral의 역할을 수행하며, 측정한 온도를 Central인 애플리케이션에 전달합니다.**



![2](/Users/tak/Documents/staktree.github.io/images/2021-05-11/2.png)





**Central은 Peripheral을 discover(검색)하고 connect(연결)을 요청합니다. 그리고 연결한 Peripheral에 대한 exploring(탐색)을 시작하고, Peripheral의 데이터와 interacting(상호작용)을 합니다.**  **이 때, Peripheral은 Central이 discover(검색)할 수 있도록 advertising(광고)합니다** (공식문서의 내용을 그대로 적어보았는데요. 반복적으로 등장하는 단어들은 그대로 사용하였습니다.  discover은 주변에 통신이 가능한 기기들이 있는지 찾아보는 것이고, exploring은 연결된 기기의 데이터를 지속적으로  탐색하는 의미라고 파악됩니다. advertising은 자신의 존재가 있다고 나타내는 것이라고 해석됩니다)**.** Cetral의 역할에서 이를 수행하는 클래스와 프로토콜은 다음과 같습니다. 





######  class [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager)

주변기기를 검색, 연결, 관리하기 위한 하나의 오브젝트입니다. 

###### class [CBPeripheral](https://developer.apple.com/documentation/corebluetooth/cbperipheral)

주변기기를 나타내며 이를 통해 데이터를 송수신합니다.

###### protocol [CBCentralManagerDelegate](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate)

주변기기를 탐색하고 관리하기 위해 발생되는 Update를 지원하는 프로토콜입니다.

###### protocol [CBPeripheralDelegate](https://developer.apple.com/documentation/corebluetooth/cbperipheraldelegate)

주변기기의 서비스들을 사용하기 위해 발생되는 Update를 지원하는 프로토콜입니다. 





**CBCentralManager는 Central에서 수행하는 discover(검색), connect(연결)를 담당하는 클래스**입니다. **검색된 Peripheral들을 CBPeripehral 객체로 관리하며Peripheral에 대한 연결을 수행합니다. 연결된 Peripheral은 CBPeripheral로 저장되며 이를 통해 데이터 통신이 수행됩니다. ** **이 모든 작업들은 CBCentralManagerDelegate과 CBPeripheralDelegate에 의해 지원**됩니다. 





###### Periphral의 데이터 구조와 서비스에 대해서

데이터 통신에 대해 더 자세히 알기 위해서는 Peripheral의 데이터 구조에 대해 알아볼 필요가 있습니다. **Peripheral의 데이터는 1개 이상의 Service로 이루어져있습니다. 또, Service는 1개 이상의 Characteristic을 가질 수 있습니다.** 이 때 **Service는** **특정 데이터를 측정하거나 특정 기기를 제어**하는 것입니다. 심장 박동을 모니터링하는 Service나 방의 온도를 제어하는 Service를 예로 들 수 있습니다. **즉, Service는 Peripheral이 가지고 있는 하나의 기능입니다.** 애플공식문서에는 **서비스는 수행하는 기능을 위한 행동이나 데이터의 집합**이라고 되어있네요. **Service는 보통 Characteristic을 갖고 있는데, Characteristic은 Peripheral이 가지고 있는 실질적인 데이터를 나타냅니다.** 심장 박동을 모니터링 하는 서비스에서는 측정된 심장 박동수, 방의 온도를 제어하는 서비스에서는 측정된 온도라고 할 수 있겠네요. 하나의 서비스는 다른 서비스를 포함할 수 있으며, 여러개의Characteristic을 가지는 것도 가능합니다.  심장 박동 모니터링 Service라면 심장 박동수와 센서의 위치와 같이 여러개의 characteristic을 포함할 수 있습니다.



![3](/Users/tak/Documents/staktree.github.io/3.png)





이와 관련된 것들도 Central, Peripheral과 마찬가지로 Topic의 Service에 정의된 클래스들을 활용하여 동작합니다!





###### class [CBService](https://developer.apple.com/documentation/corebluetooth/cbservice)

장치의 특성 또는 기능을 수행하는 관련 동작과 데이터의 컬렉션입니다. 

###### class [CBCharacteristic](https://developer.apple.com/documentation/corebluetooth/cbcharacteristic)

원격 주변기기 서비스의 Characteristic입니다. 





CBCentralManager에서 Peripheral을 연결할 때, **Peripheral의 Service들을 검색하고, 다시 Service 내의 Characteristic을 검색**합니다. 여기서 Service와 Characeteristic 각각을 **CBService와 CBCharacteristic 객체로 관리**합니다. CBCharacteristic 객체는 CBPeripheral 객체에서 데이터를 전송하기 위한 메서드로 활용되며, 이는 특정 Service의 Characteristic으로 값이 전달되도록 합니다. 





###### 정리

이번 시간에는 iOS에서 블루투스 연결을 위해 주로 BLE 모듈을 사용한다는 것과 이를 Core Bluetooth로 구현한다는 내용을 알아보았습니다. 또, Core Bluetooth에는 Central과 Peripherl의 역할이 있고, 어떤 클래스와 프로토콜들이 이를 구현하는지 알아보았습니다. 다시 한번 요약해보면 이렇습니다.





1. Central은 중앙 제어 기기, Peripheral은 주변 기기이며 CoreBluetooth에서는 이를 클래스로 추상화하여 표현한다. 
2. CBCentralManager를 통해 블루투스 주변기기를 검색하고 연결하며, 주변기기들은 CBPeripheral로 관리된다.
3. CBPeriphral은 주변기기와의 데이터 통신을 담당한다. 
4. 데이터는 service와 charateristic으로 구조화된다. Service는 특정 기능을 수행하는 하나의 단위이고, Charateristic은 Sevice에서 활용하는 실제 데이터이다. 





다음 시간에는 이를 직접 코딩해보며 Bluetooth 시리얼을 만들고, 이를 활용하여 블루투스 기기를 검색하는 내용을 다루어 보겠습니다. 





도움이 되셨길 바라며 저는 물러가보겠습니다. 다음 시간에 만나요~





### 참고

[애플공식문서 CoreBluetooth](https://developer.apple.com/documentation/corebluetooth)

[애플공식문서 Core Bluetooth Programming Guide](https://developer.apple.com/library/archive/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/AboutCoreBluetooth/Introduction.html#//apple_ref/doc/uid/TP40013257)

[BLE 관련 블로그](https://enidanny.github.io/ble/what-is-the-ble/) 