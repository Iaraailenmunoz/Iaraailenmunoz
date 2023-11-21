import requests
import RPi.GPIO as GPIO
import time
from bs4 import BeautifulSoup
import threading

# URL del sitio web de pronóstico del tiempo
WEATHER_URL = 'https://weather.com/es-US/tiempo/horario/l/1eca6317682bf481bff726a0890265b49317aca51b90a10b66e50c4f9ab6168c?par=samsung_widget&cm_ven=shortprecip_ins_nus_es_1&partner=samsung_widget'

# Configuración de pines GPIO para LEDs individuales
LED_PIN_1 = 21
LED_PIN_2 = 24
LED_PIN_3 = 17  
LED_PIN_4 = 27  

# Configuración de la Raspberry Pi
GPIO.setmode(GPIO.BCM)
GPIO.setup(LED_PIN_1, GPIO.OUT)
GPIO.setup(LED_PIN_2, GPIO.OUT)
GPIO.setup(LED_PIN_3, GPIO.OUT)
GPIO.setup(LED_PIN_4, GPIO.OUT)

# Función para obtener datos meteorológicos desde el sitio web
def get_weather():
    try:
        response = requests.get(WEATHER_URL)
        response.raise_for_status()

        soup = BeautifulSoup(response.text, 'html.parser')
        
        # Agrega una impresión para ver el HTML de la página
        print(response.text)

        weather_description = soup.find('span', class_='fR3Kx')
        
        if weather_description:
            return weather_description.text.lower()
        else:
            print('No se encontró la descripción del tiempo en la página.')
            return None
    except requests.exceptions.RequestException as e:
        print('Error de solicitud:', str(e))
        return None
    except Exception as e:
        print('Error inesperado al obtener datos:', str(e))
        return None
    
# Función para controlar los LEDs individuales según el clima
def controlar_leds_segun_clima(weather_description):
    if weather_description:
        if 'lluvia' in weather_description:
            # Lluvia (LED verde)
            GPIO.output(LED_PIN_1, GPIO.HIGH)
            GPIO.output(LED_PIN_2, GPIO.HIGH)
            GPIO.output(LED_PIN_3, GPIO.LOW)   
            GPIO.output(LED_PIN_4, GPIO.LOW)  
        elif 'soleado' in weather_description:
            # Soleado (LED amarillo)
            GPIO.output(LED_PIN_1, GPIO.HIGH)
            GPIO.output(LED_PIN_2, GPIO.LOW)
            GPIO.output(LED_PIN_3, GPIO.HIGH)  
            GPIO.output(LED_PIN_4, GPIO.LOW)
        elif 'ventoso' in weather_description:
            # Ventoso (LED rojo)
            GPIO.output(LED_PIN_1, GPIO.HIGH)
            GPIO.output(LED_PIN_2, GPIO.LOW)
            GPIO.output(LED_PIN_3, GPIO.LOW)   
            GPIO.output(LED_PIN_4, GPIO.HIGH)  
        elif 'nublado' in weather_description:
            # Nublado (LED blanco)
            GPIO.output(LED_PIN_1, GPIO.LOW)
            GPIO.output(LED_PIN_2, GPIO.HIGH)
            GPIO.output(LED_PIN_3, GPIO.LOW) 
            GPIO.output(LED_PIN_4, GPIO.HIGH) 
        else:
            # Otro (LED azul)
            GPIO.output(LED_PIN_1, GPIO.LOW)
            GPIO.output(LED_PIN_2, GPIO.LOW)
            GPIO.output(LED_PIN_3, GPIO.LOW)
            GPIO.output(LED_PIN_4, GPIO.LOW)   
    else:
        print('No se obtuvo la descripción del tiempo, no se pueden controlar los LEDs.')

# Configurar LEDs en un estado predeterminado (apagados)
GPIO.output(LED_PIN_1, GPIO.LOW)
GPIO.output(LED_PIN_2, GPIO.LOW)
GPIO.output(LED_PIN_3, GPIO.LOW)
GPIO.output(LED_PIN_4, GPIO.LOW)

# Función para actualizar datos meteorológicos en segundo plano
def actualizar_datos_meteorologicos(intervalo_actualizacion):
    while True:
        weather_description = get_weather()
        controlar_leds_segun_clima(weather_description)
        time.sleep(intervalo_actualizacion)

# Hilo para la actualización de datos meteorológicos
intervalo_actualizacion = 600  # 10 minutos
weather_thread = threading.Thread(target=actualizar_datos_meteorologicos, args=(intervalo_actualizacion,))
weather_thread.daemon = True
weather_thread.start()

# Bucle principal
while True:
    pass

# Limpieza de GPIO
GPIO.cleanup()
