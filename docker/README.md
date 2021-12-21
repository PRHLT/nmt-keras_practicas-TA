# Ejecución mediante Docker

## Tabla de Contenidos
* [Construir la imagen](#construir-la-imagen).
* [Datos](#datos).
* [Entrenamiento](#entrenamiento).
* [Traducción](#traducción).
* [Evaluación](#evaluación).
* [Ajuste de parámetros](#ajuste-de-parámetros).

## Construir la imagen
El primer paso a realizar es construir la imagen. Para ello, es necesario ejecutar la siguiente orden (bajo el supuesto de que se está en el directorio donde se encuentra este repo. Si no, habrá que reemplazar `.` con la ruta correcta):

```
docker build -t nmt-keras-ta .
```

Nota: esta imagen utiliza Cuda 10. En caso de necesitar usar Cuda 11, construir la imagen a partir del Dockerfile contenido en [este directorio](cuda11).


## Datos
Los datos a usar se encuentran ubicados en el directorio `dataset/EuTrans`. Vienen ya preparados y no requiren de ningún preproceso. Sin embargo, por compatibilidad con la versión del laboratorio, deberemos crear un directorio `data` en el directorio donde estamos trabajando y copiar los datos ahí:

```
mkdir ~/TA/Practica2/data
cp -r dataset/EuTrans ~/TA/Practica2/data
```

Nota: alternativamente, se puede editar la variable `DATA_ROOT_PATH` del fichero `config.py` para indicar la ubicación de los datos.

## Entrenamiento
Una vez copiado los datos al directorio de trabajo, el entrenamiento se inicia mediante:

```
docker container run -it --rm -v /home/$user/TA/Practica2:/data nmt-keras-ta
```
siendo `$user` tu nombre de usuario.

Este proceso dura unos minutos.

## Traducción
Una vez entrenada la red, será necesario ejecutar bash dentro del contenedor de docker como paso previo a realizar la traducción:

```
docker container run -it --rm -v /home/$user/TA/Practica2:/data nmt-keras-ta /bin/bash
```

Tras esto, la traducción se realiza de la siguiente forma:

```
ln -s /data/trained_models/EuTrans_esen_AttentionRNNEncoderDecoder_\
src_emb_64_bidir_True_enc_LSTM_64_dec_ConditionalLSTM_64_deepout_\
linear_trg_emb_64_Adam_0.001 trained_model
python sample_ensemble.py \
--models trained_model/epoch_5 \
--dataset /data/datasets/Dataset_EuTrans_esen.pkl \
--text /data/Data/EuTrans/test.es \
--dest /data/hyp.test.en
```

## Evaluación
Para evaluar la traducción, habrá que volver a ejecutar bash dentro del contenedor de docker:

```
docker container run -it --rm -v /home/$user/TA/Practica2:/data nmt-keras-ta /bin/bash
```

A continuación, la traducción se puede evaluar mediante:

```
utils/multi-bleu.perl -lc /data/Data/EuTrans/test.en  < /data/hyp.test.en
```

## Ajuste de parámetros
Para poder modificar los parámetros de la red, habrá que obtener una copia del fichero ```config.py```:

```
wget https://raw.githubusercontent.com/PRHLT/nmt-keras_practicas-TA\
/master/docker/config.py
```

Tras esto, se procederá a modificar los parametros deseados que están definidos en la
copia local que acabamos de crear. Una vez definidos los parámetros deseados, se procederá a realizar el entrenamiento de la red de la siguiente manera:

```
docker container run -it --rm \
-v /home/$user/TA/Practica2:/data \
-v $ruta/config.py:/opt/nmt-keras/config.py \
nmt-keras-ta
```
donde `$ruta` es la ruta en la que se encuentra la copia local de `config.py`.

De manera similar, la ejecución de bash (para los procesos de traducción y evaluación) pasarán a:

```
docker container run -it --rm \
-v /home/$user/TA/Practica2:/data \
-v $ruta/config.py:/opt/nmt-keras/config.py \
nmt-keras-ta /bin/bash
```
