FROM python:3.6.8-alpine

WORKDIR /app
ADD ./post-py/* /app/

RUN apk --update add --virtual build-dependencies libffi-dev openssl-dev python-dev py-pip build-base gcc musl-dev
RUN pip install -r /app/requirements.txt

ENV POST_DATABASE_HOST post_db
ENV POST_DATABASE posts

ENTRYPOINT ["python3", "post_app.py"]

