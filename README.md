# FastSpeech2 메뉴얼

# 1. Install Requirements

가상 환경 설정에 앞서 부가적으로 설치해야 하는 항목들이 있습니다.

## 1-0. virtual environment

파이썬 버전과 dependency를 위해 anaconda / python virtual env를 사용한 가상환경 사용을 추천드립니다.

가상환경 생성 시 python==3.7, pytorch==1.6을 미리 설정해 생성하는 것을 추천드립니다.

## 1-1. ffmpeg

ffmpeg은 비디오, 오디오 데이터를 다루는 프로그램으로 리눅스 환경이라면 아래 커맨드로 간단히 설치가 가능합니다.

```python
sudo apt-get install ffmpeg
```

window 환경은 아래 블로그를 참조해 주시기 바랍니다.

[FFMPEG 설치하기 - 윈도우(Windows)편 2020년 9월 18일 이후](https://m.blog.naver.com/chandong83/222095346417)

## 1-2. g2pk

g2pk는 한국어 텍스트에서 grapheme to phoneme 변환을 담당하는 라이브러리입니다.

pip로도 설치가 가능하지만, 환경이나 버전에 따라 해당 라이브러리에 필요한 mecab-ko 패키지에서 에러가 나는 경우가 발생합니다.

mecab-ko 패키지는 pip에서도 불안정하기 때문에, 수동으로 설치하는 것을 권장드립니다.

완벽한 설치 과정은 다음과 같습니다: mecab-ko 설치 → mecab-ko-dic 설치 [Bitbucket](https://bitbucket.org/eunjeon/mecab-ko-dic/src/master/)

## 1-3. python requirements

위 설치가 모두 완료되었다면 파이썬 패키지 설치로 준비는 완료됩니다.

```python
pip install -r requirements.txt
```

# 2. Preprocessing

## 2-1. Dataset Preparation

학습에 필요한 데이터셋은 문장에 대한 텍스트 파일과 문장을 발화하는 오디오 파일 pair이 필요합니다. Public dataset은 LJSpeech(영어)와 Korean Single Speaker (KSS) (한국어)가 대표적입니다. 

커스텀 데이터셋으로 학습할 경우 text-audio pair만 잘 이루어져 있다면 다음 단계에서 alignment 진행 후 바로 학습시킬 수 있습니다.

데이터셋이 준비되었다면 [hparams.py](http://hparams.py) 파일에서 “dataset” 변수를 임의로 바꾸어 주시고 “data_path” 변수에 데이터셋의 경로를 알맞게 변경해 주시면 됩니다.

## 2-2. Alignment Preparation

음성 데이터 학습에 앞서, speech-text pair에 대한 전처리가 필요합니다.

FastSpeech2는 utterance와 phoneme sequence간의 alignment(TextGrid)가 필요합니다. 이는 Montreal Forced Aligner (MFA)를 사용해 진행할 수 있습니다.

저자는 Korean Single Speaker (KSS) dataset을 사용했으며, 해당 데이터셋은 MFA를 사용한 alignment가 공개되어 있습니다.

- KSS ver.1.3. ([download](https://drive.google.com/file/d/1bq4DzgzuxY2uo6D_Ri_hd53KLnmU-mdI/view))
- KSS ver.1.4. ([download](https://drive.google.com/file/d/1LgZPfWAvPcdOpGBSncvMgv54rGIf1y-H/view))

TextGrid가 준비되었다면 preprocessed/${dataset_name} 경로에 파일명을 TextGrid.zip으로 바꿔 옮겨 두시면 됩니다. (압축 해제는 스크립트에서 자동으로 진행합니다.)

## 2-3. Preprocessing

위 과정이 모두 완료되었다면 전처리 스크립트를 실행합니다.

```python
python preprocess.py
```

위 커맨드를 실행하면 전처리된 데이터가 ./preprocessed 경로에 저장됩니다.

# 3. Training

학습에 앞서, 스펙트로그램을 오디오로 변환시키는 neural vocoder이 필요합니다.

사내에서 학습시킨 vocoder을 사용하시거나, pretrained vocoder model을 다운받아 사용하시면 됩니다.

준비된 vocoder 파일은 ./vocoder/pretrained_models/ 경로에 추가해 주시면 됩니다.

```python
python train.py
```

모델 학습은 위와 같이 간단한 커맨드로 시작할 수 있고, hyperparameter을 수정해야 한다면 [hparams.py](http://hparams.py)에서 수정한 후 학습해야 합니다.

### Tensorboard

해당 코드는 학습 로그를 텐서보드로 확인할 수 있도록 제작되었습니다. logdir은 ./log/${hp.dataset}/ 경로로 설정한 후 트래킹할 수 있습니다 (hparams.py에서 변경 가능).

```python
tensorboard --logdir=log/${hp.dataset}/
```

# 4. Synthesis

학습된 모델로 원하는 텍스트에 대해 오디오를 생성하고 싶은 경우 [synthesis.py](http://synthesis.py)를 사용하시면 됩니다. 학습된 모델은 training step마다 ./ckpt/ 경로에 저장되고 학습 중 evaluate 과정에서 생성된 음성은 ./eval/ 경로에 저장됩니다. 

```python
python synthesis.py --step ${n_training_step}
```

training step을 파라미터로 입력하고 synthesis를 진행한다면 커맨드라인에서 텍스트를 입력하는 파이프라인으로 구성되어 있습니다.