FROM python:3.9

WORKDIR /app/backend

COPY requirements.txt /app/backend

RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y gcc default-libmysqlclient-dev pkg-config \
    && rm -rf /var/lib/apt/lists/*

RUN pip install mysqlclient
RUN pip install --no-cache-dir -r requirements.txt

COPY . /app/backend

EXPOSE 8000

# If you want to run these during image build (optional)
# RUN python manage.py makemigrations
# RUN python manage.py migrate

# Default command when container starts
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
