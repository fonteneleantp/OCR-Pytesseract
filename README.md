# Detecção e Realce de Palavras-Chave em Imagens Usando Visão Computacional e OCR

## Descrição do Projeto

Este projeto demonstra a aplicação de técnicas de Visão Computacional e Reconhecimento Óptico de Caracteres (OCR) para identificar e realçar palavras-chave específicas em uma imagem que contém texto. Utilizando bibliotecas Python como OpenCV para processamento de imagens e PyTesseract para extração de texto, esta ferramenta detecta eficazmente palavras-chave predefinidas e desenha caixas delimitadoras ao redor delas.

## Principais Funcionalidades

- **Extração de Texto:** Utiliza PyTesseract para extrair texto de imagens, convertendo-o em um formato legível por máquinas.
- **Detecção de Palavras-Chave:** Implementa um algoritmo de busca de palavras-chave para identificar palavras ou frases predefinidas no texto extraído.
- **Realce com Caixa Delimitadora:** Usa OpenCV para desenhar caixas delimitadoras ao redor das palavras-chave detectadas, destacando-as visualmente na imagem.
- **Palavras-Chave Personalizáveis:** Permite que os usuários especifiquem uma lista de palavras-chave para detecção, tornando a ferramenta adaptável a diversos casos de uso.
- **Processamento de Imagens:** Emprega várias técnicas de processamento de imagens para melhorar a precisão da detecção e extração de texto.

## Tecnologias Utilizadas

- **Python:** A linguagem de programação principal usada no projeto.
- **OpenCV:** Utilizada para tarefas de processamento de imagem, como leitura, melhoria e desenho em imagens.
- **PyTesseract:** Uma ferramenta OCR que converte imagens de texto em texto codificado por máquina.
- **Numpy:** Usada para operações numéricas e manipulação de arrays.
- **Matplotlib:** Para visualizar as imagens de saída com as palavras-chave destacadas.

## Como Usar

### 1. Preparando o Ambiente

#### Instalação de Bibliotecas e Download de Dados

```bash
pip install opencv-python==4.6.0
sudo apt install tesseract-ocr
pip install pytesseract==0.3.9

git clone https://github.com/sthemonica/text-recognize
```

### 2. Executando o Script

```python
import pytesseract
import numpy as np
import cv2 
import re
import os
import matplotlib.pyplot as plt

from PIL import ImageFont, ImageDraw, Image
from pytesseract import Output
from google.colab.patches import cv2_imshow 

# Configurações iniciais
pytesseract.pytesseract.tesseract_cmd = r'/usr/bin/tesseract'

# Diretórios e arquivos de configuração
project_path = "/content/text-recognize/Imagens/Projeto"
image_paths = [os.path.join(project_path, f) for f in os.listdir(project_path)]
config_tesseract = "--tessdata-dir tessdata"

# Função para mostrar imagem
def mostrar(img):
    fig = plt.gcf()
    fig.set_size_inches(20, 10)
    plt.axis("off")
    plt.imshow(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))
    plt.show()

# Função para processar imagem com OCR
def OCR_processa(img, config_tesseract):
    texto = pytesseract.image_to_string(img, lang='por', config=config_tesseract)
    return texto

# Função para escrever texto na imagem
def escreve_texto(texto, x, y, img, fonte_dir, cor=(50, 50, 255), tamanho=16):
    fonte = ImageFont.truetype(fonte_dir, tamanho)
    img_pil = Image.fromarray(img)
    draw = ImageDraw.Draw(img_pil)
    draw.text((x, y-tamanho), texto, font=fonte, fill=cor)
    img = np.array(img_pil)
    return img

# Função para desenhar caixa de texto
def caixa_texto(i, resultado, img, cor=(255, 100, 0)):
    x = resultado["left"][i]
    y = resultado["top"][i]
    w = resultado["width"][i]
    h = resultado["height"][i]
    cv2.rectangle(img, (x, y), (x + w, y + h), cor, 2)
    return x, y, img

# Função principal de OCR que processa a imagem e encontra as palavras-chave
def OCR_processa_imagem(img, termo_pesquisa, config_tesseract, min_conf):
    resultado = pytesseract.image_to_data(img, config=config_tesseract, lang='por', output_type=Output.DICT)
    num_ocorrencias = 0
    for i in range(0, len(resultado['text'])):
        confianca = int(resultado['conf'][i])
        if confianca > min_conf:
            texto = resultado['text'][i]
            if termo_pesquisa.lower() in texto.lower():
                x, y, img = caixa_texto(i, resultado, img, (0,0,255))
                img = escreve_texto(texto, x, y, img, fonte_dir, (50,50,225), 14)
                num_ocorrencias += 1
    return img, num_ocorrencias

# Configurações de exibição e OCR
fonte_dir = '/content/text-recognize/Imagens/calibri.ttf'
min_conf = 30

# Processar e mostrar resultados para cada imagem
termo_pesquisa = 'learning'
for imagem in image_paths:
    img = cv2.imread(imagem)
    img_original = img.copy()
    nome_imagem = os.path.split(imagem)[-1]
    print('===================\n' + str(nome_imagem))
    img, numero_ocorrencias = OCR_processa_imagem(img, termo_pesquisa, config_tesseract, min_conf)
    print('Número de ocorrências para {} em {}: {}'.format(termo_pesquisa, nome_imagem, numero_ocorrencias))
    print('\n')
    mostrar(img)
```

### Exemplo de Uso

Para executar o script, siga as etapas de instalação e configuração descritas acima e execute o código fornecido. O script processará as imagens na pasta especificada, destacando as palavras-chave encontradas e exibindo os resultados. Além disso, salvará os textos extraídos em um arquivo de texto para futuras consultas.

Este projeto é uma demonstração prática de como combinar Visão Computacional e OCR para resolver problemas de detecção de texto em imagens, sendo útil em diversas aplicações como digitalização de documentos, análise de imagens e mais.
