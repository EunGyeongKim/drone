# 파이썬을 이용하여 드론 GCS(Ground Control Station) 구현하기
### 구현기간 : 2018.09.10. ~ 2019.02.28.(총 6개월) 
### 사용된 기술
- Python : PC GCS 구현 및 통신
- 라즈베리파이 : 센서 및 부품 작동

## 🤷왜 유해동물 퇴치를 드론 GCS을 만들었는가?
평소 IoT와 임베디드에 관심을 가지다 보니 자연스럽게 드론에 대해 호기심이 생겼다. 드론에 관해 이것저것 찾아보다가 전자부품연구원에서 주관하여 진행하는 '드론 SW개발 전문가 과정'을 발견하게 되었고, 드론의 HW와 SW구현에 참여하게 되었다. 드론을 어떻게 하면 효율적이며 획기적으로 이용할 수 있을지에 대해 고민하다가 평소 농사를 지으시는 아버지가 멧돼지 및 유해동물의 출몰이 늘어 농작물 피해가 늘어난다는 이야기를 듣게 되었다. 
![image](https://user-images.githubusercontent.com/26224573/192410088-4b06a553-451d-4f02-9e16-277a2cd79a7e.png)
이를 조사하여 보니, 매년 유해동물로 인한 농작물 피해가 늘고 있으며, 2018년도 9월기준 1382건에 달한다는 통계를 찾게 되었다. 기존 유해동물을 막는 방법으로는 전기 울타리가 있지만, 전기 울타리를 사용할 경우 유해동물이 아닌 사람의 피해또한 일어난다는 뉴스 보도 기사 또한 발견하게 되었다. 
그래서 농민 및 농작물의 피해를 줄이기 위해 드론을 이용하여 유해동물을 퇴치하자는 아이디어가 나와 프로젝트에 착수하게 되었다. 
![image](https://user-images.githubusercontent.com/26224573/192410148-0719295c-8e53-4e8f-8465-7762f466380c.png)
![image](https://user-images.githubusercontent.com/26224573/192413804-9130e047-8cab-4eed-a1e2-db6f748efe32.png)

지상관제시스템인 GCS을 이용해 순찰할 웨이포인트를 설정하고 열감지 카메라로 유해동물을 탐지하여 유해동물 퇴치 스프레이를 뿌려 퇴치하는 드론이다. 

## 기존 퇴치 방법
![image](https://user-images.githubusercontent.com/26224573/192409639-c3ea1e57-5142-4adb-9536-941c8dcde898.png)
현재 키프리스에 등록된 유해동물 퇴치기중 하나이다. 이 특허는 소리감지 센서를 이용해 동물의 소리를 감지하고, 동물의 소리가 감지된 부분으로 스피커를 돌려 소리를 출력하는 퇴치기이다. 
하지만 동물이 소리를 내지 않는다면 탐지 불가능 하며, 바람소리, 자동차 소리 등 생물체 판별 불가능하다. 또한 소리에 익숙해진 유해동물은 도망가지 않는다는 단점이 있다. 

![image](https://user-images.githubusercontent.com/26224573/192412631-7462ff3e-b055-44d8-a5eb-a778a6c9cb99.png)
다음 키프리스에 등록된 유해동물 퇴치기는 열감지 센서를 이용하여 유해동물을 퇴치하는 특허이다. 이 특허는 무선 통신으로 스피커들 제어하여 동물 감지 시 cctv 녹화한다. 만약 유해동물이 감지된경우 메시지를 설정한 연락처에 전송하여 사용자가 유해동물의 접근을 인지하게 한 후, 주위에 설치된 스피커를 이용해 소리를 출력하여 유해동물을 퇴치하는 퇴치기이다. 
하지만 이 퇴치기인 경우 열감지 센서로 사용하므로 멀리 있는 생물체는 판별 불가능하며, 소리에 익숙해진 유해동물은 도망가지 않는다.

![image](https://user-images.githubusercontent.com/26224573/192409854-14178a02-5ecc-4c85-9965-ecb458b9ce18.png)
이 기존 제품의 문제점을 해결하기 위해 저희 드론을 다방면에서 생각하게 되었다. 

판별하기 힘든 동물의 소리 대신 열감지 카메라로 유해동물을 감지후 소리에 익숙해진 동물들을 확실하게 내쫓기 위해 곰도 퇴치할수 있는 스프레이를 부착하였다. 
또한 상공 5미터에서 드론이 비행하기 때문에 높은 유해동물 탐지율 및 장해물에 의한 방해를 받지 않는다. 
그리고 공중에서 순찰하기 떄문에 사각지대를 최대한 줄여 효율적인 순찰 및 탐지를 할 수 있다.
또한 GCS로 실시간 감시 및 드론 자율운행이 가능하기 때문에 드론을 이용한 유해동물 퇴치를 고안하게 되었다. 

## 드론 HW / SW 구현

![image](https://user-images.githubusercontent.com/26224573/192410017-cbee6808-68c6-4bd1-8ae2-a6a90bab17b9.png)
![image](https://user-images.githubusercontent.com/26224573/192410621-90fa1220-8d75-41d3-8a9f-902fe9c0f97b.png)

드론의 대략적의 순서는 다음과 같다. 
- 드론에 부착된 라즈베리 파이와 FC(Flight Controller), 라즈베리파이와 GCS 동기화(wifi 통신)한다. 
- PC의 GCS로 웨이포인트 설정 및 드론에 웨이포인트 데이터 전달한다.
- 설정된 웨이포인트로 드론이 순찰하며, 유해동물 감지 시 경고음 및 스프레이를 분사한다.
- 또한 드론과 PC의 GCS는 문제상황 보고(생명체 감지) 및 퇴치명령을 전달하여 실시간 감시가 가능하도록 하였다.

### 드론 SW 프로그램

![image](https://user-images.githubusercontent.com/26224573/192412526-0cacadc4-587b-4676-b1e4-bf4a33c09a60.png)

### 드론 HW 구현
![image](https://user-images.githubusercontent.com/26224573/192410796-6f5ba37f-18e4-4858-9a52-110be9f8f1c1.png)
![image](https://user-images.githubusercontent.com/26224573/192411157-bf25f00f-ea2f-455b-b5bb-69cc9c564c32.png)


## 결과 

![경상대 스마트팜 아이디어 공모전 장려상](https://user-images.githubusercontent.com/26224573/192411108-efe67567-d2b6-46a8-a867-87d386c009fb.jpg)

![한밭대 드론 챔피언쉽 특별상](https://user-images.githubusercontent.com/26224573/192411116-b5ecca4d-0e97-4e1a-9692-df8a8005e24b.png)

- 경상대 스마트팜 아이디어 공모전과 한밭대 드론 챔피언쉽에서 장려상과 특별상을 수상하였다. 

## 느낀점

- 아두이노, 라즈베리파이, Flight Control, Python을 이용하여 PC에서 드론을 컨트롤할수 있는 프로젝트를 진행해서 너무너무 즐겁고 재밌었다. 
- 프로젝트를 진행하면서 파이썬을 배우고 실습까지 진행하여 정말 유익한 시간이 되었다.
- 또한, 드론의 구조와 구동방법을 알게되어 굉장히 유익하였다. 
- 통신이 약간 난잡하여 정리하고 싶었지만, 시간이 없어 굉장히 아쉬웠다.
  - FC와 라즈베리파리, 라즈베리파이와 센서들(스프레이 가동부분 + 부저 가동부분), PC GCS와 열화상카메라로 총 3개의 통신을 사용하였다. 추후 통신을 통합하여 간소화 하고 싶다.
- 열화상 카메라를 이용하여 해당 데이터를 pyhton GCS와 연동하였으나, 유해동물인지 사람인지 판단해주는 부분은 구현하지 못했다.
  - 이는 딥러닝 이미지분석을 이용하여 인간인지 동물인지 분류하였으면 좋았겠지만, 인공지능까지 학습하기에는 시간이 너무 모자랐다. 




