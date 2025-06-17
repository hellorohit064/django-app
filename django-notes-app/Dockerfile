FROM python:3.9

# Set working directory to /app
WORKDIR /app

# Copy the entire project folder
COPY . /app

# Move into django-notes-app as the working directory
WORKDIR /app/django-notes-app

# Install system dependencies
RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y gcc default-libmysqlclient-dev pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
RUN pip install mysqlclient
RUN pip install --no-cache-dir -r requirements.txt

# Expose the port Django runs on
EXPOSE 8000

# Optional: run migrations during build (not recommended in prod)
# RUN python manage.py makemigrations
# RUN python manage.py migrate

# Default command to run the server (optional, could go in docker-compose)
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
