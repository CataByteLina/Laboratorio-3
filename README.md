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

## Uso del ICA
Para implementar este método el audio debe ser mono, por lo que se pasan todos loas audios a este formato, adicionalmente se recortan todos los audios para que queden con la misma longitud.
Después de esto, se guardan los 3 archivos de audio en una matriz de mezclas.
 ```
def to_mono(data):
    return data[:, 0] if len(data.shape) > 1 else data


data_L, data_P, data_C, data_amb = map(to_mono, [data_L, data_P, data_C, data_amb])

# Asegurar misma longitud de señales
min_length = min(len(data_L), len(data_P), len(data_C), len(data_amb))
data_L, data_P, data_C, data_amb = [d[:min_length] for d in [data_L, data_P, data_C, data_amb]]

# Crear matriz de mezclas
X = np.vstack([data_L, data_P, data_C]).T  # Transpuesta para formato correcto
```
El método de separacion ICA se basa en que las voces separadas tienen una distribución menos gaussiana que la mezcla, original, de esta manera el algoritmo separa cada señal, despues de esto se normaliza para que pueda ser guardada de maner adecuada en formato .wav

 ```
ica = FastICA(n_components=3, random_state=42)
S_ = ica.fit_transform(X)  # Separar fuentes

# Seleccionar una voz separada
voice_selected = S_[:, 0]  # Puedes cambiar entre 0, 1 o 2 para elegir la mejor

# Normalizar la señal
def normalize_signal(signal):
    max_val = np.max(np.abs(signal))
    return (signal / max_val * 32767).astype(np.int16) if max_val > 0 else np.zeros_like(signal, dtype=np.int16)
voice_selected = normalize_signal(voice_selected)
 ```
## Uso del beamforming
En el caso del beamforming se usa correlación cruzada para alinear todas las señales temporalmente (esto se hace ya que los microfonos no se encendieron exactamente al mismo tiempo por lo que puede haber un pequeño delay entre las señales), una vez se encuentra el desfase, se recorta la señal si está atrasada o se aplica un delay si está adelantada, luego se almacenan en una matriz y se asegura que tengan la misma longitud.

Para hallar el maximo delay se debe tener en cuenta tanto la disposición de los micrófonos como su frecuencia de muestreo, siguiendo la siguiente ecuación:

![image](https://github.com/user-attachments/assets/e621bdd4-25d1-4619-bdfe-a15634868710)

donde Md es el delay máximo, dmax la distancia entre los 2 micrófonos más alejados, f la frecuencia de muestreo de los microfonos y Vs la velocidad del sonido.
Tomando en cuenta la disposición de nuestros micrófonos, la distancia máxima es de 4,31 m y la frecuencia de muestreo usada fue de 44100 Hz.


![image](https://github.com/user-attachments/assets/a8a17081-13c5-4843-bbc1-aa7d0b4fe9ce)


reemplazando encontramos que el Md en nuestro caso es de 554.

Despues se promedian las señales y se normalizan, esto para evitar que se vaya a distorsionar el audio y se disminuya el ruido.
```
def beamforming(signals):
    """
    Aplica Delay-and-Sum Beamforming a una lista de señales.
    """
    max_lag = 554  # Máximo retardo en muestras para alineación
    reference_signal = signals[0]  # Usamos la primera señal como referencia

    aligned_signals = []
    for signal in signals:
        corr = correlate(reference_signal, signal, mode="full")  # Correlación cruzada
        lag = np.argmax(corr) - (len(signal) - 1)
        lag = np.clip(lag, -max_lag, max_lag)  # Limitar el retardo máximo

        # Desplazar la señal
        if lag > 0:
            aligned_signal = np.pad(signal, (lag, 0), mode="constant")[:len(signal)]
        else:
            aligned_signal = np.pad(signal, (0, -lag), mode="constant")[:len(signal)]

        aligned_signals.append(aligned_signal)

    # Convertir a array y asegurar misma longitud
    aligned_signals = np.array(aligned_signals)
    beamformed_signal = np.mean(aligned_signals, axis=0)

    return normalize_signal(beamformed_signal)


# Aplicar beamforming usando las señales separadas
beamformed_voice = beamforming(S_.T)

# Guardar la señal con beamforming
wav.write("voz_beamforming.wav", sample_rate_L, beamformed_voice)

print("Se han guardado los archivos 'voz_separada.wav' y 'voz_beamforming.wav'.")
```


