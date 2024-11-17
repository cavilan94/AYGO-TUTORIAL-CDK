En este documento se describiran los pasos para crear una aplicación en AWS utilizando CDK en un ambiente de entorno windows.

Prerrequisitos:
Tener instalado AWS cli.
Tener instaaldo CDK cli.

Pasos:
1) crear el directorio sobre el cual se va a crear la applicación que se va a subir a AWS, para esto se ingresa por CMD o Powershell a la ruta donde se desea crear el directorio y se ejecutan los siguientes comandos:
   mkdir hello-cdk y luego cd hello-cdk para entrar al directorio
2) una vez dentro del directorio, se debe ejecutar el comando para crear la applicación basica que se va a subir a AWS, hay diferentes opciones dependiendo del lenguaje que se quiera utilizar para la aplicación, ene este caso se va a usar JAVA.

![image](https://github.com/user-attachments/assets/9bcb57ec-f074-4e0c-9cf0-f480e5184127)

comando: cdk init app --language java

al ejecutar este comando se crear una serie de archivos en la ruta donde se ejecuto el comando, dichos archivos componen la aplicación CDK basica creada en codigo java.

![image](https://github.com/user-attachments/assets/754336f6-cc6b-4aac-a346-ee31610ae20d)


Este proyecto se puede importar en un IDE que se tenga disponible, en este caso se abrio utilizando Netsbeans, dentro del proyecto se pueden evidenciar 2 clases las cuales son:HelloCdkApp y HellowCdkStack

![image](https://github.com/user-attachments/assets/179c2706-fec2-4538-899d-dd95f3e2de53)

HelloCdkApp es la función principal la cual hace un llamado a HellowCdkStack para realziar la construción de la aplicación CDK y adicionalmente en esta función principal es donde se ingresan los datos requeridos para desplegar la aplicación en aws, como el ID del usuario y la region.

![image](https://github.com/user-attachments/assets/955cedca-acd1-4add-9157-0960d29444c6)

HellowCdkStack es la a función donde se define que componentes o modulos va acrear en AWS y con que parametros.

3) ahora se debe configurar el ID de usuario y la región requeridos para la función HelloCdkApp, los cuale se pueden obtener con los siguientes comandos:

aws sts get-caller-identity --query "Account" --output text
aws configure get region

4)Ahora se debe realziar el proceso de bootstrap, que es el proceso a nivel de AWS por el cual se crean los recursos requeridos para el despliegue de la aplicación, para realizar el bootstrap se debe ejecutar el comando: cdk bootstrap, sin embargo al ejecutar este comando se genera un error debido a que el usuario creado ena ws para el laboratorio de estudiantes no tiene los privilegios para crear los roles requeridos por los recursos solicitados en la función.

![image](https://github.com/user-attachments/assets/d2a82b8e-ebd9-49ad-80d7-dbedeca5ea1f)

El comando utiliza la configuración por defecto donde valida contra el IAM de AWS si el usuario cuenta o no con los permisos requeridos apra generar los roles y recurso requeridos, para evitar este error y permitir la ejecución correcta del comando, se requiere generar un template el cual sera utilizado como base para la ajecución del comando boostrap, para poder generar ese template se debe ejecutar el siguiente comando: powershell "cdk bootstrap --show-template | Out-File -encoding utf8 bootstrap-template.yaml", este comando egenrara un archivo de template en formato .yaml en el directorio donde lo ejecutamos (debe ser el mismo directorio del proyecto)

![image](https://github.com/user-attachments/assets/5a86b3e6-e906-4090-a5b9-781b27f3aacc)

Sin embargo en Windows por defecto no permite la ejecución de este tipo de comandos ya que se reconoce como un script y su ejecución esta deshabilitada por razones de seguridad para evitar posibles infecciones por malware. para poder desactivar esta configuración de seguridad se deben ejecutar los siguientes comandos:

powershell "cdk bootstrap --show-template | Out-File -encoding utf8 bootstrap-template.yaml" (comando para habilitar la ejecución de scripts en la politica de ejecución con el usuario actual)
Set-ExecutionPolicy Restricted -Scope CurrentUser (comando para deshabilitar la ejecución de scripts en la politica de ejecución con el usuario actual)
Get-ExecutionPolicy (comando para verificar la politica de ejecución de scripts)

Se recomienda habilitar la ejecución de scripts, luego ejecutar el comando apra obtener el template.yaml, luego ejecutar el comando para vovler a deshabilitar al ejecución de scritps y finalmente ejcutar el comando apra verificar la politica de ejecución actual, para verificar que queo correctamente deshabilitado y evitar que se puedan ejecutar scritps no autorizados a futuro.

Al ejecutar el comando se crea un el template .yaml en la carpeta del proyecto

![image](https://github.com/user-attachments/assets/0f2b1e05-b44f-420a-b25f-40fea60155ab)

este archivo se puede abrir dentro del mismo IDE utilizado anteriormente, el archivo contiene mucha información que se peude agrupar por categorias.

![image](https://github.com/user-attachments/assets/e8b5487b-37b5-464b-a6de-22508e51b135)

Dentro de este archivo se requiere editar  2 cosas en la sección de resoruces, la primera es comentarear todas las lineas que tienen que ver con creaciones de roles, las cuales estan identificadas con la palabra clave "ROLE".

![image](https://github.com/user-attachments/assets/c71704f8-f5e9-47af-95fe-b844cb6b5722)

Lo segundo es editar el valor del campo Fn ::sub: por "*" en la sección FileAssetsBucketEncryptionKey

![image](https://github.com/user-attachments/assets/c4ada224-df59-4dfe-a66e-b72d191205c9)

![image](https://github.com/user-attachments/assets/144d2eed-f1db-4629-97be-67119e7688de)

Una vez estos campos han sido realizados, se salva el archivo y ya se puede realizar la ejecución del cmando boostrap indicando que use como base el template que fue editado.

comando: cdk bootstrap --template bootstrap-template.yaml

![image](https://github.com/user-attachments/assets/a1c24add-008e-465a-90c8-1c32df3e8627)

Esta vez la ejecución del comando fue exitosa, desde el lado de AWS en el menú de cloudformation debe aparecer una pila llamada CDKToolkit

![image](https://github.com/user-attachments/assets/fc14d44d-d66b-4a12-90cd-cacc167b3867)

5) Como siguiente paso se debe compilar la aplicación CDK para verificar que no hallan errores en el codigo, para el caso de una aplicación basada en JAVA, se hacer por medio del siguiente comando de mavens:

mvn compile -q

![image](https://github.com/user-attachments/assets/4a4f561f-1ad9-4d94-ba26-d1729f1b9d67)

6) se debe listar los CDK staks creados para verificar que todo este correcto, con el siguiente comando: cdk list

![image](https://github.com/user-attachments/assets/b3cfc91e-8f48-4091-b725-709e3e5eaed8)

7) Ahora se debe definir la función Lambda que se va a utilziar en la applicacion CDK, esto se hace dentro de la función HellowCdkStack

![image](https://github.com/user-attachments/assets/4b673b7a-63e8-4ea1-8e53-1bcac1999c55)

Dentro de esta función se debe indicar que el role del usuario a utilziar es LabRole, ya que es el unico con el que se cuenta dentro del IAM para el perfild el usuario de laboratorio, adicionalmente se debe ingresar el valor del arn asociado a ese role dentro del IAM de AWS.

![image](https://github.com/user-attachments/assets/2b985621-a622-43ec-ac2f-554dac63e17f)

Lo anterior se requiere para que al momento de crear la función lambda utilice este role por defecto y no intente crear uno nuevo, para lo cual como ya mencionó antes, no se cuenta con los privilegios.

8) Definir la URL de la función lambda, esto se hace dentro de la función HellowCdkStack

![image](https://github.com/user-attachments/assets/a92a3f6f-3c7a-4ca5-9901-23cf86ecfa92)

9) Ahora se debe realizar el proceso de sintetización de la aplicación, este proceso a nivel de AWS consiste en alistar el codigo antes de realizar el despliegue de la aplicación y adicionalmente generar un template en cloudformation, lo anterior se hace por medio del comando: cdk synth

![image](https://github.com/user-attachments/assets/4094902a-dacf-4baa-b6d5-f812689bb2c2)

![image](https://github.com/user-attachments/assets/55699d98-4807-4ea4-ad82-97591cb1368e)


Si la ejecución del comando fue correcta, se debe obtener una serie de archivos en la carpeta cdk.out

![image](https://github.com/user-attachments/assets/11fb2b47-0d0e-4b58-abc3-4b676ac9c4a1)

10) Desplegar el CDK Stack, por medio del siguiente comando: cdk deploy -r "usuario arn":role\LabRole, donde se indica el usuario que debe utilizar para el despliegue, evitando errores al intentar crear nuevos roles por defecto, para lo cual no se tienen permisos.

![image](https://github.com/user-attachments/assets/da16a619-a893-4d54-afa8-9e5df4ca40b0)

11) Probar la aplicación, si todo esta correcto al ingresar la URL indicada en la salida del comando del punto anterior se debe obtener la salida de la función lambda "Hello World"

![image](https://github.com/user-attachments/assets/7ed01665-0020-4c01-8917-c8b32a56fbd6)

