---
layout: single 

title: "[iOS]Core Bluetooth 활용하기#03 - 블루투스 데이터 통신하기"

category: iOS

tags : [iOS, swift, CoreBluetooth]

typora-copy-images-to: ../images/2021-06-27

---

**Core Bluetooth를 활용해 블루투스 데이터 통신을 구현해봅시다.**

</br></br>

###### 소개

안녕하세요. 지난 포스팅에서 [[iOS]Core Bluetooth 활용하기#02 - 블루투스 기기 검색하기](https://staktree.github.io/ios/iOS-Bluetooth-02-search-peripheral/)에 대해 다루었는데요. 오늘은 **iOS에서 Core Bluetooth를 이용하여 블루투스 데이터 통신**을 구현할 예정입니다. 오늘은 테스트를 위해 아두이노에서 블루투스를 사용하는 방법도 조금 다룰 예정이니 집중해주세요! 

</br></br>



###### Missions🦊

1. 블루투스 시리얼 만들기 - 데이터 통신 관련 코드 작성
2. 데이터 통신을 위한 UI 제작하기
3. 테스트를 위한 아두이노 코드 작성

</br></br>



오늘 할 일은 저번 포스팅에서 만들었던 블루투스 시리얼에 **데이터 통신을 위한 코드**를 추가하고, 이를 활용하는 **UI**를 만들어 **블루투스 주변기기와 직접 데이터를 주고 받는 것**입니다. 아두이노에 직접 코드를 업로드하여 iOS에서 메세지를 전송했을 때 응답하도록 할 예정이라 아두이노를 다루는 것도 조금 포함될 예정이에요. 그럼 먼저 블루투스 시리얼 파트부터 진행해봅시다!

</br></br>



###### Mission 1. 블루투스 시리얼 만들기 - 데이터 통신 관련 코드 작성

지난 번 포스팅에서 블루투스 주변기기를 검색하고 연결하는 코드를 작성하였는데요. 오늘은 대망의 데이터 통신을 구현해 볼 텐데요. 생각보다 간단한 편이라 이전의 내용을 보고 오셨다면 충분히 이해하고 넘어갈 수 있으실 거에요!

</br></br>

**BluetoothSerial.swift**

~~~swift
  /// String 형식으로 데이터를 주변기기에 전송합니다.
    func sendMessageToDevice(_ message: String) {
        guard bluetoothIsReady else { return }
        // String을 utf8 형식의 데이터로 변환하여 전송합니다.
        if let data = message.data(using: String.Encoding.utf8) {
            connectedPeripheral!.writeValue(data, for: writeCharacteristic!, type: writeType)
        }
    }
    
    /// 데이터 Array를 Byte형식으로 주변기기에 전송합니다.
    func sendBytesToDevice(_ bytes: [UInt8]) {
        guard bluetoothIsReady else { return }
        
        let data = Data(bytes: UnsafePointer<UInt8>(bytes), count: bytes.count)
        connectedPeripheral!.writeValue(data, for: writeCharacteristic!, type: writeType)
    }
    
    /// 데이터를 주변기기에 전송합니다.
    func sendDataToDevice(_ data: Data) {
        guard bluetoothIsReady else { return }
        
        connectedPeripheral!.writeValue(data, for: writeCharacteristic!, type: writeType)
    }

 // peripheral으로부터 데이터를 전송받으면 호출되는 메서드입니다.
    func peripheral(_ peripheral: CBPeripheral, didUpdateValueFor characteristic: CBCharacteristic, error: Error?) {
        // 전송받은 데이터가 존재하는지 확인합니다.
        let data = characteristic.value
        guard data != nil else { return }
        
        // 데이터를 String으로 변환하고, 변환된 값을 파라미터로 한 delegate함수를 호출합니다.
        if let str = String(data: data!, encoding: String.Encoding.utf8) {
            delegate?.serialDidReceiveMessage(message : str)
        } else {
            return
        }
    }
    
    func peripheral(_ peripheral: CBPeripheral, didWriteValueFor characteristic: CBCharacteristic, error: Error?) {
        // writeType이 .withResponse일 때, 블루투스 기기로부터의 응답이 왔을 때 호출되는 메서드입니다.
        // 필요한 로직을 작성하면 됩니다.
    }
    
    func peripheral(_ peripheral: CBPeripheral, didReadRSSI RSSI: NSNumber, error: Error?) {
        // 블루투스 기기의 신호 강도를 요청하는 peripheral.readRSSI()가 호출하는 메서드입니다.
        // 신호 강도와 관련된 코드를 작성합니다.
        // 필요한 로직을 작성하면 됩니다.
    }
~~~

</br></br>



**SendMessageToDevice(_ message : )는 string 형식으로, sendBytesToDevice(_ bytes: )는 byte 형식으로, sendDataToDevice(_ data :)는 data 형식으로 연결된 주변기기에 데이터를 전송**합니다. 사실 모두 각각의 자료형을 data 형식으로 변환하여 통신하는 메소드들인데요. 이 변환된 데이터는  **CBPeripheral의 writeValue(data, for: writeCharacteristic!, type: writeType)** 메서드를 통해서 전송됩니다. **파라미터**의 **data는 전송할 데이터**를 뜻하며, **writeCharacteristic은 연결된 주변기기의 서비스 내의 Characteristic**, **writeType은 메세지를 보냈을 때 응답이 있는지 없는지를 체크**하는 변수입니다. 여기서 **응답이 있는 경우**에는  **peripheral(_ peripheral: , didWriteValueFor characteristic: , error: )**가 호출되는데요. 여기서 **characteristic.value**를 체크하면 응답 내용을 확인할 수 있을 거라 추측됩니다. 저의 경우, 블루투스 기기가 .withoutResponse이기 때문에 정확한 확인을 하지 못했습니다. 이 점은 조금 아쉽네요. 그래서 저는 데이터를 전송한 후 따로 수신하는 방법을 사용하였습니다. **데이터를 전송받으면 peripheral(_ peripheral: , didUpdateValueFor characteristic: , error: )** 라는 메서드가 호출됩니다. 여기서 characteristic.value를 String 형식으로 변환하여 어떤 데이터인지 확인할 수 있는데요. 이 데이터를 delegate의 함수의 파라미터로 넘겨 delegate에서 UILabel으로 표시하도록 했습니다. 그럼 이 delegate 함수를 정의하기 위해 다음 미션으로 넘어가시죠! 

</br></br>



###### Mission2. 데이터 통신을 위한 UI 제작하기

<img src="/images/2021-06-27/1.PNG" alt="1" style="zoom: 67%;" />

</br></br>



먼저 제작할 UI를 stotyboard에 꾸며보았는데요. 저번 포스팅에서 scan 버튼을 넣었던 View에 **Send Message 버튼과 전송 받은 메세지를 보여줄 UILabel**을 추가하였습니다. UILabel 뒤에 UIImage를 추가하여 텍스트가 더 잘 보이도록 하였습니다.

</br></br>



**SerialViewController.swift**

~~~swift
		
		/// 전송받은 메세지를 표시하는 Label
		@IBOutlet weak var serialMessageLabel: UILabel!    

    /// send Message버튼을 클릭하면 호출되는 메서드입니다. 주변기기에 데이터를 전송합니다.
    @IBAction func sendMessageButton(_ sender: Any) {
        if !serial.bluetoothIsReady {
            print("시리얼이 준비되지 않음")
            return
        }
        // 시리얼의 delegate를 SerialViewController로 설정합니다.
        serial.delegate = self
        // msg를 설정하고 이를 연결된 Peripheral에 전송하는 메서드를 호출합니다.
        let msg = "1"
        serial.sendMessageToDevice(msg)
        // 라벨의 텍스트를 변경하여 데이터를 기다리는 중이라는 것을 표현합니다.
        serialMessageLabel.text = "waiting for Peripheral's messege"
    }
    
    //MARK: 시리얼에서 호출되는 Delegate 함수들
    
    /// 데이터가 전송된 후 Peripheral로 부터 응답이 오면 호출되는 메서드입니다.
    func serialDidReceiveMessage(message : String)
    {
        // 응답으로 온 메시지를 라벨에 표시합니다. 
        serialMessageLabel.text = message
    }
}
~~~

</br></br>



SerialViewController에 추가된 코드들만 표시하였는데요. 먼저 **SerialMessageLabel에 전송받은 메세지를 표시하는 UILabel**을 연결해주세요. 그리고 **Send Message 버튼에 sendMessageButton(_ sender :)**를 연결해주세요. 여기서는 주변기기에 "1"이라는 데이터를 보냅니다. 위에서 설명한 sendMessageToDevice(_ message :)에 파라미터로 전송할 메세지를 전달하면 주변기기로 데이터가 전송됩니다. 그 다음 데이터를 기다리고 있다는 표시로 SerialMessageLabel의 text를 message를 기다고 있다는 의미로 변경해줍니다. 

</br></br>



**BluetoothSerial.swift**

~~~swift
protocol BluetoothSerialDelegate : AnyObject {
    func serialDidDiscoverPeripheral(peripheral : CBPeripheral, RSSI : NSNumber?)
    func serialDidConnectPeripheral(peripheral : CBPeripheral)
    func serialDidReceiveMessage(message : String)
}

// 프로토콜에 포함되어 있는 일부 함수를 옵셔널로 설정합니다.
extension BluetoothSerialDelegate {
    func serialDidDiscoverPeripheral(peripheral : CBPeripheral, RSSI : NSNumber?) {}
    func serialDidConnectPeripheral(peripheral : CBPeripheral) {}
    func serialDidReceiveMessage(message : String) {}
}
~~~

</br></br>



이제 데이터를 전송받으면, **전송 받은 메세지를 SerialMessageLabel에 나타나도록** 하여야하는데요. **Mission1**에서 작성했던 **delegate 메서드**인**serialDidReceiveMessage(message : )**로 구현됩니다. 이 메서드는 serial에서 **데이터가 전송받을 때마다 호출되는 peripheral(_ peripheral: , didUpdateValueFor characteristic: , error: )**에 의해서 호출되게 되는데, **serialMessageLabel에 전송받은 message를 표시**합니다. 마지막으로 **serialDidReceiveMessage(message :)**를 bluetoothSerialDelegate에 등록하면 iOS에서 할 일은 끝입니다. 

</br></br>



###### Mission3. 테스트를 위한 아두이노 코드 작성

여기서부터는 아두이노를 위한 코드입니다. 테스트만을 위해 작성한 코드이므로 매우 간단합니다.

</br></br>



~~~c++
#include <SoftwareSerial.h>

const int rxPin = 3;
const int txPin = 4;

SoftwareSerial bluetooth(rxPin, txPin);

void setup() {
   Serial.begin(9600);
   bluetooth.begin(115200);
}

void loop() {
  if(bluetooth.available())
    {
       char c = (char)bluetooth.read();

       if(c == '1')
          bluetooth.write("Success");
       Serial.print(c);
    } 

}
~~~

</br></br>



아두이노에 **전원이 들어오기 시작하면 setup()**이 호출되고, **그 이후로 loop()가 반복적**으로 호출됩니다. **SoftwareSerial bluetooth(rxPin, txPin)**은 아두이노 내에서 **bluetooth를 사용하기 위한 Serial**이라고 생각하시면 되는데, **rxPin과 txPin에 블루투스 모듈인 HM-10이 연결**되어 있습니다. Setup()에서 **bluetooth 모듈에 설정된 보드 레이트(통신 속도)로 begin()**을 하게 되면 모듈과 아두이노가 연동되어 작동됩니다. 이제 데이터를 받았을 때 특정한 메세지를 전달하면 되겠죠? 먼저 **bluetooth.read()**를 이용하여 수신한 데이터를 읽어봅니다. **데이터가 '1'이라면** 성공적으로 데이터를 읽었으므로 **"Success"**라는 메세지로 응답하도록 합니다. Serial.print(c)는 전달받은 데이터를 아두이노 IDE의 시리얼 모니터에 나타나다록 하는 코드입니다. 

</br></br>



###### 결과

이제 테스트를 해볼 시간인데요. **Send Message 버튼을 클릭하면, 연결된 기기에 '1'이라는 메세지를 보내게되고, 연결된 기기인 아두이노는 '1'이라는 메세지를 받으면 "Success"라는 메세지로 응답**합니다. 성공적으로 데이터 전송이 이루어졌다면 **serialMessageLabel에 응답받은 "Success"**가 나타나야하는데요. 그럼 한번 테스트 해보겠습니다. 

</br></br>



<img src="images/2021-06-27/2.png" alt="2" style="zoom:50%;" />

</br></br>



Scan 버튼을 눌러 Mission 3에서 코드를 업로드했던 아두이노 기기를 연결한 후, Send Message 버튼을 눌러봅시다!그럼 아래의 "message from Peripheral"가 "Success"로 바뀌어야 할 것입니다.

</br></br>



<img src="/images/2021-06-27/3.png" alt="3" style="zoom:50%;" />





짜잔!성공적으로 바뀌었네요.짝짝짝~😍





같은 결과를 얻으셨다면 정말 잘하셨습니다!!!여기까지가 저의 iOS Core Bluetooth에 대한 포스팅이었는데요. 저의 능력이 부족하여 잘 이해가 되셨는지는 모르겠습니다. 하지만  iOS에서 블루투스를 활용하여 여러가지 재미있는 서비스를 만드는데 조금이라도 도움이 되었으면 좋겠습니다. 다음에는 조금 더 발전된 포스팅으로 찾아오겠습니다!





**소스코드 :** https://github.com/staktree/iOSBluetoothSample





### 참고

[애플공식문서 CoreBluetooth](https://developer.apple.com/documentation/corebluetooth)

[애플공식문서 Core Bluetooth Programming Guide](https://developer.apple.com/library/archive/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/AboutCoreBluetooth/Introduction.html#//apple_ref/doc/uid/TP40013257)

[CoreBluetooth 예제 참고 GitHub](https://github.com/hoiberg/HM10-BluetoothSerial-iOS/blob/master/HM10%20Serial/ScannerViewController.swift) 
