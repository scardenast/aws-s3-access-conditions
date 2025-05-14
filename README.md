# Laboratorio de control de acceso a bucket S3 por IP

Este laboratorio trata sobre restringir el acceso a un bucket de S3 usando condiciones basadas en la dirección IP del cliente. Se probó una política que permite solicitudes solo desde una IP específica y bloquea el resto. Se verificaron ambos casos —permitido y denegado— para confirmar que la regla funciona como se espera y que el control realmente se está aplicando.

---

## Objetivo

- Crear un bucket S3.
- Aplicar una política que solo permita acceso desde una IP específica.
- Validar el comportamiento con acceso permitido y denegado.

---

## Estructura del laboratorio

### Paso 0: Inicialización del repositorio Git

```bash
mkdir lab-s3-condiciones
cd lab-s3-condiciones
git init
```

![Paso 0 - Inicialización del repositorio Git](screenshots/00-inicializacion-repositorio-git.png)
---

### Paso 1: Crear bucket S3

```bash
aws s3api create-bucket \
  --bucket s3-condiciones-lab-sebastian \
  --region us-east-2 \
  --create-bucket-configuration LocationConstraint=us-east-2 \
  --profile labuser
```

![Paso 1 - Crear bucket](screenshots/01-crear-bucket.png)
![Paso 1 - Bucket creado](screenshots/02-bucket-creado.png)

---

### Paso 2: Crear política de restricción por IP

Se crea el archivo `bucket-policy-ip.json` con la siguiente política:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAccessExceptMyIP",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::s3-condiciones-lab-sebastian",
        "arn:aws:s3:::s3-condiciones-lab-sebastian/*"
      ],
      "Condition": {
        "NotIpAddress": {
          "aws:SourceIp": "IP/32"
        }
      }
    }
  ]
}
```

![Paso 2 - Crear política](screenshots/03-crear-politica.png)

---

### Paso 3: Aplicar política al bucket

```bash
aws s3api put-bucket-policy \
  --bucket s3-condiciones-lab-sebastian \
  --policy file://bucket-policy-ip.json \
  --profile labuser
```

![Paso 3 - Política aplicada](screenshots/04-politica-aplicada.png)

---

### Paso 4: Validar acceso desde IP permitida

```bash
aws s3 ls s3://s3-condiciones-lab-sebastian --profile labuser
```

![Paso 4 - Validar acceso permitido](screenshots/05-validar-acceso.png)

---

### Paso 5: Cambiar la política para denegar acceso

Se modifica el archivo `bucket-policy-ip.json` para permitir una IP falsa (simulación de IP no autorizada):

```json
"aws:SourceIp": "1.1.1.1/32"
```

![Paso 5 - Cambiar IP en política](screenshots/06-cambiar-ip.png)

---

### Paso 6: Validar denegación de acceso

```bash
aws s3 ls s3://s3-condiciones-lab-sebastian --profile labuser
```

![Paso 6 - Validar acceso denegado](screenshots/07-acceso-denegado.png)

---

Contacto: sebastian.cardenas.t@gmail.com
