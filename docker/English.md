# Running through Docker

## Table of Contents
* [Image building](#image-building).
* [Dataset](#dataset).
* [Training](#trainig).
* [Translation](#translation).
* [Evaluation](#evaluation).
* [Tunning](#Tuning).

## Image building
The first step is to build the image. You can do so by running the following command (asumming that the current directory is the one in which this repo has been cloned. Otherwise `.` should be replaced with the correct path):

```
docker build -t nmt-keras-ta .
```

Note: this image uses Cuda 10. If you need Cuda 11, you can build the image using the Dockerfile from [here](cuda11).


## Dataset
The dataset is located at `dataset/EuTrans`. It is already set up a no further preprocesses are needed. However, for compatibility with the lab version, we will need to create a `data` folder in our working directory and copy the dataset there:

```
mkdir ~/TA/Practica2/data
cp -r dataset/EuTrans ~/TA/Practica2/data
```

Note: alternatively, the variable `DATA_ROOT_PATH` from `config.py` can be edited to indicate the path to the dataset.

## Training
After copying the dataset to the working directory, you can start the training by doing:

```
docker container run -it --rm -v /home/$user/TA/Practica2:/data nmt-keras-ta
```
where `$user` is your username.

This process takes some minutes.

## Translation
After the network has been trained, you need to run bash inside the docker container as a previous step to performing the translation:

```
docker container run -it --rm -v /home/$user/TA/Practica2:/data nmt-keras-ta /bin/bash
```

After this, the translation can be performed by doing:

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

## Evaluation
To asses the translation, you will need to run bash again inside the docker container:

```
docker container run -it --rm -v /home/$user/TA/Practica2:/data nmt-keras-ta /bin/bash
```

The, the translation can be assessed by doing:

```
utils/multi-bleu.perl -lc /data/Data/EuTrans/test.en  < /data/hyp.test.en
```

## Tunning
To tune the network parameter you will need to obtain a local copy of the `config.py` file:

```
wget https://raw.githubusercontent.com/PRHLT/nmt-keras_practicas-TA\
/master/docker/config.py
```

After that, you can edit the desired parameters in the copy we have just created. Then, we can train the network as follows:

```
docker container run -it --rm \
-v /home/$user/TA/Practica2:/data \
-v $ruta/config.py:/opt/nmt-keras/config.py \
nmt-keras-ta
```
where `$ruta` is the path in which the `config.py` file has been downloaded.

Similarly, the bash execution (for conducting the translation and the evaluation) will become:

```
docker container run -it --rm \
-v /home/$user/TA/Practica2:/data \
-v $ruta/config.py:/opt/nmt-keras/config.py \
nmt-keras-ta /bin/bash
```
