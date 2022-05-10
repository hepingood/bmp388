[English](/README.md) | [ 简体中文](/README_zh-Hans.md) | [繁體中文](/README_zh-Hant.md) | [日本語](/README_ja.md) | [Deutsch](/README_de.md) | [한국어](/README_ko.md)

<div align=center>
<img src="/doc/image/logo.png"/>
</div>

## LibDriver BMP388

[![MISRA](https://img.shields.io/badge/misra-compliant-brightgreen.svg)](/misra/README.md) [![API](https://img.shields.io/badge/api-reference-blue.svg)](https://www.libdriver.com/docs/bmp388/index.html) [![License](https://img.shields.io/badge/license-MIT-brightgreen.svg)](/LICENSE)

BMP388은 입증된 감지 원리를 기반으로 하는 압력 및 온도 측정 기능이 있는 디지털 센서입니다. 센서 모듈은 풋프린트가 2.0 x 2.0mm2에 불과하고 패키지 높이가 최대 0.8mm인 초소형 10핀 금속 덮개 LGA 패키지에 들어 있습니다. 작은 크기와 3.4 uA@1Hz의 낮은 전력 소비로 휴대폰, GPS 모듈 또는 시계와 같은 배터리 구동 장치에 구현할 수 있습니다.

LibDriver BMP388은 LibDriver에서 출시한 BMP388의 전체 기능 드라이버입니다. 압력 및 온도의 지속적인 판독, 압력 및 온도의 단일 판독, 인터럽트 판독, FIFO 수집 및 기타 기능을 제공합니다. LibDriver는 MISRA를 준수합니다.

### 콘텐츠

  - [설명](#설명)
  - [설치](#설치)
  - [사용](#사용)
    - [example basic](#example-basic)
    - [example shot](#example-shot)
    - [example interrupt](#example-interrupt)
    - [example fifo](#example-fifo)
  - [문서](#문서)
  - [기고](#기고)
  - [저작권](#저작권)
  - [문의하기](#문의하기)

### 설명

/src 디렉토리에는 LibDriver BMP388의 소스 파일이 포함되어 있습니다.

/interface 디렉토리에는 LibDriver BMP388용 플랫폼 독립적인 IIC, SPI버스 템플릿이 포함되어 있습니다.

/test 디렉토리에는 LibDriver BMP388드라이버 테스트 프로그램이 포함되어 있어 칩의 필요한 기능을 간단히 테스트할 수 있습니다.

/example 디렉토리에는 LibDriver BMP388프로그래밍 예제가 포함되어 있습니다.

/doc 디렉토리에는 LibDriver BMP388오프라인 문서가 포함되어 있습니다.

/datasheet 디렉토리에는 BMP388데이터시트가 있습니다.

/project 디렉토리에는 일반적으로 사용되는 Linux 및 마이크로컨트롤러 개발 보드의 프로젝트 샘플이 포함되어 있습니다. 모든 프로젝트는 디버깅 방법으로 셸 스크립트를 사용하며, 자세한 내용은 각 프로젝트의 README.md를 참조하십시오.

### 설치

/interface 디렉토리에서 플랫폼 독립적인 IIC, SPI버스 템플릿을 참조하여 지정된 플랫폼에 대한 IIC, SPI버스 드라이버를 완성하십시오.

/src 디렉토리, /interface 디렉토리 및 /example 디렉토리를 프로젝트에 추가하십시오.

### 사용

#### example basic

```C
uint8_t res;
uint8_t i;
float temperature_c;
float pressure_pa;

res = bmp388_basic_init(BMP388_INTERFACE_IIC, BMP388_ADDRESS_ADO_LOW);
if (res != 0)
{
    return 1;
}

...

for (i = 0; i < 3; i++)
{
    bmp388_interface_delay_ms(1000);
    res = bmp388_basic_read((float *)&temperature_c, (float *)&pressure_pa);
    if (res != 0)
    {
        (void)bmp388_basic_deinit();

        return 1;
    }
    bmp388_interface_debug_print("bmp388: temperature is %0.2fC.\n", temperature_c);
    bmp388_interface_debug_print("bmp388: pressure is %0.2fPa.\n", pressure_pa);

    ...
        
}

...

(void)bmp388_basic_deinit();

return 0;
```

#### example shot

```C
uint8_t res;
uint8_t i;
float temperature_c;
float pressure_pa;

res = bmp388_shot_init(BMP388_INTERFACE_IIC, BMP388_ADDRESS_ADO_LOW);
if (res != 0)
{
    return 1;
}

...

for (i = 0; i < 3; i++)
{
    bmp388_interface_delay_ms(1000);
    res = bmp388_shot_read((float *)&temperature_c, (float *)&pressure_pa);
    if (res != 0)
    {
        (void)bmp388_shot_deinit();

        return 1;
    }
    bmp388_interface_debug_print("bmp388: temperature is %0.2fC.\n", temperature_c);
    bmp388_interface_debug_print("bmp388: pressure is %0.2fPa.\n", pressure_pa);
    
    ...
    
}

...

(void)bmp388_shot_deinit();

return 0;
```

#### example interrupt

```C
uint8_t res;
uint16_t i, timeout;
uint8_t gs_data_ready_flag;
float gs_temperature_c;
float gs_pressure_pa;

void bmp388_interrupt_receive_callback(uint8_t type)
{
    switch (type)
    {
        case BMP388_INTERRUPT_STATUS_FIFO_WATERMARK :
        {
            break;
        }
        case BMP388_INTERRUPT_STATUS_FIFO_FULL :
        {
            break;
        }
        case BMP388_INTERRUPT_STATUS_DATA_READY :
        {
            /* read temperature pressure */
            if (bmp388_interrupt_read((float *)&gs_temperature_c, (float *)&gs_pressure_pa) != 0)
            {
                bmp388_interface_debug_print("bmp388: read temperature and pressure failed.\n");
           
                return;
            }
            gs_data_ready_flag  = 1;
            
            break;
        }
        default :
        {
            break;
        }
    }
}

res = gpio_interrupt_init();
if (res != 0)
{
    return 1;
}
res = bmp388_interrupt_init(BMP388_INTERFACE_IIC, BMP388_ADDRESS_ADO_LOW, bmp388_interrupt_receive_callback);
if (res != 0)
{
    (void)gpio_interrupt_deinit();

    return 1;
}

...

gs_data_ready_flag = 0;
timeout = 5000;
for (i = 0; i < 3; i++)
{
    while (timeout != 0)
    {
        bmp388_interface_delay_ms(100);
        timeout--;
        if (gs_data_ready_flag != 0)
        {
            break;
        }
        if (timeout == 0)
        {
            (void)gpio_interrupt_deinit();
            (void)bmp388_interrupt_deinit();

            return 1;
        }
    }
    gs_data_ready_flag = 0;
    timeout = 5000;
    bmp388_interface_debug_print("bmp388: temperature is %0.2fC.\n", gs_temperature_c);
    bmp388_interface_debug_print("bmp388: pressure is %0.2fPa.\n", gs_pressure_pa);
    
    ...
    
}

...

(void)gpio_interrupt_deinit();
(void)bmp388_interrupt_deinit();

return 0;
```

#### example fifo

```C
uint8_t gs_fifo_full_flag;
uint8_t gs_fifo_watermark_flag;
uint16_t i, timeout;
uint8_t gs_buf[512];
bmp388_frame_t gs_frame[256];

void bmp388_fifo_receive_callback(uint8_t type)
{
    switch (type)
    {
        case BMP388_INTERRUPT_STATUS_FIFO_WATERMARK :
        {
            uint8_t res;
            uint16_t len;
            uint16_t i, frame_len;
            
            len = 512;
            frame_len = 256;
            res = bmp388_fifo_read(gs_buf, len, (bmp388_frame_t *)gs_frame, (uint16_t *)&frame_len);
            if (res != 0)
            {
                bmp388_interface_debug_print("bmp388: fifo read failed.\n");
                
                return;
            }
            for (i = 0; i < frame_len; i++)
            {
                if (gs_frame[i].type == BMP388_FRAME_TYPE_TEMPERATURE)
                {
                    bmp388_interface_debug_print("bmp388: fifo %d/%d.\n", i+1, frame_len);
                    bmp388_interface_debug_print("bmp388: temperature is %0.2fC.\n", gs_frame[i].data);
                }
                else if (gs_frame[i].type == BMP388_FRAME_TYPE_PRESSURE)
                {
                    bmp388_interface_debug_print("bmp388: fifo %d/%d.\n", i+1, frame_len);
                    bmp388_interface_debug_print("bmp388: pressure is %0.2fPa.\n", gs_frame[i].data);
                }
                else if (gs_frame[i].type == BMP388_FRAME_TYPE_SENSORTIME)
                {
                    bmp388_interface_debug_print("bmp388: fifo %d/%d.\n", i+1, frame_len);
                    bmp388_interface_debug_print("bmp388: sensortime is %d.\n", gs_frame[i].raw);
                }
                else
                {
                    bmp388_interface_debug_print("bmp388: fifo %d/%d.\n", i+1, frame_len);
                    bmp388_interface_debug_print("bmp388: unknow type.\n");
                }
            }
            gs_fifo_watermark_flag = 1;
            
            break;
        }
        case BMP388_INTERRUPT_STATUS_FIFO_FULL :
        {
            uint8_t res;
            uint16_t len;
            uint16_t i, frame_len;
            
            len = 512;
            frame_len = 256;
            res = bmp388_fifo_read(gs_buf, len, (bmp388_frame_t *)gs_frame, (uint16_t *)&frame_len);
            if (res != 0)
            {
                bmp388_interface_debug_print("bmp388: fifo read failed.\n");
                
                return;
            }
            for (i = 0; i < frame_len; i++)
            {
                if (gs_frame[i].type == BMP388_FRAME_TYPE_TEMPERATURE)
                {
                    bmp388_interface_debug_print("bmp388: fifo %d/%d.\n", i+1, frame_len);
                    bmp388_interface_debug_print("bmp388: temperature is %0.2fC.\n", gs_frame[i].data);
                }
                else if (gs_frame[i].type == BMP388_FRAME_TYPE_PRESSURE)
                {
                    bmp388_interface_debug_print("bmp388: fifo %d/%d.\n", i+1, frame_len);
                    bmp388_interface_debug_print("bmp388: pressure is %0.2fPa.\n", gs_frame[i].data);
                }
                else if (gs_frame[i].type == BMP388_FRAME_TYPE_SENSORTIME)
                {
                    bmp388_interface_debug_print("bmp388: fifo %d/%d.\n", i+1, frame_len);
                    bmp388_interface_debug_print("bmp388: sensortime is %d.\n", gs_frame[i].raw);
                }
                else
                {
                    bmp388_interface_debug_print("bmp388: fifo %d/%d.\n", i+1, frame_len);
                    bmp388_interface_debug_print("bmp388: unknow type.\n");
                }
            }
            gs_fifo_full_flag = 1;
            
            break;
        }
        case BMP388_INTERRUPT_STATUS_DATA_READY :
        {
            break;
        }
        default :
        {
            break;
        }
    }
}

res = gpio_interrupt_init();
if (res != 0)
{
    return 1;
}
res = bmp388_fifo_init(BMP388_INTERFACE_IIC, BMP388_ADDRESS_ADO_LOW, bmp388_fifo_receive_callback);
if (res != 0)
{
    (void)gpio_interrupt_deinit();

    return 1;
}

...

gs_fifo_watermark_flag = 0;
gs_fifo_full_flag = 0;
timeout = 5000;
for (i = 0; i < 3; i++)
{
    while (timeout != 0)
    {
        bmp388_interface_delay_ms(100);
        timeout--;
        if ((gs_fifo_watermark_flag != 0) || (gs_fifo_full_flag != 0))
        {
            break;
        }
        if (timeout == 0)
        {
            (void)gpio_interrupt_deinit();
            (void)bmp388_fifo_deinit();

            return 1;
        }
    }
    gs_fifo_watermark_flag = 0;
    gs_fifo_full_flag = 0;
    timeout = 5000;
    
    ...
    
}
(void)gpio_interrupt_deinit();
(void)bmp388_fifo_deinit();

...

return 0;
```

### 문서

온라인 문서: https://www.libdriver.com/docs/bmp388/index.html

오프라인 문서: /doc/html/index.html

### 기고

연락주세요lishifenging@outlook.com

### 저작권

저작권 (c) 2015 - 지금 LibDriver 판권 소유

MIT 라이선스(MIT)

이 소프트웨어 및 관련 문서 파일("소프트웨어")의 사본을 얻은 모든 사람은 이에 따라 무제한 사용, 복제, 수정, 통합, 출판, 배포, 2차 라이선스를 포함하여 소프트웨어를 처분할 수 있는 권리가 부여됩니다. 소프트웨어의 사본에 대한 라이선스 및/또는 판매, 그리고 소프트웨어가 위와 같이 배포된 사람의 권리에 대한 2차 라이선스는 다음 조건에 따릅니다.

위의 저작권 표시 및 이 허가 표시는 이 소프트웨어의 모든 사본 또는 내용에 포함됩니다.

이 소프트웨어는 상품성, 특정 목적에의 적합성 및 비침해에 대한 보증을 포함하되 이에 국한되지 않는 어떠한 종류의 명시적 또는 묵시적 보증 없이 "있는 그대로" 제공됩니다. 어떤 경우에도 저자 또는 저작권 소유자는 계약, 불법 행위 또는 기타 방식에 관계없이 소프트웨어 및 기타 소프트웨어 사용으로 인해 발생하거나 이와 관련하여 발생하는 청구, 손해 또는 기타 책임에 대해 책임을 지지 않습니다.

### 문의하기

연락주세요lishifenging@outlook.com