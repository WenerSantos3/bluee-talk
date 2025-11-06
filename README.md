# Bluee Talk - ComfyUI Customizado

Sistema de geração de vídeo com áudio usando ComfyUI customizado, com suporte a InfiniteTalk e modelos WanVideo.

## 📁 Estrutura

```
bluee-talk/
├── main.py              # Ponto de entrada principal do ComfyUI
├── server.py            # Servidor HTTP/WebSocket do ComfyUI
├── execute.sh           # Script de inicialização e configuração
├── generate_infinitetalk.py  # Geração de vídeos InfiniteTalk
├── core/                # Módulos core (variáveis globais, logger)
├── custom_nodes/        # Nós customizados do ComfyUI
│   ├── ComfyUI-WanVideoWrapper/  # Wrapper para modelos WanVideo
│   ├── ComfyUI-BlueeUtils/       # Utilitários Bluee
│   └── comfyui-videohelpersuite/  # Suíte de helpers para vídeo
├── modules/             # Módulos auxiliares
├── kokoro/              # Pipeline de síntese de áudio
├── wan/                 # Modelos e utilitários Wan
├── input/               # Arquivos de entrada (imagens, áudios)
├── output/              # Arquivos de saída gerados
```

## 🚀 Setup

### 1. Pré-requisitos

- Sistema Linux com NVIDIA GPU
- Docker e Docker Compose (opcional)
- Acesso à internet para download de modelos

### 2. Instalação

O projeto pode ser executado de duas formas:

#### Opção 1: Script de Inicialização (Recomendado)

```bash
chmod +x execute.sh
./execute.sh
```

#### Opção 2: Docker Compose

```bash
# Build e iniciar
make up-build

# Ou diretamente
docker compose -f Docker-compose.yml up -d --build
```

### 3. Configuração

O script `execute.sh` configura automaticamente:

- Ambiente Conda isolado por GPU
- Cache isolado por tipo de GPU
- Instalação de dependências Python
- Download de modelos necessários
- Custom nodes do ComfyUI

## 💡 Como Funciona

### Script `execute.sh`

O script `execute.sh` é responsável por toda a configuração e inicialização do sistema:

1. **Detecção de GPU**: Identifica a GPU NVIDIA disponível e cria caches isolados por GPU
2. **Instalação Conda**: Instala Miniconda se não existir e cria ambiente Python 3.10 isolado
3. **Instalação de Dependências**: Instala todos os pacotes Python necessários, incluindo:
   - PyTorch, torchvision, torchaudio
   - ComfyUI e custom nodes
   - SageAttention e outras bibliotecas especializadas
4. **Download de Modelos**: Baixa automaticamente os modelos necessários:
   - Wan2.1-InfiniteTalk (quantizado)
   - Wan2.1-I2V-14B-720P
   - MelBandRoformer
   - LoRAs (Lightx2v)
   - Encoders e VAEs
5. **Inicialização ComfyUI**: Inicia o servidor ComfyUI na porta 8188

### Fluxo Completo:

1. **Detecta GPU** e cria ambiente isolado
2. **Configura Conda** com ambiente Python 3.10
3. **Instala dependências** Python necessárias
4. **Clona/atualiza** ComfyUI e custom nodes
5. **Baixa modelos** se não existirem
6. **Verifica CUDA** antes de iniciar
7. **Inicia ComfyUI** na porta 8188 (0.0.0.0)

### Serviços:

#### ComfyUI Server (`main.py`)

- `start_comfyui()` - Inicia o servidor ComfyUI
- `prompt_worker()` - Processa prompts em fila
- `server.PromptServer` - Servidor HTTP/WebSocket

#### InfiniteTalk (`generate_infinitetalk.py`)

- Geração de vídeo com sincronização labial
- Suporte a vídeo-para-vídeo e imagem-para-vídeo
- Geração de comprimento ilimitado

## 📝 Uso

### Iniciar o Servidor

```bash
# Método 1: Script de inicialização (recomendado)
./execute.sh

# Método 2: Docker Compose
make up-build
# ou
docker compose -f Docker-compose.yml up -d --build
```

### Acessar o ComfyUI

Após iniciar, o ComfyUI estará disponível em:

- **URL**: `http://localhost:8188` ou `http://0.0.0.0:8188`
- **Interface**: Interface web do ComfyUI para criar workflows

### Gerar Vídeo com InfiniteTalk

```bash
python generate_infinitetalk.py \
    --ckpt_dir weights/Wan2.1-I2V-14B-480P \
    --wav2vec_dir 'weights/chinese-wav2vec2-base' \
    --infinitetalk_dir weights/InfiniteTalk/single/infinitetalk.safetensors \
    --input_json examples/single_example_image.json \
    --size infinitetalk-480 \
    --sample_steps 40 \
    --mode streaming \
    --motion_frame 9 \
    --save_file infinitetalk_res
```

## 🔧 Configurações

### Variáveis de Ambiente (definidas pelo `execute.sh`)

- `TORCH_HOME` - Cache do PyTorch (isolado por GPU)
- `COMFYUI_TEMP_DIR` - Diretório temporário do ComfyUI
- `CUDA_CACHE_PATH` - Cache CUDA (isolado por GPU)
- `CUDA_VISIBLE_DEVICES` - GPU a ser usada (padrão: 0)

### Portas

- **ComfyUI**: 8188 (configurável via `--port`)

### Modelos Baixados Automaticamente

O script `execute.sh` baixa os seguintes modelos:

| Modelo                                                           | Descrição                      |
| ---------------------------------------------------------------- | ------------------------------ |
| `Wan2_1-InfiniteTalk_Single_Q8.gguf`                             | Modelo InfiniteTalk quantizado |
| `wan2.1-i2v-14b-720p-Q8_0.gguf`                                  | Modelo I2V 720p quantizado     |
| `MelBandRoformer_fp16.safetensors`                               | Encoder de áudio               |
| `lightx2v_I2V_14B_480p_cfg_step_distill_rank64_bf16.safetensors` | LoRA Lightx2v                  |
| `umt5-xxl-enc-bf16.safetensors`                                  | Encoder de texto               |
| `clip_vision_h.safetensors`                                      | CLIP Vision                    |
| `Wan2_1_VAE_bf16.safetensors`                                    | VAE do Wan2.1                  |
| `chinese-wav2vec2-base.safetensors`                              | Wav2Vec2 para áudio chinês     |

## ⚠️ Observações Importantes

### Cache Isolado por GPU

O script cria caches isolados baseados no nome da GPU detectada. Isso permite:

- Múltiplas GPUs sem conflitos de cache
- Limpeza automática em caso de erro CUDA
- Recriação de ambiente em caso de problemas

### Recuperação Automática

Se ocorrer erro CUDA durante a execução:

- O ambiente Conda será removido automaticamente
- O script deve ser reiniciado para recriar o ambiente
- Os modelos baixados são preservados

### Requisitos de Memória

- **VRAM mínima**: Recomendado 16GB+ para modelos completos
- **RAM**: Recomendado 32GB+ para processamento de vídeos longos
- **Armazenamento**: ~50GB+ para modelos e cache

## 🔍 Troubleshooting

### Erro CUDA

Se aparecer erro CUDA:

```bash
# O script remove automaticamente o ambiente e sai
# Reinicie o script:
./execute.sh
```

### Porta já em uso

Se a porta 8188 estiver ocupada:

```bash
# Edite o script execute.sh e altere:
PORT=8188  # Para outra porta, ex: PORT=8189
```

### Modelos não baixados

Se os modelos não baixarem automaticamente:

```bash
# Verifique a conectividade e execute novamente
# Os modelos são baixados em: $HOME_DIR/bluee-talk/models/
```

## 📚 Custom Nodes

O projeto inclui os seguintes custom nodes:

- **ComfyUI-WanVideoWrapper**: Wrapper para modelos WanVideo e InfiniteTalk
- **ComfyUI-BlueeUtils**: Utilitários específicos do Bluee Talk
- **comfyui-videohelpersuite**: Ferramentas para processamento de vídeo
- **ComfyUI-KJNodes**: Nós utilitários do Kijai
- **ComfyUI-Manager**: Gerenciador de custom nodes

## 📜 Licença

Este projeto é baseado no ComfyUI e inclui modelos sob licença Apache 2.0. Consulte os arquivos LICENSE individuais para mais detalhes.
# bluee-talk
# bluee-talk
# bluee-talk
