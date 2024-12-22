FROM python:3.9-slim

WORKDIR /app

COPY analytics/requirements.txt /app/

# Install dependencies
RUN apt-get update -y && apt install -y build-essential libpq-dev && \
    pip install --upgrade pip setuptools wheel && \
    pip install -r requirements.txt

# Copy the rest into container
COPY analytics/app.py /app/
COPY analytics/config.py /app/

# Expose the port your app runs on
EXPOSE 5153

# Run the application (adjust this command if your application starts differently)
CMD ["python3", "app.py"]