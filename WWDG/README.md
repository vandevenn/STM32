# 서론
WWDG(윈도우 와치독)을 활용해 MCU가 리셋되는것을 확인해보자.
# 본론
## 요구사항
WWDG(윈도우 와치독)을 활용해 기존에는 led가 토글되다가 MCU가 리셋되는것을 LED를 4초간 켜서 확인해보자.
## 설정
평소와 같이 RCC는 꺼주고
![](https://velog.velcdn.com/images/bburi406/post/27c05ba8-2eb3-465e-a804-74db437133f5/image.png)
WWDG에들어가 Active 해준다.
![](https://velog.velcdn.com/images/bburi406/post/61633f27-ac6e-4c20-a68c-64d94957bd1c/image.png)
그리고 그림과 같이 세팅에서 8,80,127로 수치를 설정해준다. 설정 이유는 코드에서 말하겠다.
![](https://velog.velcdn.com/images/bburi406/post/3c8ff6d1-39bd-4907-94e5-bb30b859ff5d/image.png)
클락이 32Mhz가 들어간 것을 확인하고 코드 생성해준다.
## Code
![](https://velog.velcdn.com/images/bburi406/post/b4762105-51bc-4617-9e96-4b966fa31d21/image.png)

데이터시트를 참고했을때 와치독 시간은 다음 공식과 같은데,
PCLK1 은 32MHz이므로 tPCLK1은 32MHz의 역수 인 31.25ns가 된다.

와치독 갱신 구간을 49.152~66.56(0x3F)ms 로 되도록 설정하겠다. 
그렇다면, prescaler는 8, t[5:0]은 127 - 80이 되므로 와치독 시간이 49.152가 되기때문에 설정에서 8, 80 , 127 을 설정한 것이다.

본 그래프를 해석하자면 W[6:0]값은 계속 떨어지게 되고, 0x3F값이 된다면 MCU가 리셋이된다. 그러므로 MCU에서 계속 W[6:0]값을 업데이트를 해주어 리셋이 되지 않도록 하고 만약 MCU에 이상이 생긴다면 W값을 업데이트를 못하게 되므로 리셋이 된다. 

**와치독은 계속 MCU나 CPU를 감시하며, 이상증상이 있어 신호를 못받게되면 리셋을 시킨다. 이는 프로그래머가 계속 관리해줄수 없는 임베디드 시스템에서 복구 방법으로 널리 쓰인다.**

```
  if (__HAL_RCC_GET_FLAG(RCC_FLAG_WWDGRST) != RESET)
  {
	  HAL_GPIO_WritePin (LD2_GPIO_Port, LD2_Pin, GPIO_PIN_SET);

	  HAL_Delay (4000);
	  __HAL_RCC_CLEAR_RESET_FLAGS();
  }
  else
  {
	  HAL_GPIO_WritePin (LD2_GPIO_Port, LD2_Pin, GPIO_PIN_RESET);
  }
```
유저버튼 B1을 눌렀을때 while(1)을 통해 계속 무한루프가 돌게끔 만들어 와치독을 실행하게 만들었다. 그리고, 플래그가 뜨면 LED를 4초간 켰다 끄고 다시 프로그램을 진행하는 방식으로 만들었다.
## Github
(https://github.com/vandevenn/STM32/tree/main/WWDG)
## 결과
[구현영상 <-클릭](https://blog.naver.com/bb406/223398408265)

영상이 안올라간다, Gif는 용량 초과라고 한다.. 구현영상은 네이버 블로그 참고

led가 토글되는게 기존 프로그램이라 생각하고, 유저 버튼을 누르자 while문에 같혀 무한루프를 돌게 되어 와치독이 실행된다. 그래서 led가 4초간 켜지었다가 꺼지고(Reset) 다시 기존프로그램으로 돌아가게된다.

## 느낀점
와치독이 어느때 쓰이는지 어떤개념인지 배우게 되었다. 
다만 역시 인터넷으로 검색했을때는 임베디드 시스템이면 
다 쓰이는거처럼 설명이 되어있지만 선임께 물어보자,
복구 방식중 하나이며 굳이 계속 감시한다면 과부화가 올수도 있고 여러가지로 안 좋을수도 있다며 쓰이는 곳에만 쓰인다고 한다.
그리고 hal_delay함수를 4초나 때려넣었더니 역시나 가끔가다 리셋이 안된다. 그냥 보드가 죽어버리는거같다. Timer를 만들어서 써야되는 이유를 또 느끼게 된다.
