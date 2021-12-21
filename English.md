# NMT-Keras: Machine Translation Lab Sessions
This is a custom version used in the lab sessions of the machine translation module of the [IARFID](http://www.upv.es/titulaciones/MUIARFID/) master. For more information about *NMT-Keras*, check the [oficial repository](https://github.com/lvapeab/nmt-keras).

## Table of Contents
* [Installation](#installation).
* [Variable definition](#variable-definition).
* [Network description](#network-description).
* [Dataset](#dataset).
* [Training](#training).
* [Translation](#translation).
* [Evaluation](#evaluation).
* [Tunning](#tunning).
* [Bibliography](#bibliography).


## Installation
In order to install *NMT-Keras*, it is assumed that you have created a directory "TA" in which to conduct the lab sessions. For simplicity's sake, throught this guide we shall assume that this directory is located at the home. Similarly, we shall asume that you wish to install *NMT-Keras* into the "TA" directory. Otherwise you will need to modify `$INSTALLATION_PATH`.

  ```console
~/TA$ wget https://raw.githubusercontent.com/PRHLT/nmt-keras_practicas-TA\
/master/full_installation.sh
~/TA$ chmod +x full_installation.sh
~/TA$ export INSTALLATION_PATH=~/TA
~/TA$ ./full_installation.sh ${INSTALLATION_PATH}
  ```

Once the installation has ended, you will need to ensure that *Tensorflow* is set up as *Keras* backend. To do so you will need to check if the following line is present in the file `~/.keras/keras.json`:

```json
"backend": "tensorflow"
```

### Docker
Alternatively, in the directory [docker](docker/English.md) you can find instrucction to run the toolkit through Docker.

## Variable definition
For the correct use of *NMT-Keras*, the following variables need to be set up:

```console
~/TA$ export TA=~/TA
~/TA$ export NMT=${INSTALLATION_PATH}/NMT_TA
~/TA$ export PATH=${NMT}/miniconda/bin/:${PATH}
~/TA$ export PYTHONPATH=${PYTHONPATH}:${NMT}/nmt-keras/keras:\
${NMT}/nmt-keras/coco-caption:${NMT}/nmt-keras/multimodal_keras_wrapper
```

## Network description
The file `${NMT}/nmt-keras/config.py` contains the network configuration:

* The encoder is a bidirectional LSTM with 64 neurons.
* A source word vector of size 64.
* The decoder is an LSTM with 64 neurons.
* A target word vector of size 64.
* The learning rate is set to 0.001.
* The number of epochs is set to 5.

## Dataset
The dataset is located at `dataset/EuTrans`. It is already set up a no further preprocesses are needed. However, for compatibility with the lab version, we will need to create a `data` folder in our working directory and copy the dataset there:

```
mkdir ~/TA/Practica2/data
cp -r dataset/EuTrans ~/TA/Practica2/data
```

Note: alternatively, the variable `DATA_ROOT_PATH` from `config.py` can be edited to indicate the path to the dataset.

## Training
After copying the dataset to the working directory, you can start the training by doing:

```console
~/TA/Practica2$ python ${NMT}/nmt-keras/main.py 2>traza &
```

This process will take some minutes. You can follow its evolution by doing:

```console
~/TA/Practica2$ tail -f traza | grep "\[*\]"
```

## Translation
Once the network has been trained, the translation can be performed by doing:

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

## Evaluation
The translation hypothesis can be evaluated by doing:

```console
~/TA/Practica2$ {$NMT}/nmt-keras/utils/multi-bleu.perl \
	-lc Data/EuTrans/test.en  < hyp.test.en
```

## Tunning
In order to tune the network parameters it is recommended to make a local copy of the `config.py` file:

```console
~/TA/Practica2$ cp ${NMT}/nmt-keras/config.py .
```

After that, you can edit the desired parameters in the copy we have just created. Then, we can train the network as follows:

```console
~/TA/Practica2$ python ${NMT}/nmt-keras/main.py -c config.py 2>traza &
```

Similarly, the translation will be conducted as follows:

```console
~/TA/Practica2$ python ${NMT}/nmt-keras/sample_ensemble.py \
--models trained_model/epoch_5 \
--dataset datasets/Dataset_EuTrans_esen.pkl \
--text Data/EuTrans/test.es \
--dest hyp.test.en \
--config config.py
```

## Bibliography
Álvaro Peris and Francisco Casacuberta. [NMT-Keras: a Very Flexible Toolkit with
a Focus on Interactive NMT and Online Learning](https://ufal.mff.cuni.cz/pbml/111/art-peris-casacuberta.pdf). The Prague Bulletin of Mathematical Linguistics. 2018.
