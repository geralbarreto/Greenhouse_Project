# Firmware

## Descripción

El siguiente repositorio contiene la implementación del firmware del proyecto Invernadero Inteligente, en éste se establece la lógica utilizada por el microcontrolador Freescale MC9S08QE128 presente en la tarjeta de desarrollo DEMOQE128.

## Tabla de Contenidos
- [Lista de Módulos](#lista-de-módulos)
- [Requisitos](#requisitos)
- [Software](#software)

## Lista de Módulos

- [Firmware] (https://github.com/geralbarreto/Greenhouse_Project/blob/master/Firmware/Firmware_final/sendchar/Sources/ProcessorExpert.c)

En este módulo se configuran los componentes utilizados para la implementación del firmware, a su vez se utiliza una máquina de estados para recibir la data analógica y digital captada por la tarjeta y enviarla al ordenador vía serial mediante un protocolo de comunicaciones que se especifica a continuación:

     0xFN 0DDAAAAA 0AAAAAAA

     **N:** Numero Canales Mixtos
     **D:** Sensor Digital
     **A:** Sensor Analógico
     
Para el proyecto en cuestión se utilizan tres sensores analógicos por lo que se necesitan 3 canales mixtos cada uno de los cuales está dividido en 2 mitades de 8 bits.
     
Finalmente se utiliza el protocolo:   [11110011 0D1D2AAAAA 0AAAAAAA 0D30BBBBB 0BBBBBBB 000CCCCC 0CCCCCCC]
     
     **11110011:** Header para 3 canales
     **D1:** Medida digital del fotorresistor
     **D2:** Medida digital del higrómetro
     **D3:** Bit auxiliar para indicar si está lloviendo
     **A:** Medida del acelerómetro
     **B:** Medida del sensor de temperatura
     **C:** Medida del sensor ultrasónico

Para aplicar este protocolo a las medidas captadas se realizó el siguiente procedimiento:

La lectura del ADC es 										[AAAAAaaaaaaa0000]
Para tener solo los 12 bits se shiftean 4 bits a la derecha 					[AAAAAaaaaaaa0000]>>4 = [xxxxAAAAAaaaaaaa]
Para los bits más significativos se shiftean 7 bits a la derecha 					[xxxxAAAAAaaaaaaa]>>7 = [xxxAAAAA]&[00011111] = [000AAAAA]
Para los bits menos significativos se utiliza un AND 0x7F 	 		[xxxxAAAAAaaaaaaa]&[01111111] = [0aaaaaaa]
Para los sensores digitales se utiliza un OR con 0x40 o 0x20 en caso de que se detecte un 1	 [000AAAAA] |[01000000]= [010AAAAA]		[000AAAAA] |[00100000]= [001AAAAA]

     
- [Events.h](https://github.com/geralbarreto/Greenhouse_Project/blob/master/Firmware/Firmware_final/sendchar/Sources/Events.h) Módulo de eventos (.h) de Processor Expert.
- [Events.c](https://github.com/geralbarreto/Greenhouse_Project/blob/master/Firmware/Firmware_final/sendchar/Sources/Events.c) Módulo de eventos (.c) de Processor Expert.

La máquina de estados consiste en los estados esperar, medir y enviar, está controlada por interrupciones periódicas que se realizan cada 0.7ms, por lo tanto, se pasa del estado de espera al estado de medición con una frecuencia de 1.4KHz. Para realizar las mediciones del acelerómetro y el sensor de temperatura se utilizó el componente ADC disponible en la librería de Processor Expert que permite realizar la medición de distintos canales analógicos y devuelve el valor digital correspondiente en 12 bits. Para realizar las mediciones de la fotorresistencia y el higrómetro se utilizó el componente Bit que permite captar el valor digital medido por los pines de la tarjeta seleccionados. Finalmente, para realizar la medida del sensor ultrasónico se utilizó el componente Capture que permite medir el ancho de un pulso, esto es necesario debido a que el sensor ultrasónico emite un pulso y recibe un eco cuyo ancho es proporcional a la distancia a la cual se encuentra el objeto detectado. Cabe destacar que para activar el sensor ultrasónico fue necesario enviar un pulso de 10uS y período de 50ms lo cual se realizó implementando el componente PWM disponible en Processor Expert.

Finalmente, para implementar la toma de acciones del invernadero según la información recibida por los sensores se utilizan funciones sencillas que permitirán indicar, mediante los leds disponibles en el DEMOQE, si es necesario encender las luces, abrir las ventanas o abrir las válvulas de agua potable o de lluvia según sea el caso. En el primer caso se verifica el estado del sensor digital 1 (LDR), si éste es 0 es necesario encender las luces por lo que se enciende el led 1, para saber si es necesario abrir o cerrar las ventanas se compara el valor de la temperatura actual con respecto a uno preestablecido, por lo que si la temperatura sobrepasa el valor ideal se deben abrir las ventanas y se enciende el led 2. Para activar las válvulas de agua se verifica si el nivel del tanque está al nivel óptimo, si no es así se verifica si está lloviendo. Para notar la presencia de lluvia se utiliza la medida del acelerómetro, si esta aumenta y disminuye en un periodo corto de tiempo se asume que está lloviendo por lo que se activa una bandera que se envía por serial al computador como un sensor digital para utilizar esta información en el software de la interfaz gráfica.

## Requisitos

La implementación fue desarrollada mediante el uso de la tarjeta de desarrollo DEMOQE128 utilizando el Microcontrolador de Freescale Semiconductor: MC9S08QE128CLH y fue probada en Windows 7 de 32 y 64 bits

## Software

El software utilizado para la implementación fue CodeWarrior v10.5 + Processor Expert.
