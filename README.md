# Laboratorio-3-Fiesta-de-Coctel

En este laboratorio se implementan dos metodos de procesamiento digital de señales, el Independent component analysis (ICA) y el beamforming para solucionar el problema conocido como Fiste de Cóctel, donde se busca separar la voz de una persona en específico de las demás que están hablando en una habitación.

1. Configuración del sistema:
Se distribuyeron 3 micrófonos en una sala insonorizada de forma tal que capturarón diferentes combinaciones de las señales de las 3 voces de las personas que estarán hablando a la vez, cada una de estas personas en posiciones fijas con diferentes distancias entre sí y los micrófonos para generar el ambiente de "fiesta de cóctel".

2. Captura de la señal:
Cada persona debe decir una frase diferente durante el tiempo de grabación de la señal, en este caso se utilizará la captura correspondiente a 17,5 segundos, se utilizarón para la captura 3 celulares de referencia Samsung A12, Samsung A34 Y un Samsung A55, con la aplicación predeterminada de grabación de audio. La frecuencia de muestreo fue de 44100 Hz

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

Posteriormente para realizar el análisis espectral se utilizo la transformada de Fourier discreta (DFT).
```
def plot_signal_and_spectrum(signal, sample_rate, title):
    
    DFT
    
    time = np.linspace(0, len(signal) / sample_rate, num=len(signal))

    # Aplicar FFT y calcular la magnitud del espectro
    freq = np.fft.fftfreq(len(signal), 1 / sample_rate)
    spectrum = np.abs(fft(signal))

    # Crear una nueva figura para cada señal
    plt.figure(figsize=(12, 6))


    plt.title(f"Espectro de Frecuencia: {title}")

    plt.tight_layout()
    plt.show()
```
## Procesamiento de las señales
La señal de cada micrófono se graficó tanto en el dominio del tiempo como en de la frecuencia.
Debido a que es una señal digital de audio, su amplitud representa un numero de bits, la intensidad sonora del audio al reproducirse depende del dispositivo que se use para tal fin
```
 # Gráfico de la señal en el tiempo
    plt.subplot(2, 1, 1)
    plt.plot(time, signal, color='b')
    plt.xlabel("Tiempo (s)")
    plt.ylabel("Amplitud")
    plt.title(f"Señal en el Tiempo: {title}")
```
Se usó la transformada rápida de fourier para pasarlo al dominio de la frecuencia, observando las gráficas obtenidas se confirma que el componente principal de la señal es de baja frecuencia, esto tiene sentido ya que la frecuencia de la voz humana varía de entre 80 Hz a 300 Hz
```
# Gráfico del espectro de frecuencia
    plt.subplot(2, 1, 2)
    plt.plot(freq[:len(freq) // 2], spectrum[:len(spectrum) // 2], color='r')  # Solo parte positiva
    plt.xlabel("Frecuencia (Hz)")
    plt.ylabel("Magnitud")
    plt.title(f"Espectro de Frecuencia: {title}")
    plt.tight_layout()
    plt.show()
```
![image](https://github.com/user-attachments/assets/f3b72719-ee15-42c2-b5be-2e24befc336d)
![image](https://github.com/user-attachments/assets/aec9c4b4-c963-44b1-bd48-9beb710ec5bd)
![image](https://github.com/user-attachments/assets/154ceb67-60b6-4ae3-8871-78c12c9e8f83)
Como se puede ver la escala de la amplitud al pasarlo al dominio de la frecuencia es muy grande por lo que es más conveniente usar una escala logaritmica en decibeles, tomamos como punto de referencia la media de la señal de ruido ambiente, y se obtuvieron las siguientes gráficas:
![image](https://github.com/user-attachments/assets/ee7b8550-ad9c-4f3f-8001-2d222f669ccc)
![image](https://github.com/user-attachments/assets/11db5039-abf0-4233-8ca3-6f1d2a3052f8)
![image](https://github.com/user-attachments/assets/e9030c4e-1fcc-43d9-9a11-af80db8a93cf)



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
>[!TIP]
>Para hallar el maximo delay se debe tener en cuenta tanto la disposición de los micrófonos como su frecuencia de muestreo, siguiendo la siguiente ecuación:
>
>![image](https://github.com/user-attachments/assets/e621bdd4-25d1-4619-bdfe-a15634868710)

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
## Calculo del snr y conclusiones.


