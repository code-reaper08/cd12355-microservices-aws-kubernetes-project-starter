FROM python:3.9-slim

WORKDIR /app

# Copy the requirements.txt file from the current directory (analytics)
COPY requirements.txt /app/

RUN apt-get update -y && apt install -y build-essential libpq-dev && \
    pip install --upgrade pip setuptools wheel && \
    pip install -r requirements.txt

# Copy app.py and config.py from the current directory (analytics) to /app/
COPY app.py /app/
COPY config.py /app/

EXPOSE 5153

CMD ["python3", "app.py"]