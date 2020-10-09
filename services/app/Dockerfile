FROM python:3.6.10
WORKDIR /usr/src/py
ADD . /usr/src/py
RUN pip install -r requirements.txt
EXPOSE 8080
CMD ["python","api.py"]