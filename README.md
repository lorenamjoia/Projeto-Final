# Projeto-Final
Análise global de uso e cobertura da terra a partir de GlobCover (Serviço da ESA)
#IMPORTAR AS BIBLIOTECAS E FUNÇÕES:
#importar bibliotecas
    
from osgeo import gdal
import numpy as np
import matplotlib.pyplot as plt
import sys

#import ipywidgets as widgets (para gerar diagrama de sankey) 

#importar constantes
from gdalconst import *

#informar o uso de exceções
gdal.UseExceptions()

#PRÉ-PROCESSAMENTO, VALIDAÇÃO DOS DADOS:

#abrir com o gdal #inserir as imagens: (imagem do ano de 2015 não é 'acumulado' é anual, não encontrei o acumulado de 2015
filename_2000 = 'C:/Users/isado/Anual/ESACCI-LC-L4-LCCS-Map-300m-P1Y-2000-v2.0.7.tif'
filename_2005 = 'C:/Users/isado/Anual/ESACCI-LC-L4-LCCS-Map-300m-P1Y-2005-v2.0.7.tif'
filename_2010 = 'C:/Users/isado/Anual/ESACCI-LC-L4-LCCS-Map-300m-P1Y-2010-v2.0.7.tif'
filename_2015 = 'C:/Users/isado/Anual/ESACCI-LC-L4-LCCS-Map-300m-P1Y-2015-v2.0.7.tif'

#nomear os arquivos   
dataset_2000 = gdal.Open(filename_2000, GA_ReadOnly)
dataset_2005 = gdal.Open(filename_2005, GA_ReadOnly)
dataset_2010 = gdal.Open(filename_2010, GA_ReadOnly)
dataset_2015 = gdal.Open(filename_2015, GA_ReadOnly)

#boa prática, para validar a abertura dos arquivos:
try:
    dataset_2000 = gdal.Open(filename_2000, GA_ReadOnly)
    print('Arquivo P1Y-2000 aberto com sucesso!')
    dataset_2005 = gdal.Open(filename_2005, GA_ReadOnly)
    print('Arquivo P1Y-2005 aberto com sucesso!')
    dataset_2010 = gdal.Open(filename_2010, GA_ReadOnly)
    print('Arquivo P1Y-2010 aberto com sucesso!')
    dataset_2015 = gdal.Open(filename_2015, GA_ReadOnly)
    print('Arquivo P1Y-2015 aberto com sucesso!')
except:
    print('Erro na abertura do arquivo!')
    
    #metadados associado, consulta aos elementos ([0] coordenada x (canto superior esquerdo da imagem); [1] comprimento dos elementos "resolução espacial"; [2] orientação north 0; [3] coordenada y (canto superior esquerdo da imagem); [4] north 0; [5] altura do pixel)
print('Metadados contendo informações de georreferenciamento')
geotransform_2000 = dataset_2000.GetGeoTransform()
print('P1Y-2000:\n', geotransform_2000)
print()
geotransform_2005 = dataset_2005.GetGeoTransform()
print('P1Y-2005:\n', geotransform_2005)
print()
geotransform_2010 = dataset_2010.GetGeoTransform()
print('P1Y-2010:\n', geotransform_2010)
print()
geotransform_2015 = dataset_2015.GetGeoTransform()
print('P1Y-2015:\n', geotransform_2015)
print()

#checar sistema de coordenadas (https://spatialreference.org/)
sr_2000 = dataset_2000.GetProjectionRef()
sr_2005 = dataset_2005.GetProjectionRef()
sr_2010 = dataset_2010.GetProjectionRef()
sr_2015 = dataset_2015.GetProjectionRef()
print('Sistema de referência P1Y-2000:\n ', sr_2000)
print()
print('Sistema de referência P1Y-2005:\n ', sr_2005)
print()
print('Sistema de referência P1Y-2010:\n ', sr_2010)
print()
print('Sistema de referência P1Y-2015:\n ', sr_2015)
print()

#verificar compatibilidade de metadados (se todas as imagens possuem o mesmo sistema de coordenadas e sobreposição):
if (sr_2000 == sr_2005) and (sr_2000 == sr_2010):
    print("Sistema de referência iguais.")
elif (sr_2000 == sr_2015) and (sr_2005 == sr_2010):
    print("Sistema de referência iguais.")
elif (sr_2005 == sr_2015) and (sr_2010 == sr_2015):
    print("Sistema de referência iguais.")
else:
    print("Metadados espaciais diferentes")
    
    #obter metadados (matriz, x e y):
print('Arquivo', '' , 'Linhas', ',', 'Colunas')
line_2000 = dataset_2000.RasterYSize
col_2000 = dataset_2000.RasterXSize
print('P1Y-2000:', line_2000, ',', col_2000)
    
line_2005 = dataset_2005.RasterYSize
col_2005 = dataset_2005.RasterXSize
print('P1Y-2005:', line_2005,',', col_2005)
      
line_2010 = dataset_2010.RasterYSize
col_2010 = dataset_2010.RasterXSize
print('P1Y-2010:',line_2010,',', col_2010)
      
line_2015 = dataset_2015.RasterYSize
col_2015 = dataset_2000.RasterXSize
print('P1Y-2015:',line_2015,',', col_2015)

#o arquivo é composto por bandas?
banda_2000 = dataset_2000.RasterCount
print('Número de bandas P1Y-2000:', banda_2000)
banda_2005 = dataset_2005.RasterCount
print('Número de bandas P1Y-2005:', banda_2005)
banda_2010 = dataset_2010.RasterCount
print('Número de bandas P1Y-2010:', banda_2010)
banda_2015 = dataset_2015.RasterCount
print('Número de bandas P1Y-2015:', banda_2015)

#PREPARAR AS MATRIZES PARA PROCESSAMENTO:

band_2000 = dataset_2000.GetRasterBand(1)
band_2005 = dataset_2005.GetRasterBand(1)
band_2010 = dataset_2010.GetRasterBand(1)
band_2015 = dataset_2015.GetRasterBand(1)


NESTA ETAPA ESTAMOS COM PROBLEMA
numpy_2000 = band_2000.ReadAsArray().astype(float)
numpy_2005 = band_2005.ReadAsArray()
numpy_2010 = band_2010.ReadAsArray()
numpy_2015 = band_2015.ReadAsArray()

#entender a legenda, valores de pixel para associar as classes de cobertura do solo. 
#valores mínimos e máximos dos pixels, para compreender o intervalo de valore e se eles se relacionam com as classes:

(menor, maior) = band_2000.ComputeRasterMinMax()
print('Menor valor de P1Y-2000: ', menor)
print('Maior valor de P1Y-2000: ', maior)
