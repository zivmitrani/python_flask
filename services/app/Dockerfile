FROM python:3.6.10
WORKDIR /usr/src/app
ADD . /usr/src/app
RUN pip install -r requirements.txt
EXPOSE 8080
CMD ["python","api.py"]