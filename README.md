import requests
import RPi.GPIO as GPIO
import time
from bs4 import BeautifulSoup
import traceback
import threading

# Configuración de pines GPIO para LEDs individuales
LED_PIN_1 = 23
LED_PIN_2 = 24  # 

# Configuración de la Raspberry Pi
GPIO.setmode(GPIO.BCM)
GPIO.setup(LED_PIN_1, GPIO.OUT)
GPIO.setup(LED_PIN_2, GPIO.OUT)

# URL del sitio web de pronóstico del tiempo
WEATHER_URL = 'https://weather.com/es-AR/tiempo/hoy/l/-53.79,-67.70?par=google'

# Función para obtener datos meteorológicos desde el sitio web
def get_weather():
    try:
        response = requests.get(WEATHER_URL)
        response.raise_for_status()

        soup = BeautifulSoup(response.text, 'html.parser')
        weather_description = soup.find('div', class_='today_nowcard-phrase').text.lower() 
        return weather_description
    except requests.exceptions.RequestException as e:
        log_error('Error de solicitud:', str(e))
    except Exception as e:
        log_error('Error inesperado al obtener datos:', traceback.format_exc())
    return None

# Función para controlar los LEDs individuales según el clima
def controlar_leds_segun_clima(weather_description):
    if weather_description:
        if 'lluvia' in weather_description:
            # Lluvia (LED verde)
            GPIO.output(LED_PIN_1, GPIO.HIGH)  # LED 1 encendido (verde)
            GPIO.output(LED_PIN_2, GPIO.HIGH)  # LED 2 encendido (verde)
        elif 'soleado' in weather_description:
            # Soleado (LED amarillo)
            GPIO.output(LED_PIN_1, GPIO.HIGH)  # LED 1 encendido (amarillo)
            GPIO.output(LED_PIN_2, GPIO.LOW)   # LED 2 apagado
        elif 'ventoso' in weather_description:
            # Ventoso (LED rojo)
            GPIO.output(LED_PIN_1, GPIO.HIGH)  # LED 1 encendido (rojo)
            GPIO.output(LED_PIN_2, GPIO.LOW)   # LED 2 apagado
        elif 'nublado' in weather_description:
            # Nublado (LED blanco)
            GPIO.output(LED_PIN_1, GPIO.LOW)   # LED 1 apagado
            GPIO.output(LED_PIN_2, GPIO.HIGH)  # LED 2 encendido (blanco)
        else:
            # Otro (LED azul)
            GPIO.output(LED_PIN_1, GPIO.LOW)   # LED 1 apagado
            GPIO.output(LED_PIN_2, GPIO.LOW)   # LED 2 apagado
    else:
        log_error('No se obtuvo la descripción del tiempo, no se pueden controlar los LEDs.')

# Función para registrar errores en un archivo de registro
def log_error(message, error_message):
    with open('error_log.txt', 'a') as log_file:
        log_file.write(f'{message}\n{error_message}\n\n')

# Configurar LEDs en un estado predeterminado (apagados)
GPIO.output(LED_PIN_1, GPIO.LOW)
GPIO.output(LED_PIN_2, GPIO.LOW)

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

try:
    while True:
        pass 

except KeyboardInterrupt:

    GPIO.cleanup()
except Exception as e:
    log_error('Error inesperado:', traceback.format_exc())
