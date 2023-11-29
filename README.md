PAV - P3: estimación de pitch
=============================
Raquel Galisteo y Xavier Moreno
-------------------------------

Esta práctica se distribuye a través del repositorio GitHub [Práctica 3](https://github.com/albino-pav/P3).
Siga las instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para realizar un `fork` de la
misma y distribuir copias locales (*clones*) del mismo a los distintos integrantes del grupo de prácticas.

Recuerde realizar el *pull request* al repositorio original una vez completada la práctica.

Ejercicios básicos
------------------

- Complete el código de los ficheros necesarios para realizar la estimación de pitch usando el programa
  `get_pitch`.

   * Complete el cálculo de la autocorrelación e inserte a continuación el código correspondiente.

   ```bash
   for (unsigned int l = 0; l < r.size(); ++l) {
      r[l] = 0;
      for (unsigned int n = 0; n < x.size()-l; n++){
        r[l] += x[n]*x[n+l];
      }
      r[l] /= x.size();
    }

    if (r[0] == 0.0F) //to avoid log() and divide zero 
      r[0] = 1e-10; 
    ```

   * Inserte una gŕafica donde, en un *subplot*, se vea con claridad la señal temporal de un segmento de
     unos 30 ms de un fonema sonoro y su periodo de pitch; y, en otro *subplot*, se vea con claridad la
	 autocorrelación de la señal y la posición del primer máximo secundario.

	 NOTA: es más que probable que tenga que usar Python, Octave/MATLAB u otro programa semejante para
	 hacerlo. Se valorará la utilización de la biblioteca matplotlib de Python.

     <img src="img/grafica_fonema_autocorr.png" align="center">

   * Determine el mejor candidato para el periodo de pitch localizando el primer máximo secundario de la
     autocorrelación. Inserte a continuación el código correspondiente.

     __Como se puede observar en la imagen anterior, el mejor candidato para el periodo de pitch es de 6.875ms. Este valor puede localizarse perfectamente en ambos dominios, ya que, en la gráfica temporal, se aprecia claramente que cada periodo ocupa aproximadamente 7ms, y en el dominio de la autocorrelación se distingue claramente el primer máximo secundario, obteniendo así el valor del mejor candidato para el periodo de pitch. El código necessario para obtener estos resultados gráficos lo hemos realizado con Python y es el mostrado a continuación (también está disponible en el archivo _gráfica_autocorrelación.py_):__

     ```bash
     import matplotlib.pyplot as plt
     import numpy as np
     import soundfile as sf

     # Leemos el fichero del fonema con la libreria "soundfile"
     data, samplerate = sf.read("./fonema_E.wav") 
     t = np.arange(0, len(data))/samplerate

     # Calculamos la autocorrelación de los datos del fichero de audio
     corr = np.correlate(data, data, 'full') / len(data)
     corr = corr[int(corr.size/2):] # Consideramos solo la mitad positiva de la autocorrelación

     min_index = np.argmin(corr)
     max_index = np.argmax(corr[min_index:])
     max_value = np.max(corr[min_index:])
     fig, axs = plt.subplots(2)
            
     # Hacemos la representación gráfica
     axs[0].plot(t, data)
     axs[1].plot(t, corr)
     # Mostramos el valor del primer máximo secundario de la autoccorrelación para determinar el mejor candidato para el periodo de pitch
     axs[1].plot((min_index+max_index)/samplerate, max_value, 'ro', label='Pitch Estimation = {}ms'.format((min_index+max_index)*1000/samplerate))

     axs[0].set_title('Fonema Sonoro')
     axs[1].set_title('Autocorrelación de la señal')
     axs[1].set_xlabel('Tiempo (s)')
     axs[1].legend()

     fig.tight_layout()
     plt.show()
     ```

     __Las librerias que hemos utilizado para poder hacer la representación gráfica anterior nos permiten ver los gráficos, con matplotlib, realizar las operaciones matematicas correspondientes, con numpy, y leer archivos de audio, con soundfile.__
  
   * Implemente la regla de decisión sonoro o sordo e inserte el código correspondiente.

     __La regla de decisión sonoro o sordo se basa en 3 parámetros: la autocorrelación, la relación R(1)/R(0) y el valor de la potencia. El código correspondiente se muestsra a continuación:__

     ```bash
     if(rmaxnorm>umaxnorm && r1norm > r1thr && pot > powthr) return false;
     return true;
     ```

   * Puede serle útil seguir las instrucciones contenidas en el documento adjunto `código.pdf`.

- Una vez completados los puntos anteriores, dispondrá de una primera versión del estimador de pitch. El 
  resto del trabajo consiste, básicamente, en obtener las mejores prestaciones posibles con él.

  * Utilice el programa `wavesurfer` para analizar las condiciones apropiadas para determinar si un
    segmento es sonoro o sordo. 
	
	  - Inserte una gráfica con la estimación de pitch incorporada a `wavesurfer` y, junto a ella, los 
	    principales candidatos para determinar la sonoridad de la voz: el nivel de potencia de la señal
		(r[0]), la autocorrelación normalizada de uno (r1norm = r[1] / r[0]) y el valor de la
		autocorrelación en su máximo secundario (rmaxnorm = r[lag] / r[0]).

		Puede considerar, también, la conveniencia de usar la tasa de cruces por cero.

	    Recuerde configurar los paneles de datos para que el desplazamiento de ventana sea el adecuado, que
		en esta práctica es de 15 ms.

     

      - Use el estimador de pitch implementado en el programa `wavesurfer` en una señal de prueba y compare
	    su resultado con el obtenido por la mejor versión de su propio sistema.  Inserte una gráfica
		ilustrativa del resultado de ambos estimadores.
     
		Aunque puede usar el propio Wavesurfer para obtener la representación, se valorará
	 	el uso de alternativas de mayor calidad (particularmente Python).

     <img src="img/grafica_comparacion_pitch.png" align="center">

     __Como podemos observar, el estimador de pitch que hemos obtenido con nuestro sistema es bastante preciso, ya que, al comparar el contorno de nuestro pitch con el que estima wavesurfer, vemos que son muy similares.__
  
  * Optimice los parámetros de su sistema de estimación de pitch e inserte una tabla con las tasas de error
    y el *score* TOTAL proporcionados por `pitch_evaluate` en la evaluación de la base de datos 
	`pitch_db/train`..

   __El resultado final de nuestro sistema es el mostrado a continuación:__

   <img src="img/resultado_final.png" align="center">

Ejercicios de ampliación
------------------------

- Usando la librería `docopt_cpp`, modifique el fichero `get_pitch.cpp` para incorporar los parámetros del
  estimador a los argumentos de la línea de comandos.
  
  Esta técnica le resultará especialmente útil para optimizar los parámetros del estimador. Recuerde que
  una parte importante de la evaluación recaerá en el resultado obtenido en la estimación de pitch en la
  base de datos.

  * Inserte un *pantallazo* en el que se vea el mensaje de ayuda del programa y un ejemplo de utilización
    con los argumentos añadidos.

    <img src="img/get_pitch_ayuda.png" align="center">

- Implemente las técnicas que considere oportunas para optimizar las prestaciones del sistema de estimación
  de pitch.

  Entre las posibles mejoras, puede escoger una o más de las siguientes:

  * Técnicas de preprocesado: filtrado paso bajo, diezmado, *center clipping*, etc.
  * Técnicas de postprocesado: filtro de mediana, *dynamic time warping*, etc.
  * Métodos alternativos a la autocorrelación: procesado cepstral, *average magnitude difference function*
    (AMDF), etc.
  * Optimización **demostrable** de los parámetros que gobiernan el estimador, en concreto, de los que
    gobiernan la decisión sonoro/sordo.
  * Cualquier otra técnica que se le pueda ocurrir o encuentre en la literatura.

  Encontrará más información acerca de estas técnicas en las [Transparencias del Curso](https://atenea.upc.edu/pluginfile.php/2908770/mod_resource/content/3/2b_PS%20Techniques.pdf)
  y en [Spoken Language Processing](https://discovery.upc.edu/iii/encore/record/C__Rb1233593?lang=cat).
  También encontrará más información en los anexos del enunciado de esta práctica.

  Incluya, a continuación, una explicación de las técnicas incorporadas al estimador. Se valorará la
  inclusión de gráficas, tablas, código o cualquier otra cosa que ayude a comprender el trabajo realizado.

  También se valorará la realización de un estudio de los parámetros involucrados. Por ejemplo, si se opta
  por implementar el filtro de mediana, se valorará el análisis de los resultados obtenidos en función de
  la longitud del filtro.

   __En nuestro caso, hemos implementado dos mejoras, las cuales se tratan de añadir un filtro _center-clipping_ y un filtro de mediana.__

   __Para el filtro _center-clipping_ hemos realizado el _TODO_ del fichero _get_pitch.cpp_ que pedia que preprocesaramos la señal de entrada para facilitar la estimación de pitch. El código implementado es el siguiente:__

   ```bash
   float max = *std::max_element(x.begin(), x.end());
   for(int i = 0; i < (int)x.size(); i++) {
     if(abs(x[i]) < cclip1*max) {
       x[i] = 0.0F;
    } 
   }
   ```

   __La implementación de este filtro se ha realizado sin un valor fijo, es decir, que el filtro se adapta según la potencia máxima de cada señal.__

   __Para el filtro de mediana hemos realizado el _TODO_ del fichero _get_pitch.cpp_ que pedia que postprocesáramos la estimación para suprimir errores. El código implementado es el siguiente:__

   ```bash
   vector<float> f0_final(f0.size());
   vector<float> temp(3);
   int i;
   for(i = 1; i < (int)(f0.size() - 1); i++) {
     temp = {f0[i-1], f0[i], f0[i+1]};
     auto m = temp.begin() + temp.size()/2;
     std::nth_element(temp.begin(), m, temp.end());
     f0_final[i] = temp[temp.size()/2];
   }
   f0_final[i] = f0_final[i-1];
   f0_final[0] = f0_final[1];
   ```

   __La implementación de este filtro de mediana se ha realizado mediante la comparación de muestras contiguas, es por eso que trabajamos con tres valores, el de la muestra actual, la anterior y la posterior. Lo hemos hecho así debido a que hay fragmentos sonoros del audio que son muy cortos, por lo que, para que el filtro tenga efecto, lo mejor es hacerlo de esta manera.__

Evaluación *ciega* del estimador
-------------------------------

Antes de realizar el *pull request* debe asegurarse de que su repositorio contiene los ficheros necesarios
para compilar los programas correctamente ejecutando `make release`.

Con los ejecutables construidos de esta manera, los profesores de la asignatura procederán a evaluar el
estimador con la parte de test de la base de datos (desconocida para los alumnos). Una parte importante de
la nota de la práctica recaerá en el resultado de esta evaluación.
