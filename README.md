# Speak or Chat with Me: End-to-End Spoken Language Understanding System with Flexible Inputs

## ASR-Text-Speech End-to-End SLU Model

[Papersubmitted to Interspeech 2021.](https://arxiv.org/abs/2104.05752) 

The baseline code is adopted from [Alexa End-to-End SLU System](https://github.com/alexa/alexa-end-to-end-slu). 
The original setup is a cross-modal system that co-trains text embeddings and acoustic embeddings in a shared latent space.
We further enhance this system by utilizing an acoustic module pre-trained on LibriSpeech and domain-adapting the text module on our target datasets. The pre-trained acoustic module is adopted from [End-to-End SLU](https://github.com/lorenlugosch/end-to-end-SLU).

This framework is built using pytorch with torchaudio and the transformer package from HuggingFace.
We tested using pytorch 1.5.0 and torchaudio 0.5.0.

## Data source

The model uses the Snips SLU or the Fluent Speech dataset (FSC) with speech, ground truth and ASR transcripts as training inputs.
- [Snips SLU](https://arxiv.org/pdf/1810.12735.pdf): 
To make our results comparable to the original by [Markus et. al.](https://github.com/alexa/alexa-end-to-end-slu), we use the same partition of the data, which was kindly given to us by the author, Markus Mueller.
- [Fluent Speech Command (FSC)](https://zenodo.org/record/3509828#.YH8fauhKhPZ)
- ASR Transcripts: 
As mentioned in the paper, we generate ASR transcripts by passing the audio inputs to an existing ASR model trained on LibriSpeech using the Kaldi Speech Recognition toolkit. We are planning to share the data (.csv) with ASR transcripts in this repo.

## Installation and data preparation

The environment set up is the same as alexa-slu git repository.
To install the required python packages, please run `pip install -r requirements.txt`. This setup uses the `bert-base-cased` model.
Typically, the model will be downloaded (and cached) automatically when running the training for the first time.
In case you want to download the model explicitly, you can run the `download_bert.py` script from the `dataprep/` directory,
e.g. `python download_bert.py bert-base-cased ./models/bert-base-cased`. 

## Running experiments

Core to running experiments is the `train.py` script.
When called without any parameters, it will train a model using triplet loss on the FSC dataset.
The default location for saving intermediate results is the pre-specified `--model-dir` directory.
In case it does not yet exist, it will be created.

To customize the experiments, several command line options are available (for a full list, please refer to `parser.py`):

* --dataset (The dataset to use, e.g. `fsc`)
* --experiment (The experiment class to run, e.g. `experiments.experiment_triplet.ExperimentRunnerTriplet`)
* --infer-only (Only run inference on the saved model)
* --learning-rate-bert (The learning rate for BERT branch only)
* --learning-rate (The learning for the acoustic branch and shared classifier)
* --scheduler (Learning rate scheduler)
* --finetune-bert (A boolean parameter indicating whether or not to fine-tune pre-trained BERT)
* --bert-dir (The directory to load pre-trained or domain-adapted BERT model)
* --model-save-criteria (The criteria to select the best checkpoints, e.g. `combined`: the average of validation audio + text accuracy)
* --model-dir (The directory to save the best checkpoint)
* --unfreezing-type (Choose how many of the pre-trained acoustic layers to fine-tune, e.g. `1`: fine-tune only the word module, `2`: fine-tune both phoneme and word modules)

## Example runs

Training these following models with either Snips SLU or Fluent Speech Commands should produce the following results:
(GT: ground truth transcripts, ASR: automatic speech recognition transcripts)


### ASR-Text Model (e.g. inputs: Snips SLU - ASR+GT train data, ASR test data)

`python train.py --dataset=snips --data-path=<path to input data> --finetune-bert` 

Final test acc = 0.7892 (Table 3 in the paper)

To evaluate the trained model on a different test set (e.g. GT data), run:

`python train.py --dataset=snips --data-path=<path to input data> --infer-only`

Final test acc = 1.0000 (Table 3)


### Text-Speech Model (e.g. inputs: Snips SLU - GT+Speech train data, ASR+Speech test data)

`python train.py --dataset=snips --data-path=<path to input data> --finetune-bert --unfreezing-type=2`

Final test acc (audio) = 0.7590, final test acc (text) = 0.7831 (Table 1)

Similarly, add the `--infer-only` argument to perform inference-only on a different test set.


### ASR-Text-Speech-1 Model (e.g. inputs: Snips SLU - ASR+GT+Speech train data, ASR+Speech test data)

`python train.py --dataset=snips --data-path=<path to input data> --finetune-bert --unfreezing-type=1`

Final test acc (audio) = 0.7831, final test acc (text) = 0.8373, final test acc (combined system) = 0.8976 (Table 1)

(*Note that this 'text' and 'combined system' test accuracy depends on your test input for the text branch (e.g. GT, ASR or GT+ASR). 
In this example above our test input for the text branch is ASR, thus the 'text' here refers to ASR and 'combined system' refers to ASR-Speech.*)


### ASR-Text-Speech-2 Model (e.g. inputs: Snips SLU - ASR+GT+Speech train data, ASR+Speech test data)

`python train.py --dataset=snips --data-path=<path to input data> --bert-dir=<path to pre-trained BERT model> --unfreezing-type=2`

Final test acc (audio) = 0.8012, final test acc (text) = 0.8072, final test acc (combined system) = 0.8675 (Table 1)

(*Note that $BERT_DIR is the directory that contains the best checkpoint of the ASR-Text model trained on Snips SLU in this example.*)

## Citation
If you find this repo useful, please cite our papers:

Sujeong Cha, Wangrui Hou, Hyun Jung, My Phung, Michael Picheny, Hong-Kwang Kuo, Samuel Thomas, Edmilson Morais, "Speak or Chat with Me: End-to-End Spoken Language Understanding System with Flexible Inputs", arXiv:2104.05752.

## License

This project is licensed under the Apache-2.0 License.
