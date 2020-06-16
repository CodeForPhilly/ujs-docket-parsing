FROM python:3.7

ADD requirements.txt requirements.txt


RUN apt-get update \
    && apt-get install -y build-essential libpoppler-cpp-dev pkg-config python-dev

RUN pip install -r requirements.txt

ADD parser.py parser.py
