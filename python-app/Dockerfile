FROM python:3.10-slim-bullseye

RUN apt-get update --fix-missing

COPY . .
RUN pip install --no-cache-dir --upgrade pip
RUN pip install -r requirements.txt

EXPOSE 8000
EXPOSE 8001

ENTRYPOINT ["python", "app.py"]

## To build this image from this directory:
# $ docker build -t python-app:latest .