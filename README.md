# NMT-Keras: Prácticas Traducción Automática
Esta es una versión modificada para la realización de las prácticas de la asignatura de traducción automática del máster [IARFID](http://www.upv.es/titulaciones/MUIARFID/). Para más información acerca de *NMT-Keras*, consultar el [repositorio oficial](https://github.com/lvapeab/nmt-keras). For an English version click [here](English.md).

## Tabla de Contenidos
* [Instalación](#instalación).
* [Definición de variables](#definición-de-variables).
* [Descripción de la red](#descripción-de-la-red).
* [Datos](#datos).
* [Entrenamiento](#entrenamiento).
* [Traducción](#traducción).
* [Evaluación](#evaluación).
* [Ajuste de parámetros](#ajuste-de-parámetros).
* [Bibliografía](#bibliografía).


## Instalación
Para instalar *NMT-Keras*, se asume que se tiene creado un directorio “TA” donde se realizarán las prácticas. Por simplicidad, a lo largo de esta guía asumiremos que
este directorio se encuentra ubicado en el home. De forma similar, asumimos que
se desea instalar *NMT-Keras* en el directorio “TA”. De no ser ası́, bastará con modificar ```$INSTALLATION PATH```.

  ```console
~/TA$ wget https://raw.githubusercontent.com/PRHLT/nmt-keras_practicas-TA\
/master/full_installation.sh
~/TA$ chmod +x full_installation.sh
~/TA$ export INSTALLATION_PATH=~/TA
~/TA$ ./full_installation.sh ${INSTALLATION_PATH}
  ```

Una vez finalizada la instalación, se deberá asegurar que *Tensorflow* está configurado como backend de *Keras*. Para ello, se deberá comprobar que en el fichero ```~/.keras/keras.json``` aparece la siguiente lı́nea:

```json
"backend": "tensorflow"
```

### Docker
Alternativamente, en el directorio [docker](docker) hay instrucciones para usar el toolkit mediante Docker.

## Definición de variables
Para el correcto uso de *NMT-Keras*, es necesario configurar las siguientes variables:

```console
~/TA$ export TA=~/TA
~/TA$ export NMT=${INSTALLATION_PATH}/NMT_TA
~/TA$ export PATH=${NMT}/miniconda/bin/:${PATH}
~/TA$ export PYTHONPATH=${PYTHONPATH}:${NMT}/nmt-keras/keras:\
${NMT}/nmt-keras/coco-caption:${NMT}/nmt-keras/multimodal_keras_wrapper
```

## Descripción de la red
En el fichero ```${NMT}/nmt-keras/config.py``` está detallada la red que se va a utilizar. Por defecto, ésta se compone de:

* El codificador es un LSTM bidireccional de 64 neuronas.
* El tamaño del vector para codificar las palabras fuente es de 64.
* El decodificador es un LSTM de 64 neuronas.
* El tamaño del vector para codificar las palabras destino es de 64.
* El factor de aprendizaje inicial es de 0.001.
* El número de epochs es 5.
* Otros parámetros de la red se encuentran en ```config.py```.

## Datos
Los datos a usar se encuentran ubicados en el directorio `dataset/EuTrans`. Vienen ya preparados y no requiren de ningún preproceso. Sin embargo, por compatibilidad con la versión del laboratorio, deberemos crear un directorio `data` en el directorio donde estamos trabajando y copiar los datos ahí:

```
mkdir ~/TA/Practica2/data
cp -r dataset/EuTrans ~/TA/Practica2/data
```

Nota: alternativamente, se puede editar la variable `DATA_ROOT_PATH` del fichero `config.py` para indicar la ubicación de los datos.

## Entrenamiento
Una vez copiado los datos al directorio de trabajo, el entrenamiento se inicia mediante:

```console
~/TA/Practica2$ python ${NMT}/nmt-keras/main.py 2>traza &
```

Este proceso dura unos minutos. La evolución del mismo se puede seguir haciendo:

```console
~/TA/Practica2$ tail -f traza | grep "\[*\]"
```

## Traducción
Una vez entrenada la red, la traducción se realiza de la siguiente forma:

```console
~/TA/Practica2$ ln -s trained_models/EuTrans_esen_AttentionRNNEncoderDecoder_\
src_emb_64_bidir_True_enc_LSTM_64_dec_ConditionalLSTM_64_deepout_\
linear_trg_emb_64_Adam_0.001 trained_model
~/TA/Practica2$ python ${NMT}/nmt-keras/sample_ensemble.py \
--models trained_model/epoch_5 \
--dataset datasets/Dataset_EuTrans_esen.pkl \
--text Data/EuTrans/test.es \
--dest hyp.test.en
```

## Evaluación
La traducción se puede evaluar mediante:

```console
~/TA/Practica2$ {$NMT}/nmt-keras/utils/multi-bleu.perl \
	-lc Data/EuTrans/test.en  < hyp.test.en
```

## Ajuste de parámetros
Para poder modificar los parámetros de la red, es aconsejable realizar una copia local del fichero ```config.py```:

```console
~/TA/Practica2$ cp ${NMT}/nmt-keras/config.py .
```

Tras esto, se procederá a modificar los parametros deseados que están definidos en la
copia local que acabamos de crear. Una vez definidos los parámetros deseados, se procederá a realizar el entrenamiento de la red de la siguiente manera:

```console
~/TA/Practica2$ python ${NMT}/nmt-keras/main.py -c config.py 2>traza &
```

De manera similar, la traducción pasará a realizarse del siguiente modo:

```console
~/TA/Practica2$ python ${NMT}/nmt-keras/sample_ensemble.py \
--models trained_model/epoch_5 \
--dataset datasets/Dataset_EuTrans_esen.pkl \
--text Data/EuTrans/test.es \
--dest hyp.test.en \
--config config.py
```

## Bibliografía
Álvaro Peris and Francisco Casacuberta. [NMT-Keras: a Very Flexible Toolkit with
a Focus on Interactive NMT and Online Learning](https://ufal.mff.cuni.cz/pbml/111/art-peris-casacuberta.pdf). The Prague Bulletin of Mathematical Linguistics. 2018.
