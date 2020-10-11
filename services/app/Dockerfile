FROM python:3.6.1-alpine
WORKDIR /usr/src/app
COPY ./requirements.txt /usr/src/app/requirements.txt
RUN pip3 install -r requirements.txt
COPY ./ /usr/src/app
CMD ["python3","api.py"]
