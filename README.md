# Содержание
+ [Настройка 8/16 битного таймера/счетчика](#Timer)

+ [Настройка ШИМ сигнала на пине](#PWM)

+ [Настройка интерфейса USART](#USART)

+ [Настройка записи и чтения EEPROM](#EEPROM)

## <a name="Timer"></a>	**Настройка 8/16 битного таймера/счетчика**

#### 1. ***Инициализация***
```c++
    TCCR0B = 1 << CS01;
    TCNT0 = 56;
```
 где `TCCR0B` - Timer/Counter Control Register B,

`CS01` - Clock Select,

`TCNT0` - Timer/Counter Register.
***

Формула расчета частоты счетчика:

**f = F/ N * (TCNTmax - TCNT0)** , где

**N** - предделитель

**F** - частота кварца

#### 2. ***Прерывания***
#### 1. *Прерывание по переполнению*

Активация прерывания по переполнению:
    
```c++
    TIMSK0 = 1 << TOIE0;
```
где `TIMSK0` – Timer/Counter Interrupt Mask Register,

`TOIE0` - Timer/Counter0 Overflow Interrupt Enable.
***

Обработчик прерывания по переполнению:
```c++
    ISR(TIMER0_OVF_vect)
    {
         Код;
         TCNT0 = 56;
    }
```


## <a name="PWM"></a>	**Настройка ШИМ сигнала на пине**

#### 1. ***Инициализация***
```c
    DDRD = 1 << DDD6;
    TCCR0A = 1 << WGM00 | 1 << WGM01 | 1 <<  COM0A1;
    TCCR0B = 1 << CS00 | 1 << CS02;
```
где, `DDRD` – The Port D Data Direction Register,

`TCCR0A` – Timer/Counter Control Register A,

`WGM00` и `WGM01` - Waveform Generation Mode,

`COM0A1` - Compare Match Output A Mode,

`TCCR0B` – Timer/Counter Control Register B,

`CS00` и `CS02` - Clock Select.
***

#### 2. ***Использование***

Изменяем скважность ШИМ путем записывания в регистр OCR0A значений от 0 до 255:
```c
    OCR0A = 0...255;
```    
где `OCR0A` – Output Compare Register A.

## <a name="USART"></a>	**Настройка интерфейса USART**

### **Передача 1 байта**

#### 1. ***Инициализация***
```c
    UCSR0A = 1 << U2X0;
    UBRR0L = 16;
    UCSR0B = 1 <<  TXEN0;
```
где `UCSR0A` – USART Control and Status Register A,

`U2X0` - Double the USART Transmission Speed,

`UBRR0L` – USART Baud Rate Registers,

`UCSR0B` – USART Control and Status Register B,

`TXEN0` - Transmitter Enable.
***

#### 2. ***Использование***

Функция отправки данных:
```c
    void USART_Transmit( unsigned char data )
    {
	    while( !(UCSR0A & (1<<UDRE0)) );
	    UDR0 = data;
    }
```
где, `UCSR0A` – USART Control and Status Register A,

`UDRE0` - USART Data Register Empty,

`UDR0` – USART I/O Data Register.
***
### **Передача массива данных**

#### 1. ***Инициализация***
```c
    UCSR0A = 1 << U2X0;
    UBRR0L = 16;
    UCSR0B = 1 <<  TXEN0;
```
 где `UCSR0A` – USART Control and Status Register A,

`U2X0` - Double the USART Transmission Speed,

`UBRR0L` – USART Baud Rate Registers,

`UCSR0B` – USART Control and Status Register B,

`TXEN0` - Transmitter Enable.
***   

#### 2. ***Использование***

Функция отправки данных:
```c
    void USART_Trans(unsigned char data[], const int size)
    {
	    for (int i = 0; i < size; i++)
	    {
		while (!(UCSR0A & (1 << UDRE0)));
		UDR0 = data[i];
	    }
    }
```
где, `UCSR0A` – USART Control and Status Register A,

`UDRE0` - USART Data Register Empty,

`UDR0` – USART I/O Data Register.
***

### **Прием и передача массива данных**

#### 1. ***Инициализация***
```c
    UCSR0A = 1 << U2X0;
    UBRR0L = 16;
    UCSR0B = 1 <<  RXEN0 | 1 <<  TXEN0 | 1 << RXCIE0;

    TCCR0B = 1 << CS01 | 1 << CS00;
	TCNT0 = 56;
```
где `UCSR0A` – USART Control and Status Register A,

`U2X0` - Double the USART Transmission Speed,

`UBRR0L` – USART Baud Rate Registers,

`UCSR0B` – USART Control and Status Register B,

`TXEN0` - Transmitter Enable,

`RXEN0` - Receiver Enable,

`RXCIE0` - RX Complete Interrupt Enable,

`TCCR0B` – Timer/Counter Control Register B,

`CS00` и `CS01` - Clock Select,

`TCNT0` - Timer/Counter Register.
*** 

#### 2. ***Прерывания***

Обработчик прерывания по приему:
```C
    ISR(USART_RX_vect)
    {
	    TIFR0 = 1 << TOV0;
	    array[counter] = UDR0;
	    counter++;
	    TCNT0 = 56;
	    if ((TIMSK0 & (1 << TOIE0)) == 0) TIMSK0 = 1 << TOIE0;
	    if (counter == 100) counter = 0;
    }
```
где, `TIFR0` – Timer/Counter 0 Interrupt Flag Register,

`TOV0` - Timer/Counter0 Overflow Flag.
***
Обработчик прерывания по переполнению:
```c
    ISR(TIMER0_OVF_vect)
    {
        UCSR0B &= ~(1 << RXCIE0);
        TIMSK0 &= ~(1 << TOIE0);

        USART_Trans(uint8_t data[], uint16_t size);

        UCSR0B |= 1 << RXCIE0;
    
        counter = 0;
    }
```
где, `UCSR0B` – USART Control and Status Register B,

`RXEN0` - Receiver Enable,

`TIMSK0` – Timer/Counter Interrupt Mask Register,

`TOIE0` - Timer/Counter0 Overflow Interrupt Enable
***
#### 3. ***Использование***

Функция отправки данных:
```c
    void USART_Trans(unsigned char data[], const int size)
    {
	    for (int i = 0; i < size; i++)
	    {
		while (!(UCSR0A & (1 << UDRE0)));
		UDR0 = data[i];
	    }
    }
```
где, `UCSR0A` – USART Control and Status Register A,

`UDRE0` - USART Data Register Empty,

`UDR0` – USART I/O Data Register.
***

## <a name="EEPROM"></a>	**Настройка записи и чтения EEPROM**

#### ***Чтение***
```c
    unsigned char EEPROM_read(unsigned int uiAddress)
    {
        while(EECR & (1<<EEPE));
        EEAR = uiAddress;
        EECR |= (1<<EERE);
        return EEDR;
    }
```
где, `EECR` – The EEPROM Control Register,

`EEPE` - EEPROM Write Enable,

`EEAR` – The EEPROM Address Register,

`EERE` - EEPROM Read Enable,

`EEDR` – The EEPROM Data Register.
***

#### ***Запись***
```c
    void EEPROM_write(unsigned int uiAddress, unsigned char ucData)
    {
        while(EECR & (1<<EEPE));
        EEAR = uiAddress;
        EEDR = ucData;
        EECR |= (1<<EEMPE);
        EECR |= (1<<EEPE);
    }
 ``` 

 где, `EECR` – The EEPROM Control Register,

`EEPE` - EEPROM Write Enable,

`EEAR` – The EEPROM Address Register,  

`EEDR` – The EEPROM Data Register,

`EEMPE` - EEPROM Master Write Enable.
***