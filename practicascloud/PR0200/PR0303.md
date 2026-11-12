# practica 3 Julián Almarza Álvarez
primero debemos crear un bucket que lo haremos desde comandos y este es el comando 

``` 
 aws s3 mb s3://asir-julian-2026-storage

```
luego activamos el versionado con el siguiente comando
```
 aws s3api put-bucket-versioning \
 --bucket asir-julian-2026-storage \
 --versioning-configuration Status=Enabled
```
Despues comrpobamos si lo hemos activado bien con el siguiente coamndo 
```
    aws s3api get-bucket-versioning --bucket asir-julian-2026-storage
```
luego guardamos un archivo con texto dentro con este comnado

```
    echo "Version 1.0 - SLA 99.0% CORRUPTO" >contrato_sla.txt
    aws s3 cp contrato_sla.txt s3://asir-julian-2026-storage/contratos/contrato_sla.txt
```
Luego tenemos que recuperar el archivo anterior 

```
aws s3api list-object-versions \
 --bucket asir-julian-2026-storage \
 --prefix contratos/contrato_sla.txt \
 --query "Versions[].{VersionId:VersionId, Modificado:LastModified, EsActual:IsLatest}" \
 --output table
```
y para ello suamos la version que vimos en le anterior omando en este comando 

```
 aws s3api get-object \
 --bucket asir-julian-2026-storage \
 --key contratos/contrato_sla.txt \
 --version-id "PEGAR_AQUI_EL_VERSION_ID_V1" \
 contrato_recuperado.txt
 cat contrato_recuperado.txt
```
para el ejercicio sigueinte se nos pide que borremos el s3 nuevo usando el siguiente comando 

```
aws s3 rm s3://asir-julian-2026-storage/contratos/contrato_sla.txt
```

y con el siguiente comando comprobaos los que han sido borrados 

``` 
aws s3api list-object-versions \
>  --bucket asir-julian-2026-storage \
>  --prefix contratos/contrato_sla.txt \
>  --query "DeleteMarkers[].{VersionId:VersionId, EsActual:IsLatest}" \
>  --output table
```
 Y ahora debemos restauranrlo con la id que nos ha salido en comando anterior y usando el siguente comando

 ```
 aws s3api delete-object \
>  --bucket asir-julian-2026-storage \
>  --key contratos/contrato_sla.txt \
>  --version-id dAgZWVdoZap.bv4Z1uucnohvz7cy0XNk 
```
