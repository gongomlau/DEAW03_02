Incluid en esta carpeta todos los ficheros necesarios para el despliegue de vuestro proyecto.

## Construcción de la imagen
docker build -t lagonzalez_imagen .

## Ejecución del contenedor
docker run --name lagonzalez_contenedor -p 80:80 -p 443:443 -p 2222:22 -p 21:21 -p 30000-30010:30000-30010 lagonzalez_imagen
