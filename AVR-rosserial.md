# Использование rosserial с AVR и UART

## 1. Исходники

Исходники лежат в папке src, либо их можно достать из rosserial_arduino:
```
rosrun rosserial_arduino make_libraries.py
```
## 2. Подготовка
### 2.1 AVR Development Environment

Для сборки исходников необходимо установить следующее утилиты и библиотеки
```
sudo apt-get install binutils-avr avr-libc gcc-avr avrdude
```
### 2.2 Железо
Гайд написан для atmega2560, но можно использовать и с другими AVR. Для этого нужно изменить Makefile и регистры в модулях avr_time и avr_uart.
Связь осуществляется через UART преобразователь встроенный в плату arduino.
## 3. Загрузка исходников 

### 3.1 rosserial_arduino
Для работы rosserial используются библиотеки rosserial_arduino. Можно их достать самостоятельно, либо взять из папки src.

### 3.2 Дополнительные модули
Также дополнительно необходимы следующие модули:

- avr_time - использует прерывание для подсчета количества переполнений таймера с момента запуска программы и конвертирует его в миллисекунды.

- avr_uart - настраивает UART AVR и может отправлять и получать данные.

- Atmega2560Hardware.h - заголовочный файл для работы нашего контроллера с rosserial.

- ros.h - typedef для удобства.

## 4. Написание узла
Пример написания узла публикации приведен ниже. 

```
#include "ros.h"
#include "std_msgs/String.h"
// Include C headers (ie, non C++ headers) in this block
extern "C"
{
  #include <util/delay.h>
}

// Needed for AVR to use virtual functions
extern "C" void __cxa_pure_virtual(void);
void __cxa_pure_virtual(void) {}

ros::NodeHandle nh;

std_msgs::String str_msg;
ros::Publisher chatter("chatter", &str_msg);

char hello[13] = "hello world!";

int main()
{
  uint32_t lasttime = 0UL;

  // Initialize ROS
  nh.initNode();
  nh.advertise(chatter);

  while(1)
  {
    // Send the message every second
    if(avr_time_now() - lasttime > 1000)
    {
      str_msg.data = hello;
      chatter.publish(&str_msg);
      lasttime = avr_time_now();
    }
    nh.spinOnce();
  }

  return 0;
}
```


В целом синтаксис практически не отличается от написания узлов для ардуино, за исключением использования задержки посредствам самостоятельно написанной библиотеки avr_time.c

## 6. Makefile

Для сборки прошивки и программирования используется Makefile, который также можно найти в исходниках. Он содержит много кода, но, по существу, нас интересуют только следующие  строки:

Имя целевого файла (без расширения):
```
TARGET = avr_chatter 
```
Исходные файлы C и C++ соответсвенно:
```
SRC =avr_time.c avr_uart.c
CPPSRC = $(TARGET).cpp ros_lib/time.cpp ros_lib/duration.cpp
```

Стандарт с++
```
CPPFLAGS += c++11
```

## 7. Сборка прошивки

Откройте терминал в директории rosserial_avr_tutorial/src и выполните make.

Если сборка прошла удачно, должен появиться файл avr_chatter.hex.\
Далее при помощи make program можно прошить контроллер, при условии что вы используете usbasp и atmega 2560. В противном случае прошиваем удобным для вас способом, либо лезем в Makefile и пишем свою make program.

## P.S
Гайд основан на [этой](http://wiki.ros.org/rosserial_client/Tutorials/Using%20rosserial%20with%20AVR%20and%20UART) статье, но там используются старые исходники. Не пытайтесь их использовать, они для старой версии ROS!