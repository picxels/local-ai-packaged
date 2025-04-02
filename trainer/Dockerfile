FROM nvidia/cuda:12.1.1-cudnn8-devel-ubuntu22.04

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV PYTHONUNBUFFERED=1
ENV CUDA_HOME=/usr/local/cuda
ENV PATH=${CUDA_HOME}/bin:${PATH}
ENV LD_LIBRARY_PATH=${CUDA_HOME}/lib64:${LD_LIBRARY_PATH}

# Install system dependencies
RUN apt-get update && apt-get install -y \
    python3.11 \
    python3.11-dev \
    python3-pip \
    tesseract-ocr \
    default-jre \
    curl \
    git \
    build-essential \
    libgl1-mesa-glx \
    poppler-utils \
    && rm -rf /var/lib/apt/lists/*

# Ensure python3 and pip3 are properly installed and linked
RUN apt-get update && apt-get install -y python3 python3-pip && \
    ln -sf /usr/bin/python3 /usr/bin/python && \
    ln -sf /usr/bin/pip3 /usr/bin/pip

# Set working directory
WORKDIR /app

# Copy application code
COPY . .

# Copy requirements directory
COPY requirements /app/requirements

# Install NumPy 1.x first to avoid 2.x conflicts
RUN pip3 install --no-cache-dir "numpy<2.0"

# Install PyTorch with CUDA support
RUN pip3 install --no-cache-dir torch==2.1.2 torchvision==0.16.2 torchaudio==2.1.2 --index-url https://download.pytorch.org/whl/cu121

# Install dependencies
RUN pip3 install --no-cache-dir -r requirements/base.txt
# Install other dependencies
RUN pip3 install --no-cache-dir -r requirements/dev.txt
RUN pip3 install --no-cache-dir -r requirements/prod.txt
RUN pip3 install -e .

# Install additional API requirements
RUN pip3 install --no-cache-dir fastapi uvicorn gunicorn httpx python-multipart
RUN pip3 install --no-cache-dir sentence-transformers>=2.6.1  # Updated version

# Install pytest and dependencies
RUN pip3 install pytest pytest-cov networkx matplotlib

# Environment variables for services
ENV OLLAMA_BASE_URL=http://ollama:11434
ENV QDRANT_URL=http://qdrant:6333
ENV QDRANT_PORT=6333
ENV SUPABASE_URL=http://supabase-kong:8000

# Set Python to not create .pyc files in Docker
ENV PYTHONDONTWRITEBYTECODE=1

# Create directories
RUN mkdir -p /app/data /app/logs /app/cache /app/models

# Set permissions
RUN chmod -R 755 /app

# Expose port
EXPOSE 9099

# Health check
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
CMD ["python3", "-m", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "9099", "--log-level", "debug"]

# Set Python path
ENV PYTHONPATH=/app
