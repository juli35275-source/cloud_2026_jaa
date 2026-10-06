# practica 3 Julián Almarza Álvarez
primero debemos crear un bucket que lo haremos desde comandos y este es el comando 

``` 
 aws s3 mb s3://asir-julian-2026-storage

```
luego activamos el versionado con el siguiente comando
'''
 aws s3api put-bucket-versioning \
 --bucket asir-julian-2026-storage \
 --versioning-configuration Status=Enabled
'''
Despues comrpobamos si lo hemos activado bien con el siguiente coamndo 
'''
    aws s3api get-bucket-versioning --bucket asir-julian-2026-storage
'''
luego guardamos un archivo con texto dentro con este comnado

```
    echo "Version 1.0 - SLA 99.0% CORRUPTO" >contrato_sla.txt
    aws s3 cp contrato_sla.txt s3://asir-julian-2026-storage/contratos/contrato_sla.txt
```