---
title: "Acceso inicial abusando roles que confían en el OIDC de GitHub"
author: Adan Alvarez
classes: wide
excerpt: "En esta entrada vamos a ver como un rol de AWS configurado incorrectamente para confiar en el OIDC de GitHub como una entidad federada puede ser utilizado para conseguir acceso inicial en una cuenta de AWS."
categories:
  - AWS
tags:
  - Red Team
  - Initial Access
---

Para evitar el uso de claves de Amazon Web Services (AWS) de larga duración, es posible y se recomienda configurar AWS para que confíe en el OIDC de GitHub como una entidad federada y así permitir que los flujos de trabajo de GitHub Actions accedan a los recursos de AWS sin necesidad de almacenar las credenciales de AWS como secretos de GitHub.
{: style="text-align: justify;"}

OIDC es una capa de identidad simple que se sitúa sobre el protocolo de autorización OAuth 2.0, permitiendo a los clientes verificar la identidad del usuario final basada en la autenticación realizada por un servidor de autorización.
{: style="text-align: justify;"}

Tal y como se indica en la [documentación de GitHub](https://docs.github.com/es/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services#actualizar-tu-flujo-de-trabajo-de-github-actions), para poder acceder a recursos de AWS mediante este método es necesario:
{: style="text-align: justify;"}
 - Agregar el proveedor de identidad a AWS
 - Configurar el rol y política de confianza

Una vez realizada la configuración en AWS, podremos configurar una acción en GitHub Actions que asuma el rol configurado en AWS y realice las acciones necesarias en la plataforma de AWS.
{: style="text-align: justify;"}

Las acciones necesarias para que nuestra acción pueda assumir el rol son las siguientes:
{: style="text-align: justify;"}

1. **Solicitud del ID Token de GitHub:** La GitHub Action solicita un token de identificación llamando a la URL ACTIONS_ID_TOKEN_REQUEST_URL. Para realizar esta solicitud, la Action debe proporcionar el token ACTIONS_ID_TOKEN_REQUEST_TOKEN como autorización.
{: style="text-align: justify;"}

Ejemplo:  
    
    ```curl -sLS "ACTIONS_ID_TOKEN_REQUEST_URL&audience=sts.amazonaws.com" -H "Accept: application/json; api-version=2.0" -H "Authorization: bearer ACTIONS_ID_TOKEN_REQUEST_TOKEN" -H "Content-Type: application/json"```

GitHub, que actúa como un proveedor de identidad de OIDC, verifica la solicitud y emite un ID token en forma de JWT. Este token contiene la información de identidad del "actor" que está ejecutando la acción.
{: style="text-align: justify;"}

Entre esta información se encuentra el repositorio desde la que se ha ejecutado y desde dónde ( Pull request, push en main etc.)
{: style="text-align: justify;"}

2. **Uso del JWT Token para asumir un rol en AWS:** Una vez obtenido el JWT token, este puede ser usado para asumir el rol configurado en AWS. Esta operación se realiza mediante la función '[assume-role-with-web-identity](https://docs.aws.amazon.com/cli/latest/reference/sts/assume-role-with-web-identity.html)', que es una API de AWS que permite a las entidades con tokens de identidad de proveedores de OIDC (como GitHub) asumir un rol dentro de AWS.
{: style="text-align: justify;"}

    La función 'assume-role-with-web-identity' autentica la identidad utilizando el JWT token proporcionado y asume el rol especificado. Como resultado, devuelve un conjunto de credenciales temporales de AWS para este rol.
{: style="text-align: justify;"}

3. **Acceso a AWS:** Estas credenciales temporales pueden ser utilizadas por la GitHub Action para acceder a los servicios y datos de AWS correspondientes al rol asumido, permitiendo así operaciones seguras y eficientes en la nube.
{: style="text-align: justify;"}

El problema de esto es que si el rol de AWS está mal configurado, es decir, si las condiciones no se han especificado correctamente, esto podría permitir a cualquier acción de GitHub que pueda obtener un ID token de GitHub, asumir ese rol. Como resultado, esta acción tendría acceso a todos los recursos y operaciones que ese rol de AWS permite. 
{: style="text-align: justify;"}

Por ejemplo, en la documentación de GitHub podemos ver un ejemplo de política para este tipo de roles en la cual se da acceso desde el repositorio octo-org/octo-repo
{: style="text-align: justify;"}

En este caso todas las acciones ejecutadas desde octo-org/octo-repo podrán assumir el rol:
{: style="text-align: justify;"}

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::123456123456:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:octo-org/octo-repo:*"
                },
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                }
            }
        }
    ]
}
```

Si la condición

```
"StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:octo-org/octo-repo:*"
                }
```

No existiera, o fuera de tipo ```"repo:*"```, cualquier acción de GitHub podrá asumir ese rol.
{: style="text-align: justify;"}

Si queremos probar si un rol puede ser asumido desde una de nuestras GitHub actions, podemos crear una GitHub action como la siguiente:
{: style="text-align: justify;"}

```
name: Test Action OIDC
on:
  workflow_dispatch:
permissions:
  id-token: write # This is required for requesting the JWT
  
jobs:
  test-aws-access:
    runs-on: ubuntu-latest
    steps:
    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        role-to-assume: arn:aws:iam::123456789100:role/my-github-actions-role
        aws-region: us-east-2
    - run: |
        aws sts get-caller-identity
```

Este código hace uso de la acción aws-actions/configure-aws-credentials para asumir el rol my-github-actions-role y después mediante el comando ```aws sts get-caller-identity``` verificamos si el rol se asumió correctamente.
{: style="text-align: justify;"}

Los errores en los roles de AWS son más comunes de lo que pensamos y a día de hoy hay muchos roles permitiendo el acceso desde cualquier repositorio de GitHub. Además, el acceso desde GitHub a AWS está muy extendido. Como podemos ver en la siguiente [búsqueda de GitHub](https://github.com/search?q=%2Frole-to-assume%3A+arn%3Aaws%3Aiam%3A%3A%5Cd%7B12%7D%3Arole%5C%2F%5Ba-zA-Z0-9%2B%3D%2C.%40_-%5D%2B%24%2F&type=code): 
{: style="text-align: justify;"}

[![](https://donttouchmynet.github.io/assets/images/githubawsrolesearch.PNG)](https://donttouchmynet.github.io/assets/images/githubawsrolesearch.PNG)
