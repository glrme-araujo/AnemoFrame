# Extraindo Informações dos PDFs

O script abaixo se baseia em uma rotina de extração de informações de arquivos PDF, e a criação de um banco de dados com essas informações obtidas.

## Instalação de Bibliotecas


```python
import os # Manusear OS
import PyPDF2  # Manusear PDF
import pandas as pd # Criar Data Frame
import re  # Manusear Strings
from datetime import datetime # Extrair data de criação de arquivos
from PyPDF2.errors import PdfReadError # Tratar erros de leitura dos PDFs

 
```

## Encontrando todos os PDF's

A rotina abaixo mostra a extração dos arquivos com os seguintes critérios:

* Deve começar com 101 "f.startswith('101')"

* Deve terminar com pdf "f.lower().endswith('.pdf')"

* E não pode conter a palavra 'e1' ou 'e2' no nome do arquivo


```python
# Caminho do diretório (pode ser o caminho absoluto ou relativo)
diretorio = "caminho"

#diretorio = "C:/Users/guilhermeas/Documents/teste pdf"
# Listar todos os arquivos e pastas no diretório
arquivos = os.listdir(diretorio)

# Filtrar apenas os arquivos (ignorando subdiretórios)
arquivos_pdf = [f for f in os.listdir(diretorio) if os.path.isfile(os.path.join(diretorio, f)) and f.lower().endswith('.pdf') and f.startswith('101') and "-e1" not in f and "-e2" not in f ]
certificado = [os.path.splitext(f)[0] for f in os.listdir(diretorio) if os.path.isfile(os.path.join(diretorio, f))]


```

## Buscando Informações dentro do Arquivo e Limpando dados

Aqui extraimos todos os dados seguindo e filtramos as informações pelas palavras chave (Modelo, Tipo,Número de série)


<div aling="center">
<img src = "https://github.com/user-attachments/assets/27ee03de-f08a-4fe8-8fa4-a4c453bb7a1e"
</div>
  

A partir disso limpamos os textos e caracteres desnecessários e criamos um vetor com os dados.

O vetor armazenará:

* 1 ª Posição: Ano de criação do arquivo.
  
* 2 ª Posição: Nome do arquivo.

* 3 ª Posição: Modelo.
 
* 4 ª Posição: Tipo do equipamento.
   
* 5 ª Posição: Número de série.
     

E com o vetor, criamos um banco de dados com 5 colunas, cada coluna corresponde a um equipamento extraido de um arquivo.


```python
palavras_para_procurar  = ['Modelo' , 'Tipo','Número de série' ] # Palavras-Chave
dados = [] # Vetor com os dados encontrados
dados_final = []
lista_final = []
total_arquivos = len(arquivos_pdf)

j = 0
k= 0
for i, arquivo_pdf in enumerate(arquivos_pdf):
    
    caminho_pdf = os.path.join(diretorio, arquivo_pdf)
    timestamp_criacao = os.path.getctime(caminho_pdf)
    ano_criacao = datetime.fromtimestamp(timestamp_criacao).year
    dados.append(str(ano_criacao))
    arquivos_lidos = i + 1
    print(f"Arquivos lidos: {arquivos_lidos}/{total_arquivos}")
    
    with open(caminho_pdf, 'rb') as arquivo:
        try:

            leitor_pdf = PyPDF2.PdfReader(arquivo)
            # Lendo o texto da primeira página
            dados.append(str(arquivo_pdf))
            texto = leitor_pdf.pages[0].extract_text()
            lista = texto.splitlines()
            j = j+1
            for palavra in palavras_para_procurar:
                encontrou = False
                for linha in lista:
                    if palavra in linha:
                        encontrou = True
                        if "do copo" not in linha:
                            final = linha.replace(":", " ")
                            dados.append(final)
                if not encontrou:
                    dados.append(f"{palavra}: não consta")
                

                
        except PdfReadError:
            print(f"Erro: O arquivo '{caminho_pdf}' está corrompido ou não é um PDF válido.")
        except FileNotFoundError:
            print(f"Erro: O arquivo '{caminho_pdf}' não foi encontrado.")
        except Exception as e:
            print(f"Erro inesperado: {e}")

```
### Saida dos dados extraidos
O vetor contem todos os dados extraidos de forma sequencial.

['2018','101-164525-1.pdf','Tipo Anemômetro de pás', 'Modelo LCA6000 VT', 'Número de patrimônio Não consta', 'Número de série 102844', 'Número de identificação AA16',...]




```python
lixo = ["do copo" , "do corpo",]


for inf in dados:
    pattern = r"\b(?:{})\b".format("|".join(palavras_para_procurar))

                # Remover as palavras
    inf = re.sub(pattern, "", inf)

    pattern = r"\b(?:{})\b".format("|".join(lixo))

                # Remover as palavras
    inf = re.sub(pattern, "", inf)
    dados_final.append(inf)
    lista_final.append(dados_final)
            
    
    
```

### Transformando o vetor em tabela
Abaixo o vetor é separado em colunas, a cada itens no vetor, é realizado uma quebra de linha para gerar assim uma tabela com 5 colunas.


```python

lista_final = lista_final[1]
colunas = len(palavras_para_procurar)+2
matriz = [lista_final[i:i+colunas] for i in range(0, len(lista_final), colunas)]

for linha in matriz:
         

    print(linha)
```
['2019', '101-163257-1.pdf', '  Anemômetro de copos', '  WS200-UMB', '  001.0509.0811.010']

['2019', '101-163258-1.pdf', '  Anemômetro Ultrassônico', '  WS200-UMB', '  001.0509.0811.010']

['2018', '101-164525-1.pdf', '  Anemômetro de pás', '  LCA6000 VT', '  102844']

['2018', '101-165606-1.pdf', '  Anemômetro de pás', '   480 / 153', '  02394348 / 60349153']

['2018', '101-165710-1.pdf', '  Anemômetro de pás', '  AD-250', '  Q696136']


##  Gerando Dataframe
Aqui é gerado o dataframe, especificando os nomes de cada coluna e então é exportado para um arquivo .excel e .csv


```python

df = pd.DataFrame(matriz)
df.columns = ["Ano" , "Certificado", "Tipo", "Modelo", "Número de Série"]

df_certificados= pd.DataFrame(arquivos_pdf)
```


```python
df.to_excel('banco de dados.xlsx', index=False)
df.to_csv('banco de dados.csv', index=False)
df_certificados.to_csv('certificados salvos.csv' , index=False)
```
