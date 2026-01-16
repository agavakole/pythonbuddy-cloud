FROM python:3.9-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements first for better caching
COPY requirements.txt ./requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Create non-root user
RUN adduser --disabled-password --gecos '' appuser && chown -R appuser:appuser /app
USER appuser

# Copy application code (PythonBuddy and pylint_errors from parent directory)
COPY --chown=appuser:appuser ../PythonBuddy/ ./PythonBuddy/
COPY --chown=appuser:appuser ../pylint_errors/ ./pylint_errors/

# Expose port
EXPOSE 5000

# Use gunicorn for production
CMD ["python", "-m", "PythonBuddy.app"]