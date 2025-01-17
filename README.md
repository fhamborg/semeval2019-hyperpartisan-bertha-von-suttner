# SemEval2019/Task4 Team Bertha-von-Suttner submission
You can skip the training by using the pretrained model ensemble that is already contained in `saved_models`.

## Setup
```
conda create --yes -n semevalhyperpart python=3.6
conda activate semevalhyperpart
pip install gensim==3.4.0
pip install tensorflow==1.12.0
pip install Keras==2.2.4
pip install nltk==3.4.1
conda install pytorch=0.3.1 -c pytorch # this is what i had in the documentation, but it doesnt match with allennlp's reqs: conda install pytorch=0.4.1 -c pytorch
pip install spacy==2.0.16
pip install scikit-learn==0.20.0
pip install allennlp==0.4.3 # added myself, is also from roughly March 2018 from which the other PyPI package versions are, too
python -m spacy download en_core_web_sm
python -m nltk.downloader stopwords
mkdir elmo
cd elmo
wget https://s3-us-west-2.amazonaws.com/allennlp/models/elmo/2x4096_512_2048cnn_2xhighway/elmo_2x4096_512_2048cnn_2xhighway_weights.hdf5
wget https://s3-us-west-2.amazonaws.com/allennlp/models/elmo/2x4096_512_2048cnn_2xhighway/elmo_2x4096_512_2048cnn_2xhighway_options.json
```

Optional, because this repo contains a model ensemble in `saved_models` trained on the following two files. If you
want to train yourself, create a directory `data` and save the by-article training files into it:
  * articles-training-byarticle-20181122.xml
  * ground-truth-training-byarticle-20181122.xml 


## Training
* Convert the XML file into a tsv file with one article per line:
  * `python Preprocessing/xml2line.py -A data/articles-training-byarticle-20181122.xml -T data/ground-truth-training-byarticle-20181122.xml -F article_sent,title_sent work/train.text.tsv`
* Convert the tsv file containing text into a tsv file containing elmo embeddings:
  * If you have a GPU: `python Preprocessing/line2elmo2.py -g -l 100  work/train.text.tsv work/train.elmo.tsv`
  * Otherwise: `python Preprocessing/line2elmo2.py -l 100 work/train.text.tsv work/train.elmo.tsv`
  If you get problems with the GPU memory or RAM, use the `-b option to reduce the batch size
* Make sure the directory `saved_models` does not contain any model files from previous runs:
  * `rm saved_models/*.hdf5`
* Train the actual model: 
  `KERAS_BACKEND=tensorflow python CNN_elmo.py work/train.elmo.tsv`
  This will create a number of model files in the `saved_models` directory. The file names contain the validation accuracy.

## Infer
To infer on our own files, follow the following steps (in this example using a `mytest.jsonl). See bottom of readme.md for inferring on original semeval files. 

* Convert the JSONL to tsv:
  `python Preprocessing/jsonl2line.py -A data/mytest.jsonl -F article_sent,title_sent work/mytest.text.tsv`
* Convert the text to elmo embeddings:
  * If you have a GPU: `python Preprocessing/line2elmo2.py -g -l 100  work/mytest.text.tsv work/mytest.elmo.tsv`
  * Otherwise: `python Preprocessing/line2elmo2.py -l 100 work/mytest.text.tsv work/mytest.elmo.tsv`
* Run the actual application of the model ensemble
  * `./ensemble_pred.sh work/mytest.elmo.tsv work/mytest.preds.txt` 


## Original readme.md
This is the code for the [SemEval 2019 Task 4, Hyperpartisan News Detection](https://pan.webis.de/semeval19/semeval19-web/)
submitted by team `Bertha von Suttner`:
* [Ye Jiang](https://ye-jiang.github.io/)
* [Johann Petrak](http://johann-petrak.github.io) 
* [Xingyi Song](http://staffwww.dcs.shef.ac.uk/people/X.Song/)

All are members of the [GATE](https://gate.ac.uk) team of the [University of Sheffield Natural Language Processing group](https://www.sheffield.ac.uk/dcs/research/groups/nlp)

The model created with this was the winning entry, see the public leaderboard (sort by accuracy column, descending):
https://www.tira.io/task/hyperpartisan-news-detection/dataset/pan19-hyperpartisan-news-detection-by-article-test-dataset-2018-12-07/

[A blog article on the GATE blog](https://gate4ugc.blogspot.com/2019/02/gate-team-wins-first-prize-in.html) briefly describes the approach taken.

If you wish to see the code as it was prepared for the SemEval 2019 task, then refer to the `semval-2019` tag in the git repo.

## Preparation / Requirements
* Python 3.6 (Anaconda will work best)
* Gensim version 3.4.0
* Tensorflow version 1.12.0
* Keras version 2.2.4
* NLTK version 3.4.1
* PyTorch version 0.4.1
* Spacy version 2.0.16
* Sklearn version 0.20.0

Once spaCy is installed, you also need to install its
[`en_core_web_sm`](https://spacy.io/usage/models) model.
Like this:

```
python -m spacy download en_core_web_sm
```

Once NLTK is installed, you also need to install its
`stopwords` data:

```
python -m nltk.downloader stopwords
```

Preparation steps:
* create a directory `elmo` and store the ELMo model files in that directory:
  * https://s3-us-west-2.amazonaws.com/allennlp/models/elmo/2x4096_512_2048cnn_2xhighway/elmo_2x4096_512_2048cnn_2xhighway_weights.hdf5
  * https://s3-us-west-2.amazonaws.com/allennlp/models/elmo/2x4096_512_2048cnn_2xhighway/elmo_2x4096_512_2048cnn_2xhighway_options.json
* create a directory `data` and save the by-article training files into it:
  * articles-training-byarticle-20181122.xml
  * ground-truth-training-byarticle-20181122.xml 


## Training

Run the following steps

* Convert the XML file into a tsv file with one article per line:
  * `python Preprocessing/xml2line.py -A data/articles-training-byarticle-20181122.xml -T data/ground-truth-training-byarticle-20181122.xml -F article_sent,title_sent work/train.text.tsv`
* Convert the tsv file containing text into a tsv file containing elmo embeddings:
  * If you have a GPU: `python Preprocessing/line2elmo2.py -g -l 100  work/train.text.tsv work/train.elmo.tsv`
  * Otherwise: `python Preprocessing/line2elmo2.py -l 100 work/train.text.tsv work/train.elmo.tsv`
  If you get problems with the GPU memory or RAM, use the -b option to reduce the batch size
* Make sure the directory `saved_models` does not contain any model files from previous runs:
  * `rm saved_models/*.hdf5`
* Train the actual model: 
  `KERAS_BACKEND=tensorflow python CNN_elmo.py work/train.elmo.tsv`
  This will create a number of model files in the `saved_models` directory. The file names contain the validation accuracy.


## Application

* Convert the XML to tsv:
  `python Preprocessing/xml2line.py -A $TESTXMLFILE -F article_sent,title_sent work/test.text.tsv`
* Convert the text to elmo embeddings:
  * If you have a GPU: `python Preprocessing/line2elmo2.py -g -l 100  work/test.text.tsv work/test.elmo.tsv`
  * Otherwise: `python Preprocessing/line2elmo2.py -l 100 work/test.text.tsv work/test.elmo.tsv`
* Run the actual application of the model ensemble
  * `./ensemble_pred.sh work/test.elmo.tsv work/test.preds.txt` 
