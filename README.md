# Codigo para backend
![Python CI tests](https://github.com/kevinccbsg/KC-CICD-backend/workflows/Python%20CI%20tests/badge.svg?branch=master)

Tenemos una api rest escrita en [Flask](https://flask.palletsprojects.com/en/1.1.x/) que tiene dos posibles entradas:

 * /register (POST) --> Registra un nuevo pedido (email y nombre del articulo)
 * /items(GET) --> Lista los productos

Para ejecutar la api rest necesitamos un MySQL como base de datos (require exportar las variables de entorno para la correcta configuración, vea *MYSQL_DATABASE_* variables in _app.py_).

## Ejecutar en local

Instalar dependencias.

```
pip install -r requirements.txt
```

Arrancar la base de datos.

```
docker-compose up -d
```

Es necesario crear una serie de tablas en la base de datos dentro del container:

```
CREATE TABLE orders (email VARCHAR(20), name VARCHAR(20));

CREATE TABLE items (name VARCHAR(20), status VARCHAR(20));
```

Luego puedes arrancar con:

```
python3 app.py
```

## Linters

Podemos ejecutar dos linters, uno para el código python (flake8) y otro para el Dockerfile (hadolint):

 * flake8: `docker run --rm -v $(pwd):/apps alpine/flake8:3.5.0 --max-line-length=120 *.py`
 * hadolint: `docker run --rm -i hadolint/hadolint < Dockerfile`

## Tests

Para lanzar los tests unitarios, debemos ejecutar los siguientes pasos:

```
docker build --no-cache -t backend-test -f Dockerfile.test .
docker run -it --name backend-test backend-test
docker cp backend-test:/app/test_results.xml ./test_results.xml
docker rm backend-test
```

Eso copiará los resultados de los tests unitarios en el fichero local *test_results.xml*, además de imprimirlos por pantalla.

## Generar artefacto

Para generar una release, cree la imagen con el siguiente comando:

`docker build --no-cache -t backend-test:$(cat version) .`

Esto generará una nueva imagen de docker con la versión correspondiente al contenido del fichero _version_.

## Publicar artefacto

Para publicar un artefacto en un repositorio de artefactos, cree un nuevo tag y suba el artefacto al repositorio (sustituya _nexus.url:port_ por su dirección de repositorio de artefactos):

```
docker tag backend-test:$(cat version) nexus.url:port/backend-test:$(cat version)
docker push nexus.url:port/backend-test:$(cat version)
```
