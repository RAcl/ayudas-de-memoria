# Terraform

## Instalación

Se recomienda seguir las instrucciones de [Hashicorp](https://developer.hashicorp.com/terraform/install), actualmente en debian/ubuntu son:

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

## Resumen de comandos

### Más usuados:

* **init**: Prepara tu directorio de trabajo para otros comandos. Al realizar el init descarga los providers de la configuración.
  ```bash
  terraform init
  ```
* **validate**: Verifica si la configuración es válida.
  ```bash
  terraform validate
  ```
* **plan**: Muestra los cambios requeridos por la configuración actual. 
  ```
  terraform plan
  ```
  * -out [plan-file]: exporta el plan a un archivo. Ej:
    ```bash
    terraform plan mi-plan
    ```
* **apply**: Crea o actualiza la infraestructura. 
  ```bash
  terraform apply
  ```
  * [plan-file]: aplica el plan (sin ninguna otra confirmación) 
    ```bash
    terraform apply mi-plan
    ```
  * -var 'foo=bar': Setea una variable. 
    ```bash
    terraform apply -var VPCID=vpc-0011223344
    ```
  * -var-file=[archivo]: Setea las variables según el archivo entregado. 
    ```bash
    terraform apply -var-file=variables.txt
    ```
  * -replace [recurso]: reemplaza el recurso, lo elimina y lo crea de nuevo. Se hace para forzar el remplazo de un recurso que está fallando.
    ```
    terraform apply -replace aws_instance.bastion_instance
    ```
  * -auto-approve=true: Aplica el cambio sin la confirmación (no recomendado) ```terraform apply -auto-approve=true```
  * -destroy: Destruye la infraestructura creada anteriormente. 
    ```bash
    terraform apply -destroy
    ```
* **destroy**: Destruye la infraestructura creada anteriormente. (alias de apply -destroy) 
  ```bash
  terraform destroy
  ```

### Otros comandos:

* **console**: Prueba expresiones de Terraform en un prompt de comando interactivo.
* **fmt**: Formatéa tus archivos de configuración en el estilo estándar. Sin parametros lo hace sobre el directorio, con archivo de parámetro solamente al archivo.
* **get**: Instala o actualiza módulos remotos de Terraform.
* **graph**: Genera un gráfico Graphviz de los pasos en una operación. ejemplo: ```terraform graph| dot -Tsvg > grafico.svg```
* **import**: Asocia infraestructura existente con un recurso de Terraform. Ejemplo: ```terraform import [recurso-terraform] [aws-recurso-id]```
* **output**: Muestra valores de salida de tu módulo raíz.
* **providers**: Muestra los proveedores requeridos para esta configuración.
* **refresh**: Actualiza el estado para que coincida con los sistemas remotos.
* **show**: Muestra el estado actual o un plan guardado. Por ejemplo ```terraform show [plan-file]```
* **state**: Gestión avanzada de estado (terraform.tfstate)
  * list: Despliega los estados.
  * show [recurso]: Muestra el detalle del recurso.
  * mv [Origen] [Destino]: Renombra el recurso en el estado.
  * rm [recurso]: Elimina recurso del estado (el recurso queda sin seguimiento en terraform).
* **taint**: Marca contaminada una instancia de recurso, como no completamente funcional. Fuerza su replazo.
* **untaint**: Elimina el estado 'contaminado' de una instancia de recurso.
* **version**: Muestra la versión actual de Terraform.
* **workspace**: Gestión de espacios de trabajo.
  * list: Despliega los espacios de trabajo.
  * new [espacio]: Crea un nuevo espacio de trabajo.
  * select [espacio]: Selecciona el espacio de trabajo.
  * delete [espacio]: Elimina el espacio de trabajo.

## Logs
Exportar variables de entorno necesarias según nivel:
```bash
export TF_LOG=level
```
Donde **level** puede ser uno de los siguientes niveles:

|Detalle    |Nivel  |
|---        |---    |
|Mínimo     |INFO   |
|...        |WARING |
|...        |ERROR  |
|...        |DEBUG  |
|Máximo     |TRACE  |

Para que la salida se guarde en un archivo hay que agregar además:
```bash
export TF_LOG_PATH=path/to/file
```

Una vez realizado el debug se pueden eliminar las variables de entorno con ```unset ENV_VAR```

## Recursos

Para crear recursos ir a la documentación oficial en [https://registry.terraform.io/](https://registry.terraform.io/)
