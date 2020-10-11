FROM python:3.6.1-alpine
WORKDIR /usr/src/app
COPY ./services/app/ /usr/src/app
RUN pip3 install -r requirements.txt
CMD ["python3","api.py"]
