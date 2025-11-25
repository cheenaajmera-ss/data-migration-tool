FROM python:3.9-slim

ENV PYTHONUNBUFFERED True
ENV PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION python
ENV DEBIAN_FRONTEND=noninteractive
ARG _oracle_instant_client_file_path
ARG _oracle_odbc_version_number

ENV APP_HOME /app
WORKDIR $APP_HOME
COPY . ./

# Install dependencies and gcloud in one layer
RUN apt-get update && \
    apt-get install -y gcc curl gnupg lsb-release ca-certificates apt-transport-https alien && \
    mkdir -p /usr/share/keyrings && \
    curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | tee /etc/apt/sources.list.d/google-cloud-sdk.list && \
    apt-get update && \
    apt-get install -y google-cloud-sdk && \
    rm -rf /var/lib/apt/lists/*

# Python dependencies
RUN pip install --upgrade pip
RUN pip install -r requirements.txt
RUN pip install ibm_db_sa

# Copy service account key and activate
COPY dmt-sa-gcc-key.json /tmp/dmt-sa-gcc-key.json
RUN gcloud auth activate-service-account dmt-sa-gcc@qwiklabs-gcp-01-53eaac22177f.iam.gserviceaccount.com \
    --key-file=/tmp/dmt-sa-gcc-key.json

# Oracle DVT Setup
RUN chmod +x ./oracle/oracle_dvt_setup.sh && \
    ./oracle/oracle_dvt_setup.sh ${_oracle_instant_client_file_path} ${_oracle_odbc_version_number}

ENV PORT 8080
CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 --timeout 0 main:app
