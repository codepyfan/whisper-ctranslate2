[![PyPI 版本](https://img.shields.io/pypi/v/whisper-ctranslate2.svg?logo=pypi&logoColor=FFE873)](https://pypi.org/project/whisper-ctranslate2/)
[![PyPI 下載量](https://img.shields.io/pypi/dm/whisper-ctranslate2.svg)](https://pypistats.org/packages/whisper-ctranslate2)[!英文原版](https://github.com/Softcatala/whisper-ctranslate2)

# 介紹

Whisper 命令行客戶端，兼容原始 [OpenAI 客戶端](https://github.com/openai/whisper)，基於 CTranslate2。

它使用 [CTranslate2](https://github.com/OpenNMT/CTranslate2/) 和 [Faster-whisper](https://github.com/SYSTRAN/faster-whisper) 的 Whisper 實現，在相同準確率下，比 openai/whisper 快多達 4 倍，同時佔用更少內存。

項目目標：
* 提供一種簡便方式來使用 CTranslate2 的 Whisper 實現
* 讓正在使用 OpenAI Whisper CLI 的用戶能夠輕鬆遷移

# 🚀 **新項目上線！** 🚀

**Open dubbing** 是一個 AI 配音系統，利用機器學習模型自動將音頻對話翻譯並同步到不同語言！🎉

### **🔥 立即查看：[open-dubbing](https://github.com/jordimas/open-dubbing) 🔥**

# 安裝

要安裝最新版穩定版本，只需輸入：

    pip install -U whisper-ctranslate2

或者，如果你想安裝本倉庫中的最新開發（非穩定）版本，請輸入：

    pip install git+https://github.com/Softcatala/whisper-ctranslate2

# 使用預構建 Docker 鏡像

你可以直接使用已構建好的 docker 鏡像。首先拉取鏡像：

    docker pull ghcr.io/softcatala/whisper-ctranslate2:latest

Docker 鏡像已經包含 small、medium 和 large-v2 模型。

運行示例：

    docker run --gpus "device=0" \
        -v "$(pwd)":/srv/files/ \
        -it ghcr.io/softcatala/whisper-ctranslate2:latest \
        /srv/files/e2e-tests/gossos.mp3 \
        --output_dir /srv/files/
    
注意事項：
* _--gpus "device=0"_ 用於讓容器訪問 GPU。如果沒有 GPU，可以去掉此參數。
* _"$(pwd)":/srv/files/_ 將你當前目錄映射到容器內的 /srv/files/

如果你常常需要使用鏡像中沒有的模型，可以創建一個派生的 Docker 鏡像，並預加載該模型，也可以使用 Docker 捲來持久化和共享模型文件。

# CPU 與 GPU 支持

CPU 與 GPU 支持由 [CTranslate2](https://github.com/OpenNMT/CTranslate2/) 提供。

它兼容 x86-64 和 AArch64/ARM64 CPU，並集成了多種針對這些平台優化的後端：Intel MKL、oneDNN、OpenBLAS、Ruy 和 Apple Accelerate。

GPU 推理需要在系統上安裝 NVIDIA 的 cuBLAS 11.x 和 cuDNN 8.x 庫。請參考 [CTranslate2 文檔](https://opennmt.net/CTranslate2/installation.html)

默認情況下，將自動選擇最佳硬件進行推理。你可以使用 `--device` 和 `--device_index` 選項手動選擇硬件。

# 用法

命令行用法與 OpenAI Whisper 完全相同。

音頻轉錄：

    whisper-ctranslate2 inaguracio2011.mp3 --model medium
    
<img alt="image" src="https://user-images.githubusercontent.com/309265/226923541-8326c575-7f43-4bba-8235-2a4a8bdfb161.png">

音頻翻譯：

    whisper-ctranslate2 inaguracio2011.mp3 --model medium --task translate

<img alt="image" src="https://user-images.githubusercontent.com/309265/226923535-b6583536-2486-4127-b17b-c58d85cdb90f.png">

Whisper 的 translate 任務會將轉錄從原始語言翻譯為英文（目前僅支持英文為目標語言）。

你還可以通過：

    whisper-ctranslate2 --help

查看所有支持的選項及其幫助信息。

# CTranslate2 專有選項

除了 OpenAI Whisper 的命令行選項，還有一些由 CTranslate2 或 whisper-ctranslate2 提供的專用選項。

## 批量推理（Batched inference）

批量推理會獨立轉錄每個片段，可帶來額外 2~4 倍的速度提升：

    whisper-ctranslate2 inaguracio2011.mp3 --batched True
    
你還可以用 --batch_size 設置解碼時模型的最大並發請求數量。

批量推理會使用語音活動檢測（VAD）過濾，並忽略如下參數：compression_ratio_threshold、logprob_threshold、no_speech_threshold、condition_on_previous_text、prompt_reset_on_temperature、prefix、hallucination_silence_threshold。

## 量化（Quantization）

`--compute_type` 選項接受 _default、auto、int8、int8_float16、int16、float16、float32_ 等值，用於指定所用的[量化](https://opennmt.net/CTranslate2/quantization.html)類型。在 CPU 上，_int8_ 通常性能最佳：

    whisper-ctranslate2 myfile.mp3 --compute_type int8

## 從目錄加載模型

`--model_directory` 選項允許指定從哪個目錄加載 CTranslate2 格式的 Whisper 模型。例如，如果你想加載自己量化的 [Whisper 模型](https://opennmt.net/CTranslate2/conversion.html) 或 [Whisper 微調](https://github.com/huggingface/community-events/tree/main/whisper-fine-tuning-event) 的版本。模型必須為 CTranslate2 格式。

## 語音活動檢測（VAD）過濾

`--vad_filter` 選項可啟用語音活動檢測（VAD），以過濾掉音頻中無語音的部分。此步驟使用 [Silero VAD 模型](https://github.com/snakers4/silero-vad)：

    whisper-ctranslate2 myfile.mp3 --vad_filter True

VAD 過濾支持多種附加選項來決定過濾行為：

    --vad_onset VALUE (float)

高於此值的概率被視為語音。

    --vad_min_speech_duration_ms (int)

最終語音片段若短於 min_speech_duration_ms，將被丟棄。

    --vad_max_speech_duration_s VALUE (int)

語音片段最大時長（秒）。超出則在最後一次靜音時間點處分割。

## 彩色輸出

`--print_colors True` 選項會使用基於 [whisper.cpp](https://github.com/ggerganov/whisper.cpp) 的實驗性配色策略，用顏色高亮顯示高置信度或低置信度的詞：

    whisper-ctranslate2 myfile.mp3 --print_colors True

<img alt="image" src="https://user-images.githubusercontent.com/309265/228054378-48ac6af4-ce4b-44da-b4ec-70ce9f2f2a6c.png">

## 麥克風實時轉錄

`--live_transcribe True` 選項可啟用麥克風實時轉錄模式：

    whisper-ctranslate2 --live_transcribe True --language zh

https://user-images.githubusercontent.com/309265/231533784-e58c4b92-e9fb-4256-b4cd-12f1864131d9.mov

## 說話人分離（Diarization，實驗性）

實驗性支持說話人分離（需要 [`pyannote.audio`](https://github.com/pyannote/pyannote-audio)），可識別說話人，目前只支持片段級別。

啟用步驟如下：

1. 使用 `pip install pyannote.audio` 安裝 [`pyannote.audio`](https://github.com/pyannote/pyannote-audio)
2. 同意 [`pyannote/segmentation-3.0`](https://hf.co/pyannote/segmentation-3.0) 用戶協議
3. 同意 [`pyannote/speaker-diarization-3.1`](https://hf.co/pyannote/speaker-diarization-3.1) 用戶協議
4. 在 [`hf.co/settings/tokens`](https://hf.co/settings/tokens) 創建訪問令牌

然後帶上 HuggingFace API token 運行以啟用說話人分離：

    whisper-ctranslate2 --hf_token YOUR_HF_TOKEN

這樣輸出文件（如 JSON、VTT、STR）中會加入說話人名稱：

_[SPEAKER_00]: 這間屋子裡有很多人_

選項 `--speaker_name SPEAKER_NAME` 可自定義說話人名稱。

# 需要幫助？

請查看我們的[常見問題解答](FAQ.md)。

# 聯繫方式

Jordi Mas <jmas@softcatala.org>
