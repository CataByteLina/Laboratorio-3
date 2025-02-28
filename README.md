# Laboratorio-3-Fiesta-de-Coctel

En este laboratorio se implementan dos metodos de procesamiento digital de señales, el Independent component analysis (ICA) y el beamforming para solucionar el problema conocido como Fiste de Cóctel, donde se busca separar la voz de una persona en específico de las demás que están hablando en una habitación.

1. Configuración del sistema:
Se distribuyeron 3 micrófonos en una sala insonorizada de forma tal que capturarón diferentes combinaciones de las señales de las 3 voces de las personas que estarán hablando a la vez, cada una de estas personas en posiciones fijas con diferentes distancias entre sí y los micrófonos para generar el ambiente de "fiesta de cóctel".

2. Captura de la señal:
Cada persona debe decir una frase diferente durante el tiempo de grabación de la señal, en este caso se utilizará la captura correspondiente a 17,5 segundos, se utilizarón para la captura 3 celulares de referencia Samsung A12, Samsung A34 Y un Samsung A55, con la aplicación predeterminada de grabación de audio.

Se calculó el SNR de cada señal
```
sample_rate_L, data_L = wav.read(audioL)
sample_rate_P, data_P = wav.read(audioP)
sample_rate_C, data_C = wav.read(audioC)
sample_rate_amb, data_amb = wav.read(audio_amb)

def calcular_snr(signal, noise):
    pseñal = np.mean(signal ** 2) #potencia de la señal
    # Calcular la potencia del ruido
    pruido = np.mean(noise ** 2) #potencia del ruido
    # Calcular la SNR
    snr = 10 * np.log10(pseñal / pruido)
    return snr

# Calcular la SNR para cada archivo
snr_L = calcular_snr(data_L, data_amb)
snr_P = calcular_snr(data_P, data_amb)
snr_C = calcular_snr(data_C, data_amb)
```

![image](https://github.com/user-attachments/assets/830c4f95-ee1e-4b68-bbcc-2aea0eb86b8c)

Los SNR salieron bajos (menores a 10dB) ya que, aunque era una habitación insonorizada, las voces de las otras dos personas pueden actuar como ruido no deseado desde la perspectiva de cada grabación individual teniendo en cuenta su distancia a las fuentes de sonido, siendo así que las otras voces se interpretan como ruido. Además, los micrófonos de los celulares no son direccionales, por lo que captan todas las voces en el ambiente.





