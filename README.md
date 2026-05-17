[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/MCJunYEq)
[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=23647445&assignment_repo_type=AssignmentRepo)
# Lab06: Comunicación UART con PIC18F45K22

## Integrantes

[Samuel Forero](https://github.com/Sam232510)

[Danna Pineda](https://github.com/Danna-pineda)

## Documentación

En este laboratorio se implementó una comunicación serial entre un microcontrolador PIC18F45K22 y un computador utilizando el protocolo UART, con el objetivo de transmitir y visualizar datos en tiempo real. Inicialmente, se configuró el microcontrolador para enviar mensajes a través del puerto serial y se verificó la correcta comunicación mediante el software PuTTY, donde fue posible observar los datos recibidos desde el sistema embebido. Posteriormente, se desarrolló un programa en Python encargado de leer la información proveniente del UART y generar una gráfica a partir de los datos obtenidos, permitiendo así analizar de forma visual el comportamiento de las señales transmitidas y comprender la integración entre hardware y software en sistemas de adquisición de datos.

Luego de entender lo que se debe realizar pasaremos a describir los codigos utilizados para este laboratorio.

# Uart.c

    #include "uart.h"
    #include <stdio.h>

UART.H: este tiene las declaraciones de funciones y configuraciones relacionadas con la comunicación UART.

STUDIO.H: es la libreria de entrada y salida de datos en lenguaje C.

    TRISC6 = 0; 
    TRISC7 = 1;

TRISC6 = 0 configura el pin RC6/TX como salida para transmitir datos.
TRISC7 = 1 configura el pin RC7/RX como entrada para recibir datos.

    SPBRG1 = 25; 
    TXSTA1bits.BRGH = 0; 
    BAUDCON1bits.BRG16 = 0; 

SPBRG1 = 25 establece el valor del baudrate para trabajar a 9600 baudios usando un oscilador de 16 MHz.
BRGH = 0 selecciona el modo de baja velocidad.
BRG16 = 0 utiliza un generador de baudrate de 8 bits.

    RCSTA1bits.SPEN = 1; 
    TXSTA1bits.SYNC = 0; 
    TXSTA1bits.TXEN = 1; 
    RCSTA1bits.CREN = 1; 

SPEN = 1 habilita el módulo serial.
SYNC = 0 selecciona el modo asíncrono.
TXEN = 1 habilita la transmisión de datos.
CREN = 1 habilita la recepción continua de datos.

    PIE1bits.RC1IE = 1;  
    PIR1bits.RC1IF = 0;  
    INTCONbits.PEIE = 1;  
    INTCONbits.GIE = 1;   

RC1IE = 1 activa la interrupción cuando llega un dato por UART.
RC1IF = 0 limpia la bandera de interrupción.
PEIE = 1 habilita interrupciones de periféricos.
GIE = 1 habilita las interrupciones globales del microcontrolador.

    void UART_WriteChar(char data) {
    while (!TXSTA1bits.TRMT); // Espera a que se vacíe el buffer de transmisión
    TXREG1 = data;
    }

2Este bloque define una función encargada de transmitir un único carácter mediante comunicación UART.La función recibe como parámetro una variable llamada data de tipo char, que representa el carácter que será enviado serialmente.

    void UART_WriteString(const char* str) {
    while (*str) {
        UART_WriteChar(*str++);
    }
    }

Este bloque define una función utilizada para transmitir cadenas completas de texto mediante UART.La función recibe como parámetro un puntero llamado str, el cual apunta al inicio de una cadena de caracteres y recorre la cadena carácter por carácter mientras no se encuentre el carácter ('\0'), que indica el final del texto.


# Main.c

    #include <xc.h>
    #include "uart.h"

xc.h = contiene las definiciones y registros específicos del microcontrolador PIC.

uart.h = contiene las funciones y configuraciones relacionadas con la comunicación UART

    #pragma config FOSC = INTIO67
    #pragma config WDTEN = OFF
    #pragma config LVP = OFF

Este bloque establece los bits de configuración del microcontrolador.

    void main(void) {
    OSCCON = 0b01110000;  // Oscilador interno a 16MHz
    UART_Init();          // Inicializa UART

    while(1) {
        UART_WriteString("Hola, UART funcionando!\r\n");
        __delay_ms(1000); 
    }
    }

Este bloque corresponde a la función principal del programa, donde inicia la ejecución del microcontrolador. Primero, se configura el oscilador interno del PIC a una frecuencia de 16 MHz mediante el registro OSCCON, lo que define la velocidad de funcionamiento del sistema, despues se llama a la función UART_Init(), encargada de inicializar y configurar la comunicación serial UART. Después de realizar estas configuraciones, el programa entra en un ciclo infinito while(1), dentro del cual se envía continuamente el mensaje “Hola, UART funcionando!” a través del puerto serial utilizando la función UART_WriteString().

# lab6.py

    import serial
    import matplotlib.pyplot as plt
    import matplotlib.animation as animation
    import re
    from collections import deque

serial = permite establecer comunicación serial entre Python y el microcontrolador mediante UART.

matplotlib.pyplot = se utiliza para crear gráficas.

matplotlib.animation =  permite actualizar la gráfica en tiempo real.

re = se usa para trabajar con expresiones regulares y extraer datos específicos del texto recibido.

deque = estructura de datos que almacena elementos con un tamaño máximo definido.

    SERIAL_PORT = 'COM8'
    BAUDRATE = 9600

SERIAL_PORT = 'COM8' especifica el puerto COM utilizado para la comunicación con el microcontrolador.

BAUDRATE = 9600 establece la velocidad de transmisión de datos en 9600 baudios.

    MAX_POINTS = 100

Define la cantidad máxima de datos que se almacenarán y mostrarán en la gráfica en tiempo real.

    ser = serial.Serial(SERIAL_PORT, BAUDRATE, timeout=1)

timeout=1 indica que el programa esperará máximo un segundo por datos antes de continua

    voltages = deque(maxlen=MAX_POINTS)
    times = deque(maxlen=MAX_POINTS)
    time_counter = 0

voltages = almacena los valores de voltaje recibidos.

times = almacena los valores de tiempo correspondientes.

time_counter = contador utilizado como eje temporal de la gráfica.

    regex = re.compile(r"Voltaje:\s*([0-9.]+)")

Este bloque crea una expresión regular que busca textos con el formato:

    Voltaje: 3.25

La expresión permite extraer únicamente el valor numérico del voltaje recibido desde el microcontrolador.

    def update(frame):

Define la función encargada de leer los datos seriales y actualizar la gráfica en tiempo real.

    line = ser.readline().decode('utf-8').strip()

Lee una línea completa enviada por UART, la convierte de bytes a texto UTF-8 y elimina espacios o saltos de línea innecesarios.

    match = regex.search(line)

Busca dentro del texto recibido un valor que coincida con el formato definido en la expresión regular.

    if match:
        voltage = float(match.group(1))
        voltages.append(voltage)
        times.append(time_counter)
        time_counter += 1

match.group(1) = extrae el valor numérico detectado.

float() = convierte el texto a número decimal.

voltages.append(voltage) = guarda el voltaje recibido.

times.append(time_counter) = guarda el instante de tiempo.

time_counter += 1 = incrementa el contador temporal.

    ax.clear()
    ax.plot(times, voltages, color='green')
    ax.set_ylim(0, 5)
    ax.set_title("UART leyendo")
    ax.set_xlabel("Segundos")
    ax.set_ylabel("Voltaje")
    ax.grid(True)

ax.clear() = limpia la gráfica anterior.

ax.plot() = dibuja los datos de voltaje contra tiempo.

set_ylim(0,5) = fija el rango del eje Y entre 0 y 5 voltios.

set_title() = establece el título de la gráfica.

set_xlabel() y set_ylabel() = nombran los ejes.

grid(True) = activa la cuadrícula de referencia.

    fig, ax = plt.subplots()

Crea la figura y el sistema de ejes donde se mostrará la gráfica.

    ani = animation.FuncAnimation(fig, update, interval=1000, cache_frame_data=False)

update = función que se ejecutará continuamente.

interval=1000 = actualiza la gráfica cada 1000 ms (1 segundo).

cache_frame_data=False = evita almacenar cuadros anteriores en memoria.

    plt.tight_layout()
    plt.show()

tight_layout() = organiza automáticamente los elementos de la ventana gráfica.

show() = muestra la gráfica en pantalla y mantiene el programa en ejecución.


## Diagramas

![alt text](montaje.png)
Figura 1. Conexión Pic18f45k22-Uart-PC

## Evidencias de implementación

![alt text](uart1.gif)

En este pequeño video se observa el funcionamiento basico de la comunicación entre el el UART y el PC mediante la aplicación PuTTY. Logrando obtener el mensaje enviado desde el codigo.

![alt text](uart2.gif)

En este segundo video se evidencia el programa realizado en Python el cual es un lector de voltaje esta información no la da el UART y el programa en Python lo que nos ayuda es a graficar y verificar otro medio de comunicación que no sea a tra ves de el PuTTY.