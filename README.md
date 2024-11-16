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

4) Para eso se debe ejecutar el comando: cdk bootstrap, sin embargo al ejecutar este comando se genera un error debido a que el usuario creado ena ws para el laboratorio de estudiantes no tiene los privilegios para crear los roles requeridos por los recursos solicitados en la función.

![image](https://github.com/user-attachments/assets/d2a82b8e-ebd9-49ad-80d7-dbedeca5ea1f)

El comando utiliza la configuración por defecto donde valida contra el IAM de AWS si el usuario cuenta o no con los permisos requeridos apra generar los roles y recurso requeridos, para evitar este error y permitir la ejecución correcta del comando, se requiere generar un template el cual sera utilizado como base para la ajecución del comando boostrap, para poder generar ese template se debe ejecutar el siguiente comando: powershell "cdk bootstrap --show-template | Out-File -encoding utf8 bootstrap-template.yaml", este comando egenrara un archivo de template en formato .yaml en el directorio donde lo ejecutamos (debe ser el mismo directorio del proyecto)

![image](https://github.com/user-attachments/assets/5a86b3e6-e906-4090-a5b9-781b27f3aacc)

Sin embargo en Widnows por defecto no permite la ejecución de este tipo de comandos ya que se reconoce como un script y su ejecución esta deshabilitada por razones de seguridad para evitar posibles infecciones por malware. para poder desactivar esta configuración de seguridad se deben ejecutar los siguientes comandos:

powershell "cdk bootstrap --show-template | Out-File -encoding utf8 bootstrap-template.yaml" (comando para habilitar la ejecución de scripts en la politica de ejecución con el usuario actual)
Set-ExecutionPolicy Restricted -Scope CurrentUser (comando para deshabilitar la ejecución de scripts en la politica de ejecución con el usuario actual)
Get-ExecutionPolicy (comando para verificar la politica de ejecución de scripts)

Se recomienda habilitar la ejecución de scripts, luego ejecutar el comando apra obtener el template.yaml, luego ejecutar el comando para vovler a deshabilitar al ejecución de scritps y finalmente ejcutar el comando apra verificar la politica de ejecución actual, para verificar que queo correctamente deshabilitado y evitar que se puedan ejecutar scritps no autorizados a futuro.

Al ejecutar el comando se crea un el template .yaml en la carpeta del proyecto

![image](https://github.com/user-attachments/assets/0f2b1e05-b44f-420a-b25f-40fea60155ab)


