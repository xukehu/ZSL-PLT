# GazPNE
## 1.Basic description
We propose a hybrid method, named GazPNE, which fuses rules, gazetteers, and deep learning methods to achieve state-of-the-art-performance without requiring any manually annotated data. Specifically,  we utilize C-LSTM, a fusion of Convolutional and Long Short-Term Memory Neural Networks, to decide if an n-gram in a microblog text is a place name or not. The C-LSTM is trained on 6.8 million positive examples extracted from OpenStreetMap covering the US and India and 388.1 million negative examples synthesized by rules and  evaluated  on  4,500  disaster-related  tweets,  including  9,026 place names from three floods: 2016 in Louisiana (US), 2016 in Houston (US), and 2015 in Chennai (India). Our method achieve an F1 score of 0.84.

The Workflow of GazPNE is shown in the figure below.
![Screenshot](figure/workflow.jpg)

Note that if you just want to use our trained model, you can skip the second step. Instead, you can download the [model data](https://drive.google.com/file/d/10TokPTKJLwpjQR2oN-X03MO1GCEpeDyx/view?usp=sharing) and unzip the data into the ![model](model) folder. The Golve word embedding ([glove.6B.50d.txt](https://www.kaggle.com/watts2/glove6b50dtxt)) is also needed. Our model was trained based on the OSM data in the US and India. Thus, it can only reconginze the place in the US and India.

## 2.Model training
The first step of GazPNE is to train a classification model based on positive examples from gazetteers and negative examples sythesized by rules.
### Training data perparation
Several important data need to be prepared before generating the positive and negative examples. All data should be put in the ![data](data) folder.

**OpenStreetMap data**: We extract place names in the entire US and India from OSMNames. The extracted data are saved in us.tsv and india.tsv, respectively.


**Two word embeddings**: Google word embedding ([GoogleNews-vectors-negative300.bin](https://code.google.com/archive/p/word2vec/)) and Golve word embedding ([glove.6B.50d.txt](https://www.kaggle.com/watts2/glove6b50dtxt)).

After preparing the above data, [rawTextProcessing.py](rawTextProcessing.py) is used to extract the positive examples and negative examples. 

 > python rawTextProcessing.py --file 146 --ht 500 --lt 500 --ft 500 --unseen 1
 
Parameter <*file*> denotes the ID of the generated files, saving positive and negative examples, named positive**ID**.txt and negative**ID**.txt

We provide our extracted [positive](https://drive.google.com/file/d/1ewQH4__dpWV0sMumhf7VLVKCh3fAGSIN/view?usp=sharing) and [negative](https://drive.google.com/file/d/1KMGy2W82S5GtuJ9ghoT51MP48Gu-sqCk/view?usp=sharing) examples, named positive146.txt and negative146.txt, respectively.

Next, the file for the negative examples is split into multiple smaller files with each containing at most 10 million lines (examples) to improve the efficiency of reading the negative examples.

 > split -l 2000000 data/negative146.txt data/negative146
 
### Train a C-LSTM model

We apply the C-LSTM  model to classify the place entities, which combines the CNN and LSTM to achieve the best of both. The topology of the network is depicted as follows:
![Screenshot](figure/architecture.jpg)
[Gazetteer_weight.py](Gazetteer_weight.py) is used to train a model based on the positive and negative examples.

python -u Gazetteer_weight.py --epoch 7 --train-batch-size 1000 --test-batch-size 1000 --split_l 10000000 --lstm_dim 120 --cnn_hid 120 --filter_l 1 --max_cache 10 --hc 1 --osm_word_emb 1 --positive 146 --negative 146 --osmembed 2 --preloadsize 3000000 --emb 1 --osm_word_emb 0 --max_len=20

Two parameters <*positive*> and <*negative*> denote the ID of the file saving the positive examples and negative examples, respectively.

Then, we can get a model named like 'clstm_model_0708233420epoch0.pkl'. 0708233420 is the time to create the model, and also used as the ID of the model. We keep the trained model in each epoch. 



## 3.Place name tagging from tweet texts

The test dataset contains 4500 annotated tweets, corresponding to Chennai 2015, Louisiana 2016, and Houston 2016 floods. The three annotated [tweet data sets](https://rebrand.ly/LocationsDataset) are provided and should be put in the ![data](data) folder. The trained model is then used to extract the place name from the tweet texts through [model_test_json.py](model_test_json.py).

> python -u model_test.py --model_ID 0319140518  --epoch 11  --region 1 --thres 0.70

Parameters <*model_ID*> and <*epoch*> determine which model will be used. Parameter <*region*> denotes the test data set (Lousiana:0, Houston:1, Chennai:2). Parameter <*thres*> denotes the score threshold used to select the valid place name. 

## 4.Experimental results

![Screenshot](figure/result.jpg)

Figure below shows tagging result of 17 tweets. Bond text are the true place entity in the tweet. Underline texts are the incorrectly detected place entities.

![Screenshot](figure/example.jpg)

Apart from the annotated gold data, we also apply our approach on the data set without annotation, corresponding to the 2018 Florance Hurricane. There are over 100,000 tweets. The tagging result by our approach is saved in [florence_result.txt](experiments/florence_result.txt).
