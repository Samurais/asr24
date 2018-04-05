# asr24
24-hour ASR

Within 24 hours, train an ASR for a surprise incident language (IL), and get native transcriptions of recorded speech.

Use a pre-trained acoustic model, an IL dictionary, and an IL language model.
This approach converts phones directly to NI words, instead of using multiple cross-trained ASRs to make English words
from which phone strings are extracted, merged with [PTgen](https://github.com/uiuc-sst/PTgen), and reconstituted into IL words (which turned out to be too noisy).

## Installing

### Set up Krisztián Varga's [extension](https://chrisearch.wordpress.com/2017/03/11/speech-recognition-using-kaldi-extending-and-using-the-aspire-model/) of [ASpIRE](http://kaldi-asr.org/models.html).

- `git clone https://github.com/kaldi-asr/kaldi`
- Build Kaldi, following the instructions in Kaldi's `INSTALL` file.
- Get the [ASpIRE chain model](http://kaldi-asr.org/models.html):
```
    cd kaldi/egs/aspire/s5
    wget http://dl.kaldi-asr.org/models/0001_aspire_chain_model.tar.gz
    tar xf 0001_aspire_chain_model.tar.gz
    steps/online/nnet3/prepare_online_decoding.sh --mfcc-config conf/mfcc_hires.conf data/lang_chain exp/nnet3/extractor exp/chain/tdnn_7b exp/tdnn_7b_chain_online
    utils/mkgraph.sh --self-loop-scale 1.0 data/lang_pp_test exp/tdnn_7b_chain_online exp/tdnn_7b_chain_online/graph_pp
```
- Verify that it can transcribe a recording of English speech.  (The scripts `cmd.sh` and `path.sh` let the shell find `kaldi/src/online2bin/online2-wav-nnet3-latgen-faster`.)

`ffmpeg -i MySpeech.wav -acodec pcm_s16le -ac 1 -ar 8000 8khz.wav` or `sox MySpeech.wav -r 8000 8khz.wav`
```
. cmd.sh && . path.sh
online2-wav-nnet3-latgen-faster \
  --online=false \
  --do-endpointing=false \
  --frame-subsampling-factor=3 \
  --config=exp/tdnn_7b_chain_online/conf/online.conf \
  --max-active=7000 \
  --beam=15.0 \
  --lattice-beam=6.0 \
  --acoustic-scale=1.0 \
  --word-symbol-table=exp/tdnn_7b_chain_online/graph_pp/words.txt \
  exp/tdnn_7b_chain_online/final.mdl \
  exp/tdnn_7b_chain_online/graph_pp/HCLG.fst \
  'ark:echo utterance-id1 utterance-id1|' \
  'scp:echo utterance-id1 <your_8khz_file.wav>|' \
  'ark:/dev/null'
```

### Install our own code.
```
    cd kaldi/egs/aspire/s5
    git clone https://github.com/uiuc-sst/asr24.git
```
