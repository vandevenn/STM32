# 서론
UART통신을 통해 MCU 내부온도를 ADC기능을 통해 알아보자.
# 본론
## 요구사항
NUCLEO 보드내 2개의 12비트 ADC를 활용하여 내부의 온도센서를 이용해 MCU 내부 온도를 측정 한다.
측정한 값을 UART로 출력해 확인 한다.
## 설정
![](https://velog.velcdn.com/images/bburi406/post/a9e61a82-aed1-4703-9edf-501cf4ae5d0c/image.png)
ADC1으로 가서 TSC를 켜준다.
![](https://velog.velcdn.com/images/bburi406/post/48085bb6-8c46-4959-ac9f-687c2d082181/image.png)
아래에 ADC setting에서 Continuous Conversion Mode를 Enable해준다.
![](https://velog.velcdn.com/images/bburi406/post/dfcff24a-ea5f-45eb-b498-df6db54b44c0/image.png)

ADC Regular - Rank에서 Sampling Time을 13.5 cycles로 변경해준다.
![](https://velog.velcdn.com/images/bburi406/post/28eac2d2-8f38-4b16-9d29-f7a01ab50081/image.png)

그리고 Clock설정에서 자동설정을 OK 해주고
![](https://velog.velcdn.com/images/bburi406/post/0724fae8-8c08-415b-9d57-6d834444fa0d/image.png)
본 그림과 같이 PLLMul을 x16, ADC를 /8해서 64Mhz를 사용해되 8Mhz가 들어가게끔 설정한다.

그리고 코드를 생성하고, Uart printf문에서 사용한 포팅 코드를 옮긴다.
https://velog.io/@bburi406/STM32-UART-Printf-%EC%82%AC%EC%9A%A9 참조
## Code
```
if (HAL_ADCEx_Calibration_Start(&hadc1) != HAL_OK)
  {
	  Error_Handler();
  }
  /* Start the conversion process */
  if (HAL_ADC_Start(&hadc1) != HAL_OK)
  {
	  Error_Handler();
  }

  uint16_t adc1;

  float vSense;
  float temp;

  while (1)
  {
	  HAL_ADC_PollForConversion(&hadc1, 100);
	  adc1 = HAL_ADC_GetValue(&hadc1);
	  //printf ("ADC1_temperature: %d \n", adc1);
	  vSense = adc1 * ADC_TO_VOLT;
	  temp = (V25 - vSense) / AVG_SLOPE + 25.0;
	  printf ("temperature: %d, %f \n", adc1, temp);
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
	  HAL_Delay(100);
  }
  /* USER CODE END 3 */
}
```
ADC를 사용하기위한 Calibration code를 if문으로 선언해준다. 
![](https://velog.velcdn.com/images/bburi406/post/336ea686-a8e9-47ab-9e82-a524a779c548/image.png)

그리고 온도센서의 Datasheet를 보았을때 온돗값 계산하는 공식이 나와있는걸 참조한다.
![](https://velog.velcdn.com/images/bburi406/post/4580f5c9-f2fc-4968-aed3-7f24f952e354/image.png)
MCU 온도센서 특성도 참조한다.

참조한 정보를 이용해 온도 계산에 필요한 상숫값을 정의한다.
```
const float AVG_SLOPE = 4.3E-03;
const float V25 = 1.43;
const float ADC_TO_VOLT = 3.3 / 4096;
```
그리고 main에 loop문에 출력하는 코드를 입력한다.
다만, printf문에는 실수 출력이 되도록 옵션을 추가해야 하므로 
Project - Properties - c/c++ Build - Settings -Tool Settings -MCU Setting에서 
![](https://velog.velcdn.com/images/bburi406/post/2cd93f2a-d205-4ef9-a8b4-b27adac1feea/image.png)
그림과 같이 use float with printf를 체크하고 apply 해준다.
## Github
[vandevenn/STM32/ADC_TemperatureSensor](https://github.com/vandevenn/STM32/tree/main/ADC_TemperatureSensor)
## 결과
![](https://velog.velcdn.com/images/bburi406/post/80133220-1e5e-4237-8c35-c25c8d5799cc/image.png)
UART통신을 통해 MCU 내부온도를 ADC기능을 통해 알아볼 수 있었다.
## 느낀점
본 프로젝트를 통해 ADC란 무엇인가(아날로그신호를 디지털로 전환) 그것을 이용해 MCU 온도를 측정을 하며, UART로 출력을 해보았다. 뭐든지 측정하면 얼마인지 출력을 하는게 중요하다고 배웠다, 통신과 자료구조등의 학습의 필요성을 느꼇고, Datasheet를 보는법을 배웠지만 항상 외울수도 없고 똑같은것만 보지 않기 때문에 필요한 부분이있다면 찾을수 있는 능력
그능력을 기르는게 필요하다 느꼇다.