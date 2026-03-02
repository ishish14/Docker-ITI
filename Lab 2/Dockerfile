FROM python:3.12-alpine

WORKDIR /app

COPY . /app

RUN pip install flask requests

ENTRYPOINT ["python"]

CMD ["routes.py"]


