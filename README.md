LLM Fine-Tuning
Kilavuzu
Windows ve Mac bilgisayarlarda kendi LLM modelinizi
adim adim fine-tune edin
Mart 2026 | Baslangic Seviyesi
QLoRA + Unsloth + Ollama | Google Colab ve Yerel Kurulum
Hazirlayan: Melih Ceyhan
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 2
Icindekiler
1. Fine-Tuning Nedir?
2. Donanim Gereksinimleri
3. Yontem Secimi: LoRA vs QLoRA
4. Ortam Kurulumu
4a. Google Colab (Onerilen Baslangic)
4b. Windows Yerel Kurulum
4c. Mac (Apple Silicon) Yerel Kurulum
5. Dataset Hazirlama
6. Fine-Tuning Islemi
7. Model Test Etme
8. GGUF'a Donusturme ve Ollama'da Kullanma
9. Sik Karsilasilan Hatalar
10. Sonraki Adimlar
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 3
1. Fine-Tuning Nedir?
Fine-tuning, onceden egitilmis (pre-trained) buyuk bir dil modelini (LLM), kendi ozel verilerinizle
yeniden egitme islemidir. Model zaten genel dil bilgisine sahiptir; fine-tuning ile onu belirli bir gorev,
uslup veya alan bilgisine uyarlarsiniz.
Ozellik
Prompt Engineering
RAG
Fine-Tuning
Zorluk
Dusuk
Orta
Orta-Yuksek
Maliyet
Dusuk
Orta
GPU gerektirir
Ozellestirme
Sinirli
Bilgi ekleme
Davranis degisikligi
Veri gereksinimi
Yok
Belgeler
500+ ornek
En iyi kullanim
Hizli denemeler
Bilgi tabani
Uslup/format/gorev
Fine-tuning su durumlarda idealdir: modelin belirli bir formatta yanit vermesini, ozel bir dilde/uslupda
konusmasini, ya da belirli bir gorevi (ornegin siniflandirma, kod uretimi, musteri hizmeti) cok iyi
yapmasini istiyorsaniz.
Ipucu: Baslarken kucuk bir modelle (1B-4B parametre) ve kucuk bir veri setiyle (200-500 ornek)
deneyin. Surecin nasil istigini anlamak sonuclari iyilestirmekten daha onemlidir.
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 4
2. Donanim Gereksinimleri
Fine-tuning'in en kritik bileseni GPU bellegi (VRAM) veya Apple Silicon'da unified memory'dir. QLoRA
teknigi sayesinde tuketici sinifi donanımla bile 7B-8B parametrelik modeller egitebilirsiniz.
NVIDIA GPU (Windows/Linux)
GPU
VRAM
QLoRA Kapasitesi
Notlar
RTX 3060 / 4060
12 GB
7B-8B model
Baslangic icin yeterli
RTX 3080 / 4070 Ti
12-16 GB
7B-8B rahat
Batch size artirabilir
RTX 3090 / 4090
24 GB
13B-14B model
Ideal tuketici GPU
Colab ucretsiz (T4)
15 GB
7B-8B model
En kolay baslangic
Apple Silicon (Mac)
Cip
Unified Memory
Kapasite
Notlar
M1 / M2 (8 GB)
8 GB
3B-4B model
Kucuk denemeler icin
M1 Pro / M2 Pro
16-32 GB
7B-8B model
Iyi baslangic noktasi
M3 Max / M4 Max
36-128 GB
14B-70B model
Profesyonel seviye
Genel Gereksinimler
•
RAM: Minimum 16 GB (32 GB onerilen)
•
Disk alani: En az 30-50 GB bos alan (model dosyalari buyuk olabilir)
•
Python: 3.10 veya 3.12 onerilen
•
Isletim sistemi: Windows 10+, macOS 13+, veya Linux
Dikkat: GPU'nuz yoksa veya 8 GB'dan az VRAM varsa, Google Colab'in ucretsiz T4 GPU'su ile
baslayabilirsiniz. Bu kilavuz Colab ile baslar.
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 5
3. Yontem Secimi: LoRA vs QLoRA
Tam fine-tuning (FFT) tum model agirliklarini gunceller ve cok yuksek kaynak gerektirir. LoRA ve
QLoRA ise verimli alternatiflerdir.
Full Fine-Tuning
LoRA
QLoRA
Ne yapar?
Tum agirliklari
gunceller
Kucuk adaptorler
ekler
4-bit + adaptor
VRAM
Cok yuksek
(48+ GB)
Yuksek
(16-24 GB)
Dusuk
(8-12 GB)
Hiz
Yavas
Orta
Hizli
Dogruluk
En yuksek
Cok yakin
Yakin
Onerilen GPU
A100 / H100
RTX 3090/4090
RTX 3060+
Baslangic icin?
Hayir
Ileri seviye
Evet
Bu kilavuzda QLoRA yontemini kullanacagiz. QLoRA, temel modeli 4-bit'e sikiştirarak bellek kullanimini
%75 azaltir ve kucuk LoRA adaptorleri ekleyerek egitim yapar. Tuketici GPU'larinda buyuk modelleri
egitmeyi mumkun kilar.
LoRA Nasil Calisir?
LoRA, orijinal model agirliklarini dondurur ve her transformer katmanina iki kucuk matris (A ve B) ekler.
Egitim sirasinda sadece bu matrisler guncellenir. Sonucta elde edilen adaptör dosyasi genellikle
50-200 MB buyuklugundedir (orjinal modelin %1'i kadar).
Temel LoRA Hiperparametreleri
Parametre
Onerilen Deger
Aciklama
r (rank)
16
Adaptor matris boyutu. Yuksek = daha fazla
kapasite ama daha fazla bellek
lora_alpha
32 (2x rank)
Olcekleme faktoru. Genellikle rank'in 2 kati
target_modules
Tum linear
katmanlar
q, k, v, o, gate, up, down projeksiyonlari
learning_rate
2e-4
Ogrenme hizi. QLoRA icin baslangic noktasi
epochs
1-3
1 ile baslayip loss degerine gore artirmak en iyisi
batch_size
2
Dusuk VRAM'da 1-2, yuksek VRAM'da 4-8
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 6
4. Ortam Kurulumu
4a. Google Colab (Onerilen Baslangic)
En kolay baslangic yolu Google Colab'dir. Ucretsiz T4 GPU (15 GB VRAM) ile 7B-8B modelleri QLoRA
ile egitebilirsiniz. Hicbir kurulum gerektirmez.
1
Colab Notebook Acin
Unsloth'un hazir notebook'larini kullanin. Tarayicinizda unsloth.ai/docs/get-started/unsloth-notebooks
adresine gidin.
•
Baslangic icin "Llama 3.1 (8B)" veya "Gemma 3 (4B)" notebook'unu secin
•
"Open in Colab" butonuna tiklayin
2
GPU'yu Etkinlestirin
Colab'da Runtime > Change runtime type > T4 GPU secin. Sag ustte "Connect" butonuyla baglanti
kurun.
3
Unsloth Kurun
Notebook'un ilk hucresinde su komut olacaktir:
%%capture
!pip3 install unsloth
Ipucu: Colab notebook'larini degistirmeden 'Run all' ile calistirabilirsiniz. Ilk denemenizi boylece yapip
sureci gozlemleyin.
4
Devam
Model yukleme, dataset hazirlama ve egitim icin Bolum 5 ve 6'ya gecin. Colab'da tum bu adimlar
notebook icinde hazir gelir.
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 7
4b. Windows Yerel Kurulum
Windows'ta yerel fine-tuning icin NVIDIA GPU, CUDA Toolkit ve doğru Python ortami gereklidir.
Asagidaki adimlari izleyin.
On Kosullar
•
NVIDIA GPU surucusu (en guncel): nvidia.com/drivers adresinden indirin
•
CUDA Toolkit 12.x: developer.nvidia.com/cuda-downloads
•
cuDNN: developer.nvidia.com/cudnn (CUDA sonrasi kurun)
•
Visual Studio C++ Build Tools (2019 veya 2022)
1
Miniconda Kurun
PowerShell'de:
# Miniconda indirin ve kurun
Invoke-WebRequest -Uri
  "https://repo.anaconda.com/miniconda/
   Miniconda3-latest-Windows-x86_64.exe"
  -OutFile ".\miniconda.exe"
Start-Process -FilePath ".\miniconda.exe"
  -ArgumentList "/S" -Wait
del .\miniconda.exe
2
Python Ortami Olusturun
conda create --name finetune python=3.12 -y
conda activate finetune
3
PyTorch + CUDA Kurun
pip3 install torch torchvision \
  --index-url https://download.pytorch.org/whl/cu130
GPU'nun gorunur olduğunu dogrulayin:
python3 -c "import torch;
  print(torch.cuda.is_available())"
# True yazdirmali
4
Unsloth Kurun
pip3 install unsloth
Dikkat: Kurulumdan sonra bilgisayari yeniden baslatin. Windows'ta num_proc=1 ayari gerekebilir
(Bolum 9'da detaylar).
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 8
4c. Mac (Apple Silicon) Yerel Kurulum
Mac'te Apple'in MLX framework'u ile fine-tuning yapabilirsiniz. MLX, Apple Silicon'in unified memory
mimarisini kullanarak verimli egitim saglar. Metal GPU hizlandirmasi ile calısir.
Dikkat: MLX sadece Apple Silicon (M1/M2/M3/M4/M5) cipler uzerinde calisir. Intel Mac
desteklenmez.
1
Python Ortami Olusturun
# Homebrew ile Python kurun (yoksa)
brew install python@3.12
# Virtual environment olusturun
python3 -m venv ~/finetune-env
source ~/finetune-env/bin/activate
2
MLX ve MLX-LM Kurun
pip3 install mlx mlx-lm
Bu iki paket Apple Silicon uzerinde LLM calistirma ve fine-tuning icin gereken her seyi icerir.
3
Fine-Tuning Baslatin
MLX-LM, komut satirindan tek satirda fine-tuning baslatmanizi saglar:
mlx_lm.lora \
  --model mlx-community/Llama-3.2-3B-Instruct-4bit \
  --train \
  --data ./my_dataset \
  --iters 300 \
  --batch-size 4
4
Adaptoru Birlestirin
mlx_lm.fuse \
  --model mlx-community/Llama-3.2-3B-Instruct-4bit \
  --adapter-path adapters \
  --save-path ./my-finetuned-model
Ipucu: Mac'te MLX yolu daha basit bir kurulum sunar. Ancak Unsloth'un Colab notebook'lari sureci
anlamak icin en iyi baslangic noktasidir.
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 9
5. Dataset Hazirlama
Dataset kalitesi, fine-tuning basarisinin en onemli faktorudur. 500 yuksek kaliteli ornek, 5.000 dusuk
kaliteli ornekten daha iyi sonuc verir.
Dataset Formati (ChatML)
Modern LLM'ler icin en yaygin format, her ornegin system/user/assistant rolleriyle tanimlandigi
ChatML formatidir. Verinizi JSONL veya JSON formatinda hazirlayin.
Ornek dataset.jsonl dosyasi:
{
  "messages": [
    {"role": "system",
     "content": "Sen yardimci bir asistansin."},
    {"role": "user",
     "content": "Python'da liste nasil olusturulur?"},
    {"role": "assistant",
     "content": "Python'da liste olusturmak icin
     koseli parantez kullaniriz:
     liste = [1, 2, 3]"}
  ]
}
Dataset Hazirlama Ipuclari
•
Kalite > Miktar: 200-500 iyi ornekle baslayın
•
Tutarlilik: Tum orneklerde ayni format ve uslup kullanin
•
Cesitlilik: Farkli soru tipleri ve baglamlar ekleyin
•
Temizlik: Yazim hatalari, tutarsizliklari duzurun
•
Uzunluk: Cok kisa/uzun yanitlardan kacinin, dengeli tutun
Veri Kaynaklari
•
Manuel olusturma: En kaliteli yontem. Kendi alan bilginizle ornekler yazin
•
Sentetik veri: ChatGPT/Claude ile ornekler uretin, sonra manuel kontrol edin
•
Hazir datasetler: Hugging Face Hub'da binlerce dataset mevcut (huggingface.co/datasets)
•
Dokuman donusumu: PDF/web iceriklerinizi soru-cevap formatina cevirtin
Hizli Sentetik Veri Olusturma Ornegi
ChatGPT veya Claude'a soyle bir prompt verin:
Prompt: Benim icin [KONU] hakkinda 50 tane soru-cevap
cifti olustur. Her biri su JSON formatinda olsun:
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 10
{"messages": [
  {"role": "system",
   "content": "[SISTEM MESAJI]"},
  {"role": "user", "content": "[SORU]"},
  {"role": "assistant", "content": "[CEVAP]"}
]}
Cevaplar detayli ve dogru olsun.
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 11
6. Fine-Tuning Islemi (Unsloth)
Asagida Unsloth ile adim adim QLoRA fine-tuning kodu bulunmaktadir. Bu kodu Colab notebook'ta
veya yerel Jupyter/Python'da calistirabilirsiniz.
1
Model Yukleme
from unsloth import FastLanguageModel
model, tokenizer = FastLanguageModel.from_pretrained(
  model_name = "unsloth/Llama-3.2-3B-Instruct",
  max_seq_length = 2048,
  load_in_4bit = True,  # QLoRA icin
)
load_in_4bit=True parametresi QLoRA'yi etkinlestirir ve VRAM kullanimini yaklasik 4 kat azaltir.
2
LoRA Adaptorleri Ekle
model = FastLanguageModel.get_peft_model(
  model,
  r = 16,
  lora_alpha = 32,
  lora_dropout = 0,
  target_modules = [
    "q_proj", "k_proj", "v_proj", "o_proj",
    "gate_proj", "up_proj", "down_proj"
  ],
  use_gradient_checkpointing = "unsloth",
)
3
Dataset Yukle ve Formatla
from datasets import load_dataset
# Yerel dosyadan:
dataset = load_dataset("json",
  data_files="dataset.jsonl", split="train")
# veya HuggingFace'den:
# dataset = load_dataset("kullanici/dataset-adi",
#  split="train")
# Chat template uygula
def format_chat(example):
  text = tokenizer.apply_chat_template(
    example['messages'],
    tokenize=False,
    add_generation_prompt=False
  )
  return {'text': text}
dataset = dataset.map(format_chat)
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 12
4
Egitimi Baslat
from trl import SFTTrainer
from transformers import TrainingArguments
trainer = SFTTrainer(
  model = model,
  tokenizer = tokenizer,
  train_dataset = dataset,
  dataset_text_field = "text",
  max_seq_length = 2048,
  args = TrainingArguments(
    output_dir = "./output",
    per_device_train_batch_size = 2,
    gradient_accumulation_steps = 4,
    num_train_epochs = 1,
    learning_rate = 2e-4,
    logging_steps = 10,
    warmup_steps = 5,
    fp16 = True,
    optim = "adamw_8bit",
  ),
)
trainer.train()
Ipucu: Egitim sirasinda training loss degerini izleyin. 0.5-1.0 arasi genellikle iyi bir isarettir. Loss
azalmiyorsa learning_rate'i dusurun. Loss 0'a yaklasiyorsa overfitting olabilir.
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 13
5
Modeli Kaydet
# LoRA adaptorunu kaydet (kucuk dosya, ~100-200 MB)
model.save_pretrained("./my-lora-adapter")
tokenizer.save_pretrained("./my-lora-adapter")
# Hugging Face Hub'a yuklemek icin (istege bagli):
# model.push_to_hub("kullanici/model-adi")
# tokenizer.push_to_hub("kullanici/model-adi")
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 14
7. Model Test Etme
Fine-tuning sonrasi modeli test ederek egitimin basarili olup olmadigini dogrulayin.
Unsloth ile Hizli Test
from unsloth import FastLanguageModel
FastLanguageModel.for_inference(model)
messages = [
  {"role": "user",
   "content": "Merhaba, bana Python hakkinda
   bilgi verir misin?"}
]
inputs = tokenizer.apply_chat_template(
  messages,
  tokenize=True,
  add_generation_prompt=True,
  return_tensors="pt"
).to(model.device)
outputs = model.generate(
  input_ids=inputs,
  max_new_tokens=256,
  temperature=0.7
)
print(tokenizer.decode(outputs[0],
  skip_special_tokens=True))
Kontrol Listesi
•
Yanit, egitim verisindeki format ve usluba uyuyor mu?
•
Model, egitim verisinde olmayan sorulara da uygun yanitliyor mu?
•
Halusinasyon (uydurma bilgi) orani kabul edilebilir mi?
•
Yanit uzunlugu beklentilere uygun mu?
Dikkat: Eger model genel/jenerik yanitlar veriyorsa veya egitim verisindeki formati takip etmiyorsa,
dataset kalitenizi kontrol edin ve gerekirse daha fazla/daha iyi orneklerle tekrar egitin.
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 15
8. GGUF'a Donusturme ve Ollama'da Kullanma
Fine-tune edilmis modelinizi gunluk kullanim icin Ollama'ya tasimak en pratik yoldur. Bunun icin modeli
GGUF formatina donusturmeniz gerekir.
Yontem A: Unsloth ile Dogrudan GGUF Kaydetme
Unsloth, dogrudan GGUF formatinda kaydetme destegi sunar:
# 4-bit quantize GGUF olarak kaydet
model.save_pretrained_gguf(
  "./my-model-gguf",
  tokenizer,
  quantization_method = "q4_k_m"
)
# Diger quantize secenekleri:
# "q8_0"  - Daha kaliteli, daha buyuk dosya
# "q4_k_m" - Dengeli (onerilen)
# "f16"   - Tam hassasiyet, en buyuk dosya
Yontem B: llama.cpp ile Manuel Donusum
# llama.cpp reposunu klonlayin
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
# Donusturun
python3 convert_hf_to_gguf.py \
  ../my-merged-model \
  --outfile my-model.gguf \
  --outtype q4_k_m
Ollama'da Kullanma
1
Ollama Kurun
ollama.com adresinden isletim sisteminize uygun versiyonu indirip kurun.
2
Modelfile Olusturun
Modelfile adinda bir dosya olusturun:
FROM ./my-model.gguf
PARAMETER temperature 0.7
PARAMETER top_p 0.9
SYSTEM """Sen yardimci bir Turkce asistansin."""
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 16
3
Modeli Olusturun ve Calistirin
# Modeli olustur
ollama create my-model -f Modelfile
# Modeli calistir
ollama run my-model
# Artik terminal'den sohbet edebilirsiniz!
Ipucu: Ollama'da modelinizi olusturduktan sonra, Open WebUI gibi arayuzlerle de kullanabilirsiniz.
Docker ile: docker run -p 3000:8080 ghcr.io/open-webui/open-webui:main
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 17
9. Sik Karsilasilan Hatalar ve Cozumleri
Hata
Olasi Sebep
Cozum
CUDA Out of Memory
Yetersiz GPU bellegi
batch_size=1 yapin, max_seq_length azaltin, daha
kucuk model deneyin
torch.cuda.is_available()
= False
CUDA dogru
kurulmamis
CUDA Toolkit + cuDNN kurun, PyTorch'u CUDA
destekli kurun
num_proc hatasi
(Windows)
Coklu islem destegi
Dataset yukleme/map islemlerinde num_proc=1
kullanin
Training loss azalmiyor
Ogrenme hizi uygun
degil
learning_rate'i 1e-4 veya 5e-5'e dusurun
Training loss = 0
Overfitting
Epoch sayisini azaltin, daha fazla veri ekleyin
Model jenerik yanit
veriyor
Yetersiz/kalitesiz veri
Dataset kalitesini artirin, daha tutarli ornekler
ekleyin
Tokenizer hatasi
Yanlis chat template
Modele uygun chat template kullandiginizdan
emin olun
Mac: MLX import hatasi
Intel Mac veya eski
macOS
Apple Silicon (M1+) ve macOS 13+ gerekli
Performans Iyilestirme Ipuclari
•
Gradient accumulation: Batch size kucukse, gradient_accumulation_steps artirarak efektif batch
buyutun
•
max_seq_length: Ihtiyacinizdan fazla tutmayin. 1024 veya 2048 genellikle yeterli
•
Mixed precision: fp16=True veya bf16=True kullanarak hizi artirin
•
Gradient checkpointing: Unsloth'un ozel modu ('unsloth') standart PyTorch'tan daha az bellek
kullanir
LLM Fine-Tuning Kilavuzu | Windows & Mac
Sayfa 18
10. Sonraki Adimlar
Ilk fine-tuning deneyiminizi tamamladiginizda, asagidaki konularla devam edebilirsiniz:
Ogrenme Yol Haritasi
Seviye
Konu
Aciklama
Baslangic
Dataset buyutme
Ornek sayisini 1000-5000'e cikarin
Baslangic
Farkli modeller
Llama, Gemma, Qwen, Mistral deneyin
Orta
Hiperparametre ayari
rank, alpha, LR, epoch deneyleri
Orta
DPO/ORPO
Tercih optimizasyonu ile kalite artirma
Orta
Evaluation
Otomatik degerlendirme metrikleri olusturun
Ileri
GRPO (RL)
Odul fonksiyonuyla davranis egitimi
Ileri
Multi-turn egitim
Cok turlu konusma verileriyle egitim
Ileri
vLLM ile deploy
Uretim ortaminda yuksek performansli sunma
Faydali Kaynaklar
•
Unsloth Dokumantasyonu - Resmi kilavuzlar ve notebook'lar
•
Hugging Face Training Rehberi - Transformer egitim temelleri
•
MLX Dokumantasyonu - Apple Silicon icin ML framework
•
Ollama - Yerel LLM calistirma araci
•
Hugging Face Datasets - Hazir veri setleri
Kaynaklar
1. Unsloth Fine-Tuning Guide, https://unsloth.ai/docs/get-started/fine-tuning-llms-guide
2. Unsloth Windows Kurulum, https://unsloth.ai/docs/get-started/install/windows-installation
3. Apple MLX LLM Rehberi, https://machinelearning.apple.com/research/exploring-llms-mlx-m5
4. WWDC25 MLX LLM, https://developer.apple.com/videos/play/wwdc2025/298/
5. QLoRA Fine-Tuning Consumer GPU, https://www.stack-junkie.com/blog/lora-fine-tuning-consumer-gpu
6. Ollama Model Import, https://docs.ollama.com/import
7. Unsloth LoRA Hyperparameters, https://unsloth.ai/docs/get-started/fine-tuning-llms-guide/lora-hyperparameters-guide
8. Hugging Face Transformers Training, https://huggingface.co/docs/transformers/training
