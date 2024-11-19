FROM python:3-alpine

WORKDIR /usr/src/app
EXPOSE 5002
COPY main.py requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
ENV LOG_LEVEL="INFO"
CMD uvicorn main:app --host 0.0.0.0 --port 5002